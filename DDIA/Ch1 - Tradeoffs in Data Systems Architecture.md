Intro
- usually DDIA applies to backend b/c frontend handles 1 user's data, backend handles all
- backend service reachable via HTTP/websocket, requests often stateless (w/ info stored on data systems)

Operational vs analytical systems
- operational: data is created, apps read and modify data based on user actions
	- looks up small number of records by key (with restricted change permissions)
- analytics: usually read-only copy of data from operational (keep a separate system, data warehouse)
	- aggregates
	- keeping analytics separate enables optimization to usage patterns, can resolve data silo issues, and splits load between transactions and analytics
	- data lake: raw data, before ETL pipeline for warehouse
- system of record and derived data
	- sys of rec is the source of truth
	- derived data systems are all copies of the original, modified in some way to help with query performance based on need

Cloud vs self hosting
- loss of control, benefit of less hassle
- separation of storage and compute

Distributed vs single-node
- distributed torubles: network reliance, troubleshooting harder, data consistency
- microservices architecture: each service has 1 purpose (ex. s3 is storage), exposing an API for clients
	- pro: services can be updated independently, assigned different hardware it needs
	- cons: API can change, testing during development harder b/c need spin up all other services it depends on, deployment also harder
	- fundamentally it enables different teams to make progress independently (therefore might not need in a startup)
- serverless: infra outsourced to a cloud vendor (no need startup VM)
	- cold start, also might impose time limit