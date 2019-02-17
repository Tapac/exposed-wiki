### Working with DataBase and DataSource
Every database access using Exposed is starting by obtaining a connection and creating a transaction.  

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

### DataSource

* PostgreSQL
```Kotlin
Database.connect("jdbc:postgresql://localhost:12346/test", driver = "org.postgresql.Driver", 
                 user = "root", password = "your_pwd")  
//Gradle
compile "org.postgresql:postgresql:42.2.2"  
```
* MySQL
```Kotlin
Database.connect("jdbc:mysql://localhost:3306/test", driver = "com.mysql.jdbc.Driver", 
                 user = "root", password = "your_pwd")  
//Gradle
compile "mysql:mysql-connector-java:5.1.46"  
```
* Oracle
```Kotlin
Database.connect("jdbc:jdbc:oracle:thin:@//localhost:1521/test", driver = "oracle.jdbc.OracleDriver", 
                 user = "root", password = "your_pwd")  
//Gradle
// Oracle jdbc-driver should be obtained from Oracle maven repo: https://blogs.oracle.com/dev2dev/get-oracle-jdbc-drivers-and-ucp-from-oracle-maven-repository-without-ides
```
+ SQLite
```Kotlin
// In file
Database.connect("jdbc:sqlite:/data/data.db", "org.sqlite.JDBC")  
// In memory
Database.connect("jdbc:sqlite:file:test?mode=memory&cache=shared", "org.sqlite.JDBC")  
//Gradle
compile "org.xerial:sqlite-jdbc:3.21.0.1"  
```  
* H2
```Kotlin
// In memory
Database.connect("jdbc:h2:mem:regular", "org.h2.Driver")  
// In memory / keep alive between connections/transactions
Database.connect("jdbc:h2:mem:regular;DB_CLOSE_DELAY=-1;", "org.h2.Driver")  
//Gradle
compile "com.h2database:h2:1.4.197"  
```  
* SQL Server
```Kotlin
Database.connect("jdbc:sqlserver://localhost:32768;databaseName=test", "com.microsoft.sqlserver.jdbc.SQLServerDriver", 
                 user = "root", password = "your_pwd")  
//Gradle
compile "com.microsoft.sqlserver:mssql-jdbc:6.4.0.jre7"  
```  