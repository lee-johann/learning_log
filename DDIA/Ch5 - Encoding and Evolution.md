
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

Dataflow through DB (p.178)