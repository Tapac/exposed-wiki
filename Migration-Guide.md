## Migrating from Exposed 0.17.x and earlier to 0.18.+
### Dependencies

See [["Exposed 0.18.1 and higher" section|LibDocumentation#exposed-0181-and-higher]] in Modules Documentation

### Code changes:
Some functions and extensions had changed their places, another became extension functions instead of member functions.
In both cases it could require re-import to be possible to compile your code again.

#### JDBC related classes 
Those classes were replaced with Exposed alternatives. Only part of original interface functions reused in new interfaces.

`java.sql.Connection` -> `org.jetbrains.exposed.sql.statements.api.ExposedConnection` 
`java.sql.Savepoint` -> `org.jetbrains.exposed.sql.statements.api.ExposedSavepoint`
`java.sql.Blob` and `javax.sql.rowset.serial.SerialBlob` -> `org.jetbrains.exposed.sql.statements.api.ExposedBlob`
`java.sql.PreparedStatement` -> `org.jetbrains.exposed.sql.statements.api.PreparedStatementApi`
`java.sql.DatabaseMetadata` -> `org.jetbrains.exposed.sql.statements.api.ExposedDatabaseMetadata` (access it with `Database.metadata { }` function)

If you need to get original jdbc connection (and you use `exposed-jdbc` as a dependency) you should cast `ExposedConnection` instance to `JdbcConnectionImpl` and get value from a `connection` field.
```kotlin
val jdbcConnection: java.sql.Conneciton = (transaction.connection as JdbcConnectionImpl).connection
```

The same goes for `java.sql.PreparedStatement` and `PreparedStatementApi` but you should cast it to `JdbcPreparedStatementImpl`
```kotlin
val exposedStatement : PreparedStatementApi = // e.g. received from a StatementInterceptor
val preparedStatement: java.sql.PreparedStatement = (exposedStatement as JdbcPreparedStatementImpl).statement
```
### JodaTime and Java Time Api
Since 0.18.1 datetime functions have their own modules for both JodaTime and Java Time implementations.
If you already use JodaTime just re-import date related functions as they become extensions.
But if you want to switch from JodaTime to Java Time you have to:
1. Use `exposed-java-time` instead of `exposed-jodatime`
2. Fix your code according to a fact what `date` column will return `java.time.LocalDate` and `datetime` -> `java.time.LocalDateTime`

## Migrating to 0.19.+
To allow Exposed to work with Java 9 module system some classes in a `exposed-core` modules were changed their packages.
Also, `exposed-jodatime` functions/classes were moved to a new package.

Affected classes:
`org.jetbrains.exposed.dao.EntityID`     -> `org.jetbrains.exposed.dao.id.EntityID` 
`org.jetbrains.exposed.dao.IdTable`      -> `org.jetbrains.exposed.dao.id.IdTable` 
`org.jetbrains.exposed.dao.IntIdTable`   -> `org.jetbrains.exposed.dao.id.IntIdTable` 
`org.jetbrains.exposed.dao.LongIdTable`  -> `org.jetbrains.exposed.dao.id.LongIdTable` 
`org.jetbrains.exposed.dao.UUIDTable`    -> `org.jetbrains.exposed.dao.id.UUIDTable` 

As this change could hardly affect large projects we move existing classes with their original package to `exposed-dao` module, deprecate them and prepare migration steps with IntelliJ IDEA:
1. Find any of deprecated tables in your project and use `Alt+Enter` quick-fix with "Replace in the whole project". Repeat for all *IdTables.
2. Run "Edit > Find > Replace in Path..."
3. Enter `import org.jetbrains.exposed.dao.*` in a search field
4. Enter in a replace field:
```
import org.jetbrains.exposed.dao.id.EntityID
import org.jetbrains.exposed.dao.*
```
5. Run "Replace" in the whole project.
6. Search for `DateColumnType`, `date()`, `datetime()` and re-import them manually.
7. Run "Optimize imports" on all files in a current change list.
8. Run "Build" and fix the rest problems.
