### Q: Can I use multiple Database Connections?

A: Yes. See [[Working with a multiple databases|Transactions#working-with-a-multiple-databases]]

### Q: Is `Array` column type supported?

A: Not at the moment. More info here: https://github.com/JetBrains/Exposed/issues/150  
The complete list of supported data types can be found here: [[Data Types|DataTypes]].

### Q: Is `upsert` supported?

A: Upsert is an instruction to the Database to insert a new row or update existing row based on a table key. It is not supported as part of the library but it is possible to implement on top of it. See this issue: https://github.com/JetBrains/Exposed/issues/167 and example here: https://medium.com/@OhadShai/first-steps-with-kotlin-exposed-cb361a9bf5ac

### Q: Is `json` type supported?

A: Not at the moment. Here is the issue: https://github.com/JetBrains/Exposed/issues/127  
The complete list of supported data types can be found here: [[Data Types|DataTypes]].

### Q: How to get a plain SQL query which will be executed?

A: 
```kotlin
val plainSQL = FooTable.select {}.prepareSQL(QueryBuilder(false)) 
```
Use QueryBuiler with `false` - if you want to inline statement arguments, true - to see '?' in query.

### Q: Is it possible to use native sql / sql as a string?

A: It is not supported as part of the library but it is possible to implement on top of it and use it like this:
```kotlin
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

A: No. Its possible in sql directly in a couple of steps. More details here: https://github.com/JetBrains/Exposed/issues/185

### Q: How can I implement nested queries?

A: See example here: https://github.com/JetBrains/Exposed/issues/248

### Q: SQLite3 fails with the error Transaction attempt #0 failed: SQLite supports only TRANSACTION_SERIALIZABLE and TRANSACTION_READ_UNCOMMITTED.

A: Make your connect code look like below (taken from [ThreadLocalManagerTest.kt](https://github.com/JetBrains/Exposed/blob/4360ae18c708072dfda261742e65b8b56a696adc/src/test/kotlin/org/jetbrains/exposed/sql/tests/shared/ThreadLocalManagerTest.kt#L282-L291)).  The setting below tells SQLite3 to enforce transactional integrity by [serializing writes from different connections](https://sqlite.org/isolation.html).
```
Database.connect("jdbc:sqlite:my.db", "org.sqlite.JDBC")
TransactionManager.manager.defaultIsolationLevel = Connection.TRANSACTION_SERIALIZABLE
```


### More questions on Stack Overflow:
https://stackoverflow.com/questions/tagged/kotlin-exposed