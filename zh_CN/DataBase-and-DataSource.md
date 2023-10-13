### 使用数据库和数据源 datasource
使用 Exposed 的每个数据库访问都是通过获取连接和创建事务来启动的.
首先，您必须告诉 Exposed 如何使用 `Database.connect` 函数连接到数据库. 
它不会创建真正的数据库连接，而只会提供一个描述符descriptor以供将来使用.

将通过调用事务 `transaction` lambda函数, 稍后来实例化一个真正的数据库连接（详情请阅 [事务](https://github.com/JetBrains/Exposed/wiki/Transactions)）.

通过简单提供连接参数, 来获取数据库实例:
```kotlin
val db = Database.connect("jdbc:h2:mem:test", driver = "org.h2.Driver")
```
也可以通过 `javax.sql.DataSource` 来提供连接池等高级行为:
```kotlin
val db = Database.connect(dataSource)
```
* 注意：从 Exposed 0.10 开始，在您的应用程序中, 每个数据库多次执行此代码会造成泄漏，因此建议将其存储起来以备后用.
例如:
```kotlin
object DbSettings {
   val db by lazy { 
       Database.connect(/* setup connection */)
   }
}
```
### 数据源 DataSource
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

或者... 返回 [主页](Home.md)