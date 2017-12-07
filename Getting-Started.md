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
    <version>0.9.1</version>
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
  compile 'org.jetbrains.exposed:exposed:0.9.1'
}
```

* Note: There is a separate artifact for `spring-transaction`

## Your first Exposed DAO

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