## Download

### Maven

```xml
<!-- Versions after 0.30.1 -->
<repositories>
    <repository>
        <id>mavenCentral</id>
        <name>mavenCentral</name>
        <url>https://repo1.maven.org/maven2/</url>
    </repository>
</repositories>

<!-- Versions before 0.30.1 -->
<repositories>
    <repository>
        <id>jcenter</id>
        <name>jcenter</name>
        <url>https://jcenter.bintray.com</url>
    </repository>
</repositories>


<dependencies>
    <dependency>
      <groupId>org.jetbrains.exposed</groupId>
      <artifactId>exposed-core</artifactId>
      <version>0.37.3</version>
    </dependency>
    <dependency>
      <groupId>org.jetbrains.exposed</groupId>
      <artifactId>exposed-dao</artifactId>
      <version>0.37.3</version>
    </dependency>
    <dependency>
      <groupId>org.jetbrains.exposed</groupId>
      <artifactId>exposed-jdbc</artifactId>
      <version>0.37.3</version>
    </dependency>
</dependencies>

```

### Gradle Kotlin Script

如果您使用的是旧版本的 Gradle，请将以下内容添加到您的 `build.gradle` 文件中.

```
repositories {
  // Versions after 0.30.1
  mavenCentral()
  
  // Versions before 0.30.1
  jcenter()
}
dependencies {
  implementation("org.jetbrains.exposed", "exposed-core", "0.37.3")
  implementation("org.jetbrains.exposed", "exposed-dao", "0.37.3")
  implementation("org.jetbrains.exposed", "exposed-jdbc", "0.37.3")
}
```

如果您使用的是较新版本的 Gradle，您可以将以下内容添加到您的 `build.gradle.kts`.

```
val exposedVersion: String by project
dependencies {
    implementation("org.jetbrains.exposed:exposed-core:$exposedVersion")
    implementation("org.jetbrains.exposed:exposed-dao:$exposedVersion")
    implementation("org.jetbrains.exposed:exposed-jdbc:$exposedVersion")
}
```

以及在您的 `gradle.properties` 文件中添加版本号

```
exposedVersion=0.37.3
```

- 注意: 还有一些其他模块。详细信息位于[[Modules 模块文档|LibDocumentation]]部分.

## 入门

### 开启一个事务

使用 Exposed 的每个数据库访问都是通过获取连接和创建事务来启动的。

获取连接:

```kotlin
Database.connect("jdbc:h2:mem:test", driver = "org.h2.Driver")
```

也可以通过 `javax.sql.DataSource` 来提供连接池等高级行为

```kotlin
Database.connect(dataSource)
```

详情见 [[数据库和数据源|DataBase-and-DataSource]]

获得连接后，所有 SQL 语句都应该放在一个事务中:

```kotlin
transaction {
  // Statements here
}
```

想要查看实际的数据库调用，请添加一个记录器logger:

```kotlin
transaction {
  // print sql to std-out
  addLogger(StdOutSqlLogger)
}
```

### DSL & DAO

Exposed 有两种形式：DSL（领域特定语言）和 DAO（数据访问对象）.  
On a high level, DSL means type-safe syntax that is similar to SQL whereas DAO means doing CRUD operations on entities.
在上层，DSL 意味着类似于 SQL 的类型安全语法，而 DAO 意味着对实体进行 CRUD 操作.
观察以下示例，并前往每个 API 的特定部分了解更多详细信息.

### Your first Exposed DSL

```kotlin


fun main(args: Array<String>) {
  // 一个连接 H2 数据的示例
  Database.connect("jdbc:h2:mem:test", driver = "org.h2.Driver")

  transaction {
    // 将 sql 打印到标准输出
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

详见 [[DSL API|DSL]]

### Your first Exposed DAO

```kotlin


fun main(args: Array<String>) {
  // 一个连接 H2 数据的示例
  Database.connect("jdbc:h2:mem:test", driver = "org.h2.Driver")

  transaction {
    // 将 sql 打印到标准输出
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

详见 [[DAO API|DAO]]

或者... 返回 [[简介|Home]]
