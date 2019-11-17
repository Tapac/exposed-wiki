## Dependencies
Exposed modules available from JFrog Bintray and JCenter repositories.
To use them you have to add appropriate dependency into your repositories mapping.

#### Maven
```xml
<repositories>
  <repository>
    <id>jcenter</id>
    <name>jcenter</name>
    <url>http://jcenter.bintray.com</url>
  </repository>
</repositories>
```

#### Gradle Groovy and Kotlin DSL

```kotlin
repositories {
  jcenter()
}
```

## Base Modules
### Exposed 0.17.x and lower
Prior Exposed 0.18.1 there was only one base module `exposed` which contains everything you may need including JodaTime as date-time library.
To add `Exposed` framework of that version you had to use: 
    
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
  compile 'org.jetbrains.exposed:exposed:0.17.7'
}
```
#### Gradle Kotlin DSL
```kotlin
dependencies {
    compile("org.jetbrains.exposed", "exposed", "0.17.7")
}
```

### Exposed 0.18.1 and higher
To move forward and support such features as Java 8 Time, async drivers and so on it was decided to split Exposed into more specific modules. It will allow you to take the only modules you need and will add flexibility in the future.

At the moment `Exposed` coexists of five modules
* exposed-core - base module, which contains both DSL api along with mapping
* exposed-dao - DAO api 
* exposed-jdbc - transport level implementation based on Java JDBC API
* exposed-jodatime - date-time extensions based on JodaTime library
* exposed-java-time - date-time extensions based on Java8 Time API

Dependencies mapping listed bellow is similar (by functionality) to the previous versions:
#### Maven
```xml

<dependencies>
  <dependency>
    <groupId>org.jetbrains.exposed</groupId>
    <artifactId>exposed-core</artifactId>
    <version>0.18.1</version>
  </dependency>
  <dependency>
    <groupId>org.jetbrains.exposed</groupId>
    <artifactId>exposed-dao</artifactId>
    <version>0.18.1</version>
  </dependency>
  <dependency>
    <groupId>org.jetbrains.exposed</groupId>
    <artifactId>exposed-jdbc</artifactId>
    <version>0.18.1</version>
  </dependency>
  <dependency>
    <groupId>org.jetbrains.exposed</groupId>
    <artifactId>exposed-jodatime</artifactId>
    <version>0.18.1</version>
  </dependency>
</dependencies>

```

#### Gradle Groovy
```groovy
dependencies {
  compile 'org.jetbrains.exposed:exposed-core:0.18.1'
  compile 'org.jetbrains.exposed:exposed-dao:0.18.1'
  compile 'org.jetbrains.exposed:exposed-jdbc:0.18.1'
  compile 'org.jetbrains.exposed:exposed-jodatime:0.18.1'
}
```
#### Gradle Kotlin DSL
```kotlin
dependencies {
    compile("org.jetbrains.exposed", "exposed-core", "0.18.1")
    compile("org.jetbrains.exposed", "exposed-dao", "0.18.1")
    compile("org.jetbrains.exposed", "exposed-jdbc", "0.18.1")
    compile("org.jetbrains.exposed", "exposed-jodatime", "0.18.1")
}
```

