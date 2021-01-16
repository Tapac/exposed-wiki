### Q: [Squash](https://github.com/orangy/squash) is same as Exposed. Where is the difference? 
A: [Ilya Ryzhenkov](https://github.com/orangy/) (Squash maintainer) answers: 
> Squash is an attempt to refactor Exposed (long time ago) to fix DSL issues, extensibility on dialect side, support graph fetching and avoid TLS-stored transactions. Unfortunately, I didn’t have enough time to finish the work, but I still hope to return to it some day. We are talking with Exposed maintainer [@tapac](https://github.com/orangy/) about coordinating efforts and eventually joining forces. Note, that none of these libs are “official” JetBrains Kotlin SQL libs, they are both side projects of their respective authors.

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
Use QueryBuiler with `false` - if you want to inline statement arguments, `true` - to see '?' in query.

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

A: Yes. See example here: https://github.com/JetBrains/Exposed/wiki/DSL#update

### Q: How can I add another type of Database?

A: Implement `DatabaseDialect` interface and register it with `Database.registerDialect()`.  
If the implementation adds a lot of value consider contributing it as a PR to Exposed.

### Q: Is it possible to create tables with cross / cyclic reference?

A: Yes, it's possible since Exposed 0.11.1 version

### Q: How can I implement nested queries?

A: See example here: https://github.com/JetBrains/Exposed/issues/248

### Q: How can I use SAVEPOINT?
A: It possible only through using a raw connection. See example [here](https://github.com/JetBrains/Exposed/issues/320#issuecomment-394825415).

### Q: How to prepare query like: `SELECT * FROM table WHERE (x,y) IN ((1, 2), (3, 4), (5, 6))`
A: It possible with custom function. See [example](https://github.com/JetBrains/Exposed/issues/373#issuecomment-414123325).

### Q: Where can I find snapshot builds of Exposed
A: You could use jitpack.io service for that. 

Add jitpack.io to repositories:
```
repositories {
    maven { url 'https://jitpack.io' }
}
```
Then add Exposed dependency as stated below:
```
dependencies {
    implementation 'com.github.JetBrains:Exposed:-SNAPSHOT'
}
```

### Q: How can I specify a primary key column type e.g StringIdTable?
A: You need to define your own! See examples:  
[#855](https://github.com/JetBrains/Exposed/issues/855)  
https://stackoverflow.com/a/61940820/1155026

### More questions on Stack Overflow:
https://stackoverflow.com/questions/tagged/kotlin-exposed