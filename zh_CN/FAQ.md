### Q: [Squash](https://github.com/orangy/squash) 与 Exposed 类似. 它们的区别在哪? 
A: [Ilya Ryzhenkov](https://github.com/orangy/) (Squash 维护者) 回答: 
> Squash is an attempt to refactor Exposed (long time ago) to fix DSL issues, extensibility on dialect side, support graph fetching and avoid TLS-stored transactions. Unfortunately, I didn’t have enough time to finish the work, but I still hope to return to it some day. We are talking with Exposed maintainer [@tapac](https://github.com/orangy/) about coordinating efforts and eventually joining forces. Note, that none of these libs are “official” JetBrains Kotlin SQL libs, they are both side projects of their respective authors.

### Q: 我可以使用多个数据库连接吗?

A: 可以. 相见 [[Working with a multiple databases|Transactions#working-with-a-multiple-databases]]

### Q: 是否支持 `Array` column列类型?

A: 目前不行. 更多信息在这里: https://github.com/JetBrains/Exposed/issues/150  
可在此处找到支持的数据类型的完整列表: [[Data Types|DataTypes]].

### Q: 是否支持 `upsert` 操作?

A: Upsert is an instruction to the Database to insert a new row or update existing row based on a table key. It is not supported as part of the library but it is possible to implement on top of it. See this issue: https://github.com/JetBrains/Exposed/issues/167 and example here: https://medium.com/@OhadShai/first-steps-with-kotlin-exposed-cb361a9bf5ac

### Q: 是否支持 `json` 类型?

A: 目前不行. issue问题在这里: https://github.com/JetBrains/Exposed/issues/127  
可在此处找到支持的数据类型的完整列表: [[Data Types|DataTypes]].

### Q: 如何获取将执行的普通 SQL 查询?

A: 
```kotlin
val plainSQL = FooTable.select {}.prepareSQL(QueryBuilder(false)) 
```
使用QueryBuilder，带有 `false` - 如果你想获取内联语句参数;
带有`true` - 可以查看到带有 '?' 的查询语句.

### Q: 是否可以使用原生sql sql 作为字符串?

A: 作为库的一部分它是不支持, 但可以在它之上实现, 像这样使用:
```kotlin
fun <T:Any> String.execAndMap(transform : (ResultSet) -> T) : List<T> {
     val result = arrayListOf<T>()
     TransactionManager.current().exec(this) { rs ->
          while (rs.next()) {
               result += transform(rs)
          }
     }
     return result
}

"select u.name, c.name from user u inner join city c where blah blah".execAndMap { rs ->
    rs.getString("u.name") to rs.getString("c.name") 
}
```
更多信息查看这个issue问题: https://github.com/JetBrains/Exposed/issues/118

### Q: 是否可以相对于当前字段值更新字段?

A: 可以. 在此处查看示例: https://github.com/JetBrains/Exposed/wiki/DSL#update

### Q: How can I add another type of Database?

A: Implement `DatabaseDialect` interface and register it with `Database.registerDialect()`.  
If the implementation adds a lot of value consider contributing it as a PR to Exposed.

### Q: Is it possible to create tables with cross / cyclic reference?

A: Yes, it's possible since Exposed 0.11.1 version

### Q: How can I implement nested queries?

A: See example here: https://github.com/JetBrains/Exposed/issues/248

### Q: How can I use SAVEPOINT?
A: It possible only through using a raw connection. See example [here](https://github.com/JetBrains/Exposed/issues/320#issuecomment-394825415).

### Q: How to prepare query like: `SELECT * FROM table WHERE (x,y) IN ((1, 2), (3, 4), (5, 6))`
A: It possible with custom function. See [example](https://github.com/JetBrains/Exposed/issues/373#issuecomment-414123325).

### Q: Where can I find snapshot builds of Exposed
A: You could use jitpack.io service for that. 

Add jitpack.io to repositories:
```
repositories {
    maven { url 'https://jitpack.io' }
}
```
Then add Exposed dependency as stated below:
```
dependencies {
    implementation 'com.github.JetBrains:Exposed:-SNAPSHOT'
}
```

### Q: How can I specify a primary key column type e.g StringIdTable?
A: You need to define your own! See examples:  
[#855](https://github.com/JetBrains/Exposed/issues/855)  
https://stackoverflow.com/a/61940820/1155026

### Q: 如何创建自定义column列类型
A: 只要实现 [IColumnType](https://github.com/JetBrains/Exposed/blob/76a671e57a0105d6aed79e256c088690bd4a56b6/exposed-core/src/main/kotlin/org/jetbrains/exposed/sql/ColumnType.kt#L25)
 并使用 [registerColumn](https://github.com/JetBrains/Exposed/blob/76a671e57a0105d6aed79e256c088690bd4a56b6/exposed-core/src/main/kotlin/org/jetbrains/exposed/sql/Table.kt#L387)
 来 [扩展](https://kotlinlang.org/docs/extensions.html) 一个 [Table](https://github.com/JetBrains/Exposed/blob/76a671e57a0105d6aed79e256c088690bd4a56b6/exposed-core/src/main/kotlin/org/jetbrains/exposed/sql/Table.kt#L326)


例如: **创建一个自定义 UUID 类型 (灵感来自 [@pjagielski 文章](https://medium.com/@pjagielski/how-we-use-kotlin-with-exposed-at-touk-eacaae4565b5#e4e4))**
```kotlin
abstract class TypedId(open val id: UUID): Serializable, Comparable<TypedId> {
    override fun compareTo(other: TypedId) = this.id.compareTo(other.id)
}

class TypedUUIDColumnType<T: TypedId>(val constructor: (UUID) -> T, private val uuidColType: UUIDColumnType = UUIDColumnType()): IColumnType by uuidColType {
    override fun valueFromDB(value: Any) = constructor(uuidColType.valueFromDB(value))
    override fun notNullValueToDB(value: Any): Any = uuidColType.notNullValueToDB(valueUnwrap(value))
    override fun nonNullValueToString(value: Any): String = uuidColType.nonNullValueToString(valueUnwrap(value))
    private fun valueUnwrap(value: Any) = (value as? TypedId)?.id ?: value
}

fun <T: TypedId> Table.typedUuid(name: String, constructor: (UUID) -> T) = registerColumn<T>(name, TypedUUIDColumnType<T>(constructor))
fun <T: TypedId> Column<T>.autoGenerate(constructor: (UUID) -> T): Column<T> = clientDefault { constructor(UUID.randomUUID()) }


class StarWarsFilmId(id: UUID): TypedId(id)

object StarWarsFilms : Table() {
  val id = typedUuid("id") { StarWarsFilmId(it) }.autoGenerate{ StarWarsFilmId(it) }
  val name: Column<String> = varchar("name", 50)
  val director: Column<String> = varchar("director", 50)
  final override val primaryKey = PrimaryKey(id)
}
```


参考: [#149](https://github.com/JetBrains/Exposed/issues/149)

### 更多问题在 Stack Overflow:
https://stackoverflow.com/questions/tagged/kotlin-exposed
