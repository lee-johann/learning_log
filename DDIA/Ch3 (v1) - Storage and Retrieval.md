- indexes incur overhead during write to hopefully speed up reads

hash index: store hashmap in-memory that maps from key to location of value
	- pro: no I/O b/c in-memory; con: keys need to fit in memory, range queries bad performance. Therefore: good if many writes per key, but few keys
	- snapshots for recovery
- how to keep append-only log from getting too big? Compaction (throw away duplicate keys, only keep most recent value per key)
- why is append only good?
	- sequential access faster than random access
	- concurrency and crash recovery easier if append only

SSTable & LSM Tree
- string sorted table: same as before but sort keys
	- advantage of hash index: don't need to keep key index (even faster if you store some of the keys' corresponding locations)
	- cons: writes now need to look for where to write
	- on write since we scan multiple keys anyways, can run compaction
	- use any in-memory tree (memtable) to maintain the sorted-ness
	- when memtable gets large, write it out to disk (keep log of memtable for recovery) as SStable file
	- to find a key, check memtable first, then next SSTable, then the one after
	- Lucene, index engine of Elasticsearch, uses smth similar to store word to all docs with the word
- Performance
	- can be slow if looking up key that doesn't exist
		- to fix, use bloom filter (approximation of what's in a set)
- LSM tree basic idea: cascade of SSTables that are merged in the background

B-tree (bost common index)
- standard index implementation of all relational 
- breaks db down into fixed (instead of variable) sized blocks (pages)
- structured likke [ref | boundary | ref | boundary | ... ref] , each ref points to a child with a similar structure (the boundaries indicate the range of the keys)
	- leafs contain value or ref to pages where value can be found
	- number of ref... is called branching factor (typically several hundred)
- upon add either modify leaf directly, or if leaf gets too big split leaf then update its parent
- note that write process is an overwrite instead of append
	- dangerous if crash in the middle, therefore keep a write-ahead log: append only file to record modifications before they're done
	- also needs concurrency control

B-tree vs LSM tree
- LSM > B
	- on write B needs to write twice (once to write-ahead log, once to tree)
		- LSM also writes multiple times during merging
	- on write B needs to write the whole page
		- LSM can write sequentially (so diff bigger on hard drives)
	- B tree higher storage fragmentation (unused disk space from page that isn't full)
- B > LSM
	- LSM compaction can steal resources from incoming writes
	- B tree 1 source of truth per key, so good for transactinoal
other indexes
- secondary indexes
	- value of index sometimes is location of row (known as heap file) instead of actual row to avoid duplicating the data for every secondary index
	- alternatively can store the row's values, this slows down writes (concurrency guarantees) and incurs storage cost, but speeds up reads
- multi-column index: several concatenated keys map to a value (ex. first-last -> number)
	- for this, you can use the index to find all first or all first-last but the index doesn't help with all last
	- implemented with R-trees (unimportant)
- fuzzy indexes (not covered in this book)
- in-memory DB (maintain durability by append-only log to disk, reads served from memory) ex. redis

Transactoin processing vs analytics
- 