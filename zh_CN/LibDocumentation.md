## 依赖Dependencies
Exposed 模块可从 Maven Central（旧版本的 JFrog Bintray 和 JCenter）存储库中获得.
要使用它们，您必须在repositories存储库映射中添加适当的依赖项.

#### Maven
```xml
<!-- 0.30.1 之后的版本 -->
<repositories>
    <repository>
        <id>mavenCentral</id>
        <name>mavenCentral</name>
        <url>https://repo1.maven.org/maven2/</url>
    </repository>
</repositories>

<!-- 0.30.1 之前的版本 -->
<repositories>
    <repository>
        <id>jcenter</id>
        <name>jcenter</name>
        <url>https://jcenter.bintray.com</url>
    </repository>
</repositories>
```

#### Gradle Groovy and Kotlin DSL

```kotlin
repositories {
    // 0.30.1 之后的版本
    mavenCentral()

    // 0.30.1 之前的版本
    jcenter()
}
```

## 基础模块
### Exposed 0.17.x 以及较低版本
Exposed 0.18.1 之前的版本, 只有一个基本模块`exposed`，其中包含您可能需要的所有内容，包括 JodaTime 作为date-time库.
你必须使用如下方式添加该版本的`Exposed`框架:
    
#### Maven
```xml
<dependencies>
  <dependency>
    <groupId>org.jetbrains.exposed</groupId>
    <artifactId>exposed</artifactId>
    <version>0.17.7</version>
  </dependency>
</dependencies>

```

#### Gradle Groovy
```groovy
dependencies {
  implementation 'org.jetbrains.exposed:exposed:0.17.7'
}
```
#### Gradle Kotlin DSL
```kotlin
dependencies {
    implementation("org.jetbrains.exposed", "exposed", "0.17.7")
}
```

### Exposed 0.18.1 以及较高版本
为了向前推进和支持 Java 8 Time、异步驱动程序等特性，决定将 Exposed 拆分为更具体的模块。它将允许您使用您需要的唯一模块，并将在未来增加灵活.

目前 `Exposed` 提供的共存模块如下
* exposed-core - 基本模块, 其中包含 DSL api 和 mapping映射
* exposed-dao - DAO api 
* exposed-jdbc - 基于 Java JDBC API 的传输层实现
* exposed-jodatime - 基于 JodaTime 库的 date-time 扩展功能
* exposed-java-time - 基于 Java8 Time API的 date-time 扩展功能
* exposed-kotlin-datetime - 基于 kotlinx-datetime的 date-time 扩展功能
* exposed-money - 从`javax.money:money-api` 获取对 MonetaryAmount 的扩展支持

下面列出的依赖关系映射与以前的版本相似（按功能）:
#### Maven
```xml

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
  <dependency>
    <groupId>org.jetbrains.exposed</groupId>
    <artifactId>exposed-jodatime</artifactId>
    <version>0.37.3</version>
  </dependency>
  <dependency>
    <groupId>org.jetbrains.exposed</groupId>
    <artifactId>exposed-java-time</artifactId>
    <version>0.37.3</version>
  </dependency>
</dependencies>

```

#### Gradle Groovy
```groovy
dependencies {
  implementation 'org.jetbrains.exposed:exposed-core:0.37.3'
  implementation 'org.jetbrains.exposed:exposed-dao:0.37.3'
  implementation 'org.jetbrains.exposed:exposed-jdbc:0.37.3'
  implementation 'org.jetbrains.exposed:exposed-jodatime:0.37.3'
  // or
  implementation 'org.jetbrains.exposed:exposed-java-time:0.37.3'
}
```
#### Gradle Kotlin DSL
在 `build.gradle.kts` 中的内容:
```kotlin
val exposedVersion: String by project
dependencies {
    implementation("org.jetbrains.exposed:exposed-core:$exposedVersion")
    implementation("org.jetbrains.exposed:exposed-dao:$exposedVersion")
    implementation("org.jetbrains.exposed:exposed-jdbc:$exposedVersion")
    implementation("org.jetbrains.exposed:exposed-jodatime:$exposedVersion")
    // or
    implementation("org.jetbrains.exposed:exposed-java-time:$exposedVersion")
}
```
在 `gradle.properties` 中的内容:
```
exposedVersion=0.37.3
```

### JDBC driver and logging
您还需要为正在使用的数据库系统提供 JDBC 驱动程序 (详见 [[数据库和数据源|数据库和数据源]]) 和 一个日志实现 `addLogger(StdOutSqlLogger)`. 例如 (Gradle 语法):
```kotlin
dependencies {
    // for H2
    implementation("com.h2database:h2:1.4.199")
    // for logging (StdOutSqlLogger), see
    // http://www.slf4j.org/codes.html#StaticLoggerBinder
    implementation("org.slf4j:slf4j-nop:1.7.30")
}
```

或者... 返回 [主页](Home.md)