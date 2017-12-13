### Q: Can I use multiple Database Connections?

A: Yes. At the moment the connection usage is implicit (ie: via `ThreadLocal`), and it is not possible to specify connection explicitly. So switching between connections is not supported at the moment. There is an open issue about it: https://github.com/JetBrains/Exposed/issues/93.  

### Q: Is Array column type supported?

A: Not at the moment. More info here: https://github.com/JetBrains/Exposed/issues/150

### Q: Is upsert supported?

A: Upsert is an instruction to the Database to insert a new row or update existing row based on a table key. It is not supported as part of the library but it is possible to implement on top of it. See this issue: https://github.com/JetBrains/Exposed/issues/167 and example here: https://medium.com/@OhadShai/first-steps-with-kotlin-exposed-cb361a9bf5ac

### Q: Is json type supported?

A: Not at the moment. Here is the issue: https://github.com/JetBrains/Exposed/issues/127
