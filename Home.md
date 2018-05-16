Welcome to the Exposed wiki! (Still Under Construction)

Exposed is a lightweight SQL library on top of JDBC driver for Kotlin language.  
Exposed has two flavors of database access: typesafe SQL wrapping DSL and lightweight Data Access Objects (DAO)

This wiki contains the following pages:

  * [[Introduction|Home]] - This page
  * [[Getting Started|Getting-Started]]
  * [[DSL API|DSL]]
  * [[DAO API|DAO]]
  * [[Modules Documentation|LibDocumentation]]
  * [[Frequently Asked Questions|FAQ]]
  * [[Contributing|Contributing]]


Currently supported database dialects are:
> Gray is the database connection  

* PostgreSQL
* MySQL
    > Code:  
    > Database.connect("jdbc:mysql://localhost:3306/niuniu_server",driver = "org.sqlite.JDBC", user = "root", password = "your_pwd")  
    > Gradle:  
    > maven{ url 'https://mvnrepository.com/artifac/'}
    > compile "mysql:mysql-connector-java:5.1.46"
* Oracle
+ SQLite  
    > Code:  
    > Database.connect("jdbc:sqlite:/data/data.db", "org.sqlite.JDBC")  
    > Gradle:  
    > maven{ url 'https://mvnrepository.com/artifac/'}  
    > compile group: 'org.xerial', name: 'sqlite-jdbc', version: '3.7.2'
* H2
* SQL Server
