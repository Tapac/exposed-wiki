## Download

### Maven

```xml
<repositories>
  <repository>
    <id>exposed</id>
    <name>exposed</name>
    <url>https://dl.bintray.com/kotlin/exposed</url>
  </repository>
</repositories>

<dependencies>
  <dependency>
    <groupId>org.jetbrains.exposed</groupId>
    <artifactId>exposed</artifactId>
    <version>0.10.2</version>
  </dependency>
</dependencies>

```

### Gradle

```
repositories {
  maven {
    url  "https://dl.bintray.com/kotlin/exposed" 
  }
}
dependencies {
  compile 'org.jetbrains.exposed:exposed:0.10.2'
}
```

* Note: There is a separate artifact for `spring-transaction`

## Getting Started

### Starting a transaction

Every database access using Expose is starting by obtaining a connection and creating a transaction.  

To get a connection:

```kotlin
Database.connect("jdbc:h2:mem:test", driver = "org.h2.Driver")
```

It is also possible to provide `javax.sql.DataSource` for advanced behaviors such as connection pooling:
```kotlin
Database.connect(dataSource)
```

After obtaining a connection all SQL statements should be placed inside a transaction:
```kotlin
transaction {
  // Statements here
}
```

To see the actual DB calls, add a logger:
```kotlin
transaction {
  // print sql to std-out
  logger.addLogger(StdOutSqlLogger)
} 
```

### DataSource

* PostgreSQL
* MySQL
    > Code:  
    > Database.connect("jdbc:mysql://localhost:3306/niuniu_server",driver = "com.mysql.jdbc.Driver", user = "root", password = "your_pwd")  
    > Gradle:  
    > maven{ url 'https://mvnrepository.com/artifac/'}  
    > compile "mysql:mysql-connector-java:5.1.46"  
* Oracle
+ SQLite  
    > Code:  
    > Database.connect("jdbc:sqlite:/data/data.db", "org.sqlite.JDBC")  
    > Gradle:  
    > maven{ url 'https://mvnrepository.com/artifac/'}  
    > compile group: 'org.xerial', name: 'sqlite-jdbc', version: '3.21.0.1'  
* H2
* SQL Server

### DSL & DAO 

Expose comes in two flavors: DSL (Domain Specific Language) and DAO (Data Access Object).  
On a high level, DSL means type-safe syntax that is similar to SQL whereas DAO means doing CRUD operations on entities.  
Observe the below examples and head on to the specific section of each API for more details.

### Your first Exposed DSL

```kotlin


fun main(args: Array<String>) {
  //an example connection to H2 DB  
  Database.connect("jdbc:h2:mem:test", driver = "org.h2.Driver")

  transaction {
    // print sql to std-out
    logger.addLogger(StdOutSqlLogger)
    
    // insert new city. SQL: INSERT INTO Cities (name) VALUES ('St. Petersburg')
    val stPeteId = Cities.insert {
      it[name] = "St. Petersburg"
    } get Cities.id
    
    // 'select *' SQL: SELECT Cities.id, Cities.name FROM Cities
    println("Cities: ${Cities.selectAll()}")
  }
}

object Cities: IntIdTable() {
    val name = varchar("name", 50)
}

```
More on [[DSL API|DSL]]
### Your first Exposed DAO

```kotlin


fun main(args: Array<String>) {
  //an example connection to H2 DB  
  Database.connect("jdbc:h2:mem:test", driver = "org.h2.Driver")

  transaction {
    // print sql to std-out
    logger.addLogger(StdOutSqlLogger)
    
    // insert new city. SQL: INSERT INTO Cities (name) VALUES ('St. Petersburg')
    val stPete = City.new {
            name = "St. Petersburg"
    }
    
    // 'select *' SQL: SELECT Cities.id, Cities.name FROM Cities
    println("Cities: ${City.all()}")
  }
}

object Cities: IntIdTable() {
    val name = varchar("name", 50)
}

class City(id: EntityID<Int>) : IntEntity(id) {
    companion object : IntEntityClass<City>(Cities)

    var name by Cities.name
}
```
More on [[DAO API|DAO]]

Or... back to [[Introduction|Home]]