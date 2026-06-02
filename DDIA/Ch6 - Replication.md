Intro
- reasons for replication: geographical proximity, reliable availability, read throughput
- replication difficult b/c data changes (every write needs to be processed by every replica)
- Choices ex. async or not, how to handle failed replicas
- Eventual consistency
- different from backups: replicas store the current state, backups store past states

Single leader replication
- process
	- one node designated as leader, client writes get sent to leader
	- other replicas designated followers, when leader writes, it also sends the change to all its followers as part of a replication log / change stream
	- replicas apply the writes in the change stream
	- read operations can hit any node
- for sharded DB, each shard has 1 leader
- build-in feature of PostgreSQL, Mongo and DynamoDB, Kafka, Raft (CockroachDB)

Synch vs Async
- synch: leader wait until follower received write before reporting success to the user
	- pro: followers guaranteed to be consistent with leader
	- con: if the follower doesn't respond, the write cannot be processed <– this is blocking
	- therefore most "sync" implementations are semi-sync: only enforce sync from 1 of the followers (if slow, someone else is now the sync follower)
	- quorem: majority are sync, often used for eventual consistency or use consensus for automatic leader election
- asynch: leader doesn't wait for follower response
	- con: if leader fails, any writes that have not been replicated are lost (writes therefore not guaranteed to be durable)
	- pro: leader can write even if followers fallen behind, useful is many followers

Setting up new followers
- can't copy files over b/c copying files takes time and data can change inbetween
- instead send snapshot (in replication log) + data changes since snapshot
	- archiving the replication log + periodic snapshots also good for backups (WAL-G does this for  PostgreSQL)
		- object stores like S3 cheaper than virtual disks like Amazon EBS, but higher latency, no file-system interface, has API call fees (which forces batching), FUSE (filesystem in userspace) helps but still lack many file system features
			- can use tiered storage, EBS for WAL, or even serverless (all in S3)

Handling Node Outages
- if follower crashes can request change log, leader clears change log when all followers up-to-date till that point
- leader failure more complicated:
	- failover: promote a follower to leader, client reconfigured to send writes to new leader, other followers start consuming data changes from new leader
	- automatic failover: detecting new leader through timeout, new leader usually most up-to-date follower (decided through consensus), client sends data to new leader
- failover has many things that can go wrong
	- if replication async, new leader might not have received all writes from old leader before it failed (very bad)
	- two followers can accidentally both think they're leader, conflicts are a hassle
	- guarding against ^ is called fencing
	- right timeout is hard too (could be traffic spike)

Implementing replication logs
- sending every write request
	- be careful about nondeterministic functions like NOW() or RANDOM() or triggers
	- replicas need to execute in the same order (limits concurrency)
- physical write-ahead log shipping
	- WAL allows followers to build the same info as leader (PostgreSQL uses this)
	- con: WAL very granular (byte level changes), need storage format to be the same across database software versions (to update all nodes)
- logical (row-based) log replication
	- use diff log formats for replication and for the storage engine
	- logical log keeps what data changes are made (row insert/update/delete instead of WAL which specifies which physical bytes write to where)

problems with replication log
- replication reasons: failure tolerance, scalability, latency
- async will have replication lag
- reading your own writes
	- can write to leader but read from follower with lag (to the user it looks like request was lost)
	- implementing read-after-write consistency: 
		- read from leader (if only small portion of website editable by a user)
		- monitor replication lag and prevent read from hitting lagged ones after a write
		- timestamp (log- sequence number b/c clocks might not be realibale) the write and only read from followers who have caught up to that time
- monotonic reads (p.212 in book, 236 in pdf)
	- if reading from different replicas, state being read can move backwards in time
	- 