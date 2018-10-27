## Download

### Maven

```xml
<repositories>
  <repository>
    <id>jcenter</id>
    <name>jcenter</name>
    <url>http://jcenter.bintray.com</url>
  </repository>
</repositories>

<dependencies>
  <dependency>
    <groupId>org.jetbrains.exposed</groupId>
    <artifactId>exposed</artifactId>
    <version>0.11.1</version>
  </dependency>
</dependencies>

```

### Gradle

```
repositories {
  jcenter()
}
dependencies {
  compile 'org.jetbrains.exposed:exposed:0.11.1'
}
```

* Note: There is a separate artifact for `spring-transaction`

## Getting Started

### Starting a transaction

Every database access using Exposed is started by obtaining a connection and creating a transaction.  

To get a connection:

```kotlin
Database.connect("jdbc:h2:mem:test", driver = "org.h2.Driver")
```

It is also possible to provide `javax.sql.DataSource` for advanced behaviors such as connection pooling:
```kotlin
Database.connect(dataSource)
```

More details on [[DataBase and DataSource|DataBase-and-DataSource]]

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
  addLogger(StdOutSqlLogger)
} 
```

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
    addLogger(StdOutSqlLogger)
    
    SchemaUtils.create (Cities)

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
    addLogger(StdOutSqlLogger)
    
    SchemaUtils.create (Cities)

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