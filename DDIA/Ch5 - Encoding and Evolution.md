
Intro
- code will evolve. Due to rollouts or client-side apps, you'll need backwards and forwards compatibility
- forwards compatibility needs: preserve unknown fields

JSON
- in-memory object to storage format is called encoding
- beware encoding large numbers in JSON, XML, CSV b/c they don't store format (large numbers might overflow on read)
- JSON can set constraints on fields (patternProperties), enforce schema (additionalProperties=false blocks fields that aren't explicitly defined), can also if/else schema logic and use references
	- GPT says by using default additionalProperties=true, you keep fields optional, which helps with forwards compatibility. Other tips: not narrowing field type, not removing enum value, not renaming a field

Binary encoding (speed): Protocol Buffers, Avro
- how does the reader know which schema the writer used?
	- can attach schema pointer and version number, or initiate when handshaking

Dataflow through DB
- You can think of storing data in a DB as sending a message to your feature self. Therefore forward and backward compatibility also important
- schema migration for compatibility is a pain b/c saved past snapshots and too expensive to do at once

data flow through services
- GET requests data, POST submits data
- service design philosophy REST emphasizes simple data formats, using URLs for identifying resources, HTTP for cacahe control, auth, and content type negotiation
	- Interface Definition Language, ex. Swagger (OpenAPI) for web services and Protocol Buffers for gRPC, documents service's API endpoints and data models
- Service framework (ex. Spring Boot, FastAPI, gRPC) simplifies implementing the service by handling routing, metrics, caching, auth etc.
	- FastAPI generates IDL automatically
- Remote Procedure Call tries to make a request to a remote network service look the same as calling a function within the same process
	- network might have problems, timeout (retry might accidentally dupe if the first call didn't actually fail), can't pass references
	- ^ therefore fundamentally flawed, REST treats state transfer differently than a func
- Service discovery (where is the endpoint) for load balancing
	- software load balancer (ex. NGINX)
	- DNS caches entries, which migh go stale
	- Service discovery systems: centralized registry like ZooKeeper tracks which service endpoints available, monitors heartbeats
	- Service mesh (ex. Istio, Linkerd, more complicated): software load balancer + service discovery systems, run both on client and server 
- Can assume RPC servers updated first, clients second, therefore only need backward compatibility on requests, forward compatibility on responses

Durable Execution
- def workflow: graph of tasks
- workflow engine many types (ex. Airflow, Dagster, Prefect integrate with data systems for ETL tasks)
- Temporal and Restate provide durable execution for transactions: on re-execution don't rerun prev successful steps (so won't charge credit card twice for example), implemented using a write-ahead log
	- can't reorder function calls or you'll break it (solution is to deploy new version separately), nondeterministic code also problamatic

Event-Driven Architectures
- Sender doesn't wait for recipient to process the event (async send), events also not sent directly but through a message broker (ex. kafka, SQS, RabbitMQ)
	- broker acts as service discovery, buffer, and handles redelivery if process crashes
	- two models: queues and topic subscription
		- queue: process adds message to queue, consumers take individual messages from that queue
		- topic: process publishes messages to a named topic, broker delivers that message to all subscribers of that topic
	- 