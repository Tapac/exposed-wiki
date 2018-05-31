### Working with DataBase and DataSource
Every database access using Expose is starting by obtaining a connection and creating a transaction.  

To get a connection:

```kotlin
val db = Database.connect("jdbc:h2:mem:test", driver = "org.h2.Driver")
```

It is also possible to provide `javax.sql.DataSource` for advanced behaviors such as connection pooling:
```kotlin
val db = Database.connect(dataSource)
```

* Note: Starting Exposed 0.10 executing this code more than once per db will create leaks in your application, hence it is recommended to store it for later use.

For example:

```kotlin
object DbSettings {
   val db by lazy { 
       Database.connect(...)
   }
}
```