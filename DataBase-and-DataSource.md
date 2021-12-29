### Working with DataBase and DataSource
Every database access using Exposed is starting by obtaining a connection and creating a transaction.  
First of all, you have to tell Exposed how to connect to a database by using `Database.connect` function. 
It won't create a real database connection but only provide a descriptor for future usage.

A real connection will be instantiated later by calling `transaction` lambda (see [Transaction](https://github.com/JetBrains/Exposed/wiki/Transactions) for more details).

To get a Database instance by simple providing connection parameters:
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
       Database.connect(/* setup connection */)
   }
}
```
### DataSource
* PostgreSQL
```kotlin
Database.connect("jdbc:postgresql://localhost:12346/test", driver = "org.postgresql.Driver", 
                 user = "root", password = "your_pwd")  
//Gradle
implementation("org.postgresql:postgresql:42.2.2")  
```
* PostgreSQL using the pgjdbc-ng JDBC driver
```kotlin
Database.connect("jdbc:pgsql://localhost:12346/test", driver = "com.impossibl.postgres.jdbc.PGDriver", 
                 user = "root", password = "your_pwd")  
//Gradle
implementation("com.impossibl.pgjdbc-ng", "pgjdbc-ng", "0.8.3")  
```
* MySQL/MariaDB
```kotlin
Database.connect("jdbc:mysql://localhost:3306/test", driver = "com.mysql.cj.jdbc.Driver", 
                 user = "root", password = "your_pwd")  
//Gradle
implementation("mysql:mysql-connector-java:8.0.2")
```
* MySQL/MariaDB with latest JDBC driver + Hikari pooling
```kotlin
val config = HikariConfig().apply {
    jdbcUrl         = "jdbc:mysql://localhost/dbname"
    driverClassName = "com.mysql.cj.jdbc.Driver"
    username        = "username"
    password        = "secret"
    maximumPoolSize = 10
}
val dataSource = HikariDataSource(config)
Database.connect(dataSource)
// Gradle
implementation "mysql:mysql-connector-java:8.0.19"
implementation "com.zaxxer:HikariCP:3.4.2"
```
* Oracle
```kotlin
Database.connect("jdbc:oracle:thin:@//localhost:1521/test", driver = "oracle.jdbc.OracleDriver", 
                 user = "root", password = "your_pwd")  
//Gradle
// Oracle jdbc-driver should be obtained from Oracle maven repo: https://blogs.oracle.com/dev2dev/get-oracle-jdbc-drivers-and-ucp-from-oracle-maven-repository-without-ides
```
+ SQLite
```kotlin
// In file
Database.connect("jdbc:sqlite:/data/data.db", "org.sqlite.JDBC")  
// In memory
Database.connect("jdbc:sqlite:file:test?mode=memory&cache=shared", "org.sqlite.JDBC")  
// For both: set SQLite compatible isolation level, see 
// https://github.com/JetBrains/Exposed/wiki/FAQ
TransactionManager.manager.defaultIsolationLevel = 
    Connection.TRANSACTION_SERIALIZABLE
    // or Connection.TRANSACTION_READ_UNCOMMITTED
//Gradle
implementation("org.xerial:sqlite-jdbc:3.30.1")  
```  
* H2
```kotlin
// Database in file, needs full path or relative path starting with ./
Database.connect("jdbc:h2:./myh2file", "org.h2.Driver")
// In memory
Database.connect("jdbc:h2:mem:regular", "org.h2.Driver")  
// In memory / keep alive between connections/transactions
Database.connect("jdbc:h2:mem:regular;DB_CLOSE_DELAY=-1;", "org.h2.Driver")  
//Gradle
implementation("com.h2database:h2:1.4.199")  
```  
* SQL Server
```kotlin
Database.connect("jdbc:sqlserver://localhost:32768;databaseName=test", "com.microsoft.sqlserver.jdbc.SQLServerDriver", 
                 user = "root", password = "your_pwd")  
//Gradle
implementation("com.microsoft.sqlserver:mssql-jdbc:6.4.0.jre7")  
```
