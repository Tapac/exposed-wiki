## Download

### Maven

```xml
<!-- Versions after 0.30.1 -->
<!-- Versions before 0.30.1 is unavailable for now-->
<repositories>
    <repository>
        <id>mavenCentral</id>
        <name>mavenCentral</name>
        <url>https://repo1.maven.org/maven2/</url>
    </repository>
</repositories>

<dependencies>
    <dependency>
      <groupId>org.jetbrains.exposed</groupId>
      <artifactId>exposed-core</artifactId>
      <version>0.38.2</version>
    </dependency>
    <dependency>
      <groupId>org.jetbrains.exposed</groupId>
      <artifactId>exposed-dao</artifactId>
      <version>0.38.2</version>
    </dependency>
    <dependency>
      <groupId>org.jetbrains.exposed</groupId>
      <artifactId>exposed-jdbc</artifactId>
      <version>0.38.2</version>
    </dependency>
</dependencies>

```

### Gradle Kotlin Script

If you're using older version of Gradle, add the following to your `build.gradle` file.

```
repositories {
  // Versions after 0.30.1
  // Versions before 0.30.1 is unavailable for now
  mavenCentral()
}
dependencies {
  implementation("org.jetbrains.exposed", "exposed-core", "0.38.2")
  implementation("org.jetbrains.exposed", "exposed-dao", "0.38.2")
  implementation("org.jetbrains.exposed", "exposed-jdbc", "0.38.2")
}
```

If you're using newer versions of Gradle, you can add the following to your `build.gradle.kts`.

```
val exposedVersion: String by project
dependencies {
    implementation("org.jetbrains.exposed:exposed-core:$exposedVersion")
    implementation("org.jetbrains.exposed:exposed-dao:$exposedVersion")
    implementation("org.jetbrains.exposed:exposed-jdbc:$exposedVersion")
}
```

And the version in your `gradle.properties`

```
exposedVersion=0.38.2
```

- Note: There are another modules. Detailed information located in [[Modules Documentation|LibDocumentation]] section.

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

Exposed comes in two flavors: DSL (Domain Specific Language) and DAO (Data Access Object).  
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
