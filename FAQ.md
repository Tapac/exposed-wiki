### Q: Can I use multiple Database Connections?

A: Yes. At the moment the connection usage is implicit (ie: via `ThreadLocal`), and it is not possible to specify connection explicitly. So switching between connections is not supported at the moment. There is an open issue about it: https://github.com/JetBrains/Exposed/issues/93.  

### Q: Is `Array` column type supported?

A: Not at the moment. More info here: https://github.com/JetBrains/Exposed/issues/150  
The complete list of supported data types can be found here: [[Data Types|DataTypes]].

### Q: Is `upsert` supported?

A: Upsert is an instruction to the Database to insert a new row or update existing row based on a table key. It is not supported as part of the library but it is possible to implement on top of it. See this issue: https://github.com/JetBrains/Exposed/issues/167 and example here: https://medium.com/@OhadShai/first-steps-with-kotlin-exposed-cb361a9bf5ac

### Q: Is `json` type supported?

A: Not at the moment. Here is the issue: https://github.com/JetBrains/Exposed/issues/127  
The complete list of supported data types can be found here: [[Data Types|DataTypes]].

### Q: Is it possible to use native sql / sql as a string?

A: It is not supported as part of the library but it is possible to implement on top of it and use it like this:
```
fun <T:Any> String.execAndMap(transform : (ResultSet) -> T) : List<T> {
     val result = arrayListOf<T>()
     TransactionManager.current().exec(this) { rs ->
          while (rs.next()) {
               result += transform(rs)
          }
     }
     return result
}

"select u.name, c.name from user u inner join city c where blah blah".execAndMap { rs ->
    rs.getString("u.name") to rs.getString("c.name") 
}
```
More info in this issue: https://github.com/JetBrains/Exposed/issues/118

### Q: Is it possible to update a field relative to current field value?

A: Yes. See example here: https://github.com/JetBrains/Exposed/issues/82

### Q: How can I add another type of Database?

A: Implement `DatabaseDialect` interface and register it with `Database.registerDialect()`.  
If the implementation adds a lot of value consider contributing it as a PR to Exposed.

### Q: Is it possible to create tables with cross / cyclic reference?

A: No. Its possible in sql directly in couple of steps. More details here: https://github.com/JetBrains/Exposed/issues/185

### More questions on Stack Overflow:
https://stackoverflow.com/questions/tagged/kotlin-exposed