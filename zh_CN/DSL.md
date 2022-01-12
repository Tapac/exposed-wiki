* [概览](#概览)
* [基本的增删改查操作](#基本的增删改查操作)
  * [新增](#新增)
  * [查询](#查询)
  * [修改](#修改)
  * [删除](#删除)
* [Where 条件表达式](#Where条件表达式)
* [有条件的 where](#有条件的where)
* [统计Count](#统计Count)
* [排序Order-by](#排序Order-by)
* [分组Group-by](#分组Group-by)
* [限制Limit](#限制Limit)
* [Join](#join)
* [Union](#union)
* [别名Alias](#别名Alias)
* [Schema](#schema)
* [序列Sequence](#序列Sequence)
* [批量插入Batch Insert](#批量插入Batch Insert)
* [查询新增Insert From Select](#查询插入Insert From Select)
***
## 概览
Exposed 的 DSL（Domain Specific Language）API , 类似 Kotlin 提供的具有类型安全性的实际 SQL 语句.
一个数据库table表, 由继承自 `org.jetbrains.exposed.sql.Table` 的一个 `object`表示，如下所示:
```kotlin
object StarWarsFilms : Table() {
  val id: Column<Int> = integer("id").autoIncrement()
  val sequelId: Column<Int> = integer("sequel_id").uniqueIndex()
  val name: Column<String> = varchar("name", 50)
  val director: Column<String> = varchar("director", 50)
  override val primaryKey = PrimaryKey(id, name = "PK_StarWarsFilms_Id") // PK_StarWarsFilms_Id is optional here
}
```
包含 `Int` 类型 id 字段的表, 可以这样声明:
```kotlin
object StarWarsFilms : IntIdTable() {
  val sequelId: Column<Int> = integer("sequel_id").uniqueIndex()
  val name: Column<String> = varchar("name", 50)
  val director: Column<String> = varchar("director", 50)
}
``` 
## 基本的增删改查操作
### 新增
```kotlin
val id = StarWarsFilms.insertAndGetId {
  it[name] = "The Last Jedi"
  it[sequelId] = 8
  it[director] = "Rian Johnson"
}
```
### 查询
```kotlin
val query: Query = StarWarsFilms.select { StarWarsFilms.sequelId eq 8 }
```
`Query` 继承 `Iterable`, 因此可以使用 map/foreach 等遍历它. 例如:
```kotlin
StarWarsFilms.select { StarWarsFilms.sequelId eq 8 }.forEach {
  println(it[StarWarsFilms.name])
}
```
`slice` 函数可让您选择特定的列或/和表达式.
```kotlin
val filmAndDirector = StarWarsFilms.
   slice(StarWarsFilms.name, StarWarsFilms.director).
   selectAll().map {
      it[StarWarsFilms.name] to it[StarWarsFilms.director]
   }
```
如果您只想选择不同的值，请使用 `withDistinct()` 函数:
```kotlin
val directors = StarWarsFilms.
   slice(StarWarsFilms.director).
   select { StarWarsFilms.sequelId less 5 }.
   withDistinct().map {
      it[StarWarsFilms.director]
   }
```
### 修改
```kotlin
StarWarsFilms.update ({ StarWarsFilms.sequelId eq 8 }) {
  it[StarWarsFilms.name] = "Episode VIII – The Last Jedi"
}
```
如果您想使用表达式来更新列值(例如增量increment)，请使用 `update` 函数或setter:
```kotlin
StarWarsFilms.update({ StarWarsFilms.sequelId eq 8 }) {
    with(SqlExpressionBuilder) {
       it.update(StarWarsFilms.sequelId, StarWarsFilms.sequelId + 1)
       // or 
       it[StarWarsFilms.sequelId] = StarWarsFilms.sequelId + 1
    }
} 
```
### 删除
```kotlin
StarWarsFilms.deleteWhere { StarWarsFilms.sequelId eq 8 }
```
## Where条件表达式
查询表达式（where）需要一个布尔运算符（即：`Op<Boolean>`）.  
以下是允许的条件(conditions):  
```
eq - (==)
neq - (!=)
isNull()
isNotNull()
less - (<)
lessEq - (<=)
greater - (>)
greaterEq - (>=)
like - (=~)
notLike - (!~)
exists
notExists
regexp
notRegexp
inList
notInList
between
match (MySQL MATCH AGAINST) 
```
以下是允许的逻辑条件(logical conditions):
```
not
and
or
```
## 有条件的where
It rather common case when your query's `where` condition depends on some other code conditions. Moreover, it could be independent or nested conditions what make it more complicated to prepare such `where`. 
Let's imagine that we have a form on a website where a user can optionally filter "Star Wars" films by a director and/or a sequel.
In Exposed version before 0.8.1 you had to code it like:
一种常见的情况是, 您的查询的 `where` 条件取决于其他一些代码条件时. 此外, 可能是独立的或嵌套的条件, 这使得准备这样的 `where` 变得更加复杂。假设我们在网站上有一个表单，用户可以在该表单中选择过滤导演或续集的“星球大战”电影。在 0.8.1 之前的 Exposed 版本中，您必须将其编码为:
```Kotlin 
val condition = when {
    directorName != null && sequelId != null ->
        Op.build { StarWarsFilms.director eq directorName and (StarWarsFilms.sequelId eq sequelId) }
    directorName != null ->
        Op.build { StarWarsFilms.director eq directorName }
    sequelId != null ->
        Op.build { StarWarsFilms.sequelId eq sequelId }
    else -> null
}
val query = condition?.let { StarWarsFilms.select(condition) } ?: StarWarsFilms.selectAll()
```
或者 
```Kotlin
val query = when {
    directorName != null && sequelId != null ->
        StarWarsFilms.select { StarWarsFilms.director eq directorName and (StarWarsFilms.sequelId eq sequelId) }
    directorName != null ->
        StarWarsFilms.select { StarWarsFilms.director eq directorName }
    sequelId != null ->
        StarWarsFilms.select { StarWarsFilms.sequelId eq sequelId }
    else -> StarWarsFilms.selectAll()
}
```
这是一个非常原始的示例，但是您应该了解该问题的主要思想.
现在, 让我们尝试以更简单的方式编写相同的查询（`andWhere` 函数从 0.10.5 开始可用）:
```Kotlin
val query = StarWarsFilms.selectAll()
directorName?.let {
    query.andWhere { StarWarsFilms.director eq it }
}
sequelId?.let {
    query.andWhere { StarWarsFilms.sequelId eq it }
}
```
但是，如果我们有条件地从另一个表中选择并且只想在条件为true时加入它怎么办?
您必须使用 `adjustColumnSet` 和 `adjustSlice` 函数（自 0.8.1 起可用），这些函数允许扩展和修改查询的 `join` 和 `slice` 部分（请参阅有关该函数的 kdoc）:
```Kotlin
playerName?.let {
    query.adjustColumnSet { innerJoin(Players, {StarWarsFilms.sequelId}, {Players.sequelId}) }
        .adjustSlice { slice(fields + Players.columns) }
        .andWhere { Players.name eq playerName }
}
```
## 统计Count
`count()` 是 `Query` 的一个方法，如下例所示:
```kotlin
val count = StarWarsFilms.select { StarWarsFilms.sequelId eq  8 }.count()
```
## 排序Order-by
Order-by 接受一个list(元素为columns到布尔指示的映射)，指示排序应该是升序还是降序.
例子:
```kotlin
StarWarsFilms.selectAll().orderBy(StarWarsFilms.sequelId to SortOrder.ASC)
```
## 分组Group-by
在 group-by 中，通过 `slice()` 方法定义字段及其函数（例如 `count`）.
```kotlin
StarWarsFilms
  .slice(StarWarsFilms.sequelId.count(), StarWarsFilms.director)
  .selectAll()
  .groupBy(StarWarsFilms.director)
```
可用的函数如下:
```
count
sum
max
min
average
...
``` 
## 限制Limit
You can use limit function to prevent loading large data sets or use it for pagination with second `offset` parameter.
您可以使用 limit 函数来防止加载大型数据集或使用它带有第二个`offset`偏移参数的进行分页操作.
```kotlin
// 获取第 1 部电影之后的 2 部电影
StarWarsFilms.select { StarWarsFilms.sequelId eq Players.sequelId }.limit(2, offset = 1)
```
## Join
join 示例考虑如下 tables 表:
```kotlin
object StarWarsFilms : IntIdTable() {
  val sequelId: Column<Int> = integer("sequel_id").uniqueIndex()
  val name: Column<String> = varchar("name", 50)
  val director: Column<String> = varchar("director", 50)
}
object Players : Table() {
  val sequelId: Column<Int> = integer("sequel_id").uniqueIndex()
  val name: Column<String> = varchar("name", 50)
}
```
连接(join)查询统计每一部电影有多少player玩家参演:
```kotlin
(Players innerJoin StarWarsFilms)
  .slice(Players.name.count(), StarWarsFilms.name)
  .select { StarWarsFilms.sequelId eq Players.sequelId }
  .groupBy(StarWarsFilms.name)
``` 
* 如果有外键，可以将 `select{}` 替换为 `selectAll()`
  使用完整语法的相同示例:
```kotlin
Players.join(StarWarsFilms, JoinType.INNER, additionalConstraint = {StarWarsFilms.sequelId eq Players.sequelId})
  .slice(Players.name.count(), StarWarsFilms.name)
  .selectAll()
  .groupBy(StarWarsFilms.name)
```
## Union
您可以使用 `.union(...)` 组合多个查询的结果集.
根据 SQL 规范, 查询必须具有相同数量的列, 并且不标记为更新.
当数据库支持时, 可以合并子查询.
```kotlin
val lucasDirectedQuery = StarWarsFilms.slice(StarWarsFilms.name).select { StarWarsFilms.director eq "George Lucas" }
val abramsDirectedQuery = StarWarsFilms.slice(StarWarsFilms.name).select { StarWarsFilms.director eq "J.J. Abrams" }
val filmNames = lucasDirectedQuery.union(abramsDirectedQuery).map { it[StarWarsFilms.name] }
```
默认情况下仅返回唯一的 rows 行数据. 可以使用 `.unionAll()` 返回重复的 rows 行数据.
```kotlin
val lucasDirectedQuery = StarWarsFilms.slice(StarWarsFilms.name).select { StarWarsFilms.director eq "George Lucas" }
val originalTrilogyQuery = StarWarsFilms.slice(StarWarsFilms.name).select { StarWarsFilms.sequelId inList (3..5) }
val filmNames = lucasDirectedQuery.unionAll(originalTrilogyQuery).map { it[StarWarsFilms.name] }
```
## 别名Alias
alias 别名可以防止混淆字段名和表名.
使用别名变量而不是原始变量:
```Kotlin
val filmTable1 = StarWarsFilms.alias("ft1")
filmTable1.selectAll() // can be used in joins etc'
```
此外，alias 别名允许您在连接中多次使用同一个表:
```Kotlin
val sequelTable = StarWarsFilms.alias("sql")
val originalAndSequelNames = StarWarsFilms
    .innerJoin(sequelTable, { StarWarsFilms.sequelId }, { sequelTable[StarWarsFilms.id] })
    .slice(StarWarsFilms.name, sequelTable[StarWarsFilms.name])
    .selectAll()
    .map { it[StarWarsFilms.name] to it[sequelTable[StarWarsFilms.name]] }
```
当使用selecting from 子查询时, 可以使用它们:
```kotlin
val starWarsFilms = StarWarsFilms
    .slice(StarWarsFilms.id, StarWarsFilms.name)
    .selectAll()
    .alias("swf")
val id = starWarsFilms[StarWarsFilms.id]
val name = starWarsFilms[StarWarsFilms.name]
starWarsFilms
    .slice(id, name)
    .selectAll()
    .map { it[id] to it[name] }
```
## Schema
你可以创建一个schema 或 删除一个已经存在的:
```Kotlin
val schema = Schema("my_schema") // my_schema is the schema name.
// Creates a Schema
SchemaUtils.createSchema(schema)
// Drops a Schema
SchemaUtils.dropSchema(schema)
```
此外，您可以像这样指定 schema 所有者（某些数据库需要显式所有者）:
```Kotlin
val schema = Schema("my_schema", authorization = "owner")
```
如果你有很多 schema 并且你想设置一个默认 schema ，你可以使用:
```Kotlin
SchemaUtils.setSchema(schema)
```
## 序列Sequence
如果你想使用 Sequence，Exposed 是允许的:
### 定义一个 Sequence
```Kotlin
val myseq = Sequence("my_sequence") // my_sequence is the sequence name.
```
可以指定几个参数来控制 sequence 的属性:
```Kotlin
private val myseq = Sequence(
        name = "my_sequence",
        startWith = 4,
        incrementBy = 2,
        minValue = 1,
        maxValue = 10,
        cycle = true,
        cache = 20
    )
```
### 创建或删除一个 Sequence
```Kotlin
// Creates a sequence
SchemaUtils.createSequence(myseq)
// Drops a sequence
SchemaUtils.dropSequence(myseq)
```
### 使用 NextVal 函数
您可以这样使用 nextVal 函数:
```Kotlin
val nextVal = myseq.nextVal()
val id = StarWarsFilms.insertAndGetId {
  it[id] = nextVal
  it[name] = "The Last Jedi"
  it[sequelId] = 8
  it[director] = "Rian Johnson"
}
```
```Kotlin
val firstValue = StarWarsFilms.slice(nextVal).selectAll().single()[nextVal]
```
## 批量插入Batch Insert
批量插入允许在一个 sql 语句中将实体列表映射到数据库原始数据。它比一个一个插入更有效，因为它只初始化一个statement语句。这是一个例子:
```kotlin
val cityNames = listOf("Paris", "Moscow", "Helsinki")
val allCitiesID = cities.batchInsert(cityNames) { name ->
  this[cities.name] = name
}
```
*注意:* `batchInsert` 函数在与数据库交互时仍会创建多个 `INSERT` 语句。您很可能希望将此与相关 JDBC 驱动程序的 `rewriteBatchedInserts=true`（或 `rewriteBatchedStatements=true`）选项结合使用，这会将它们转换为单个 bulkInsert.
您可以在 MySQL 中找到此选项的文档 [这里](https://dev.mysql.com/doc/connector-j/5.1/en/connector-j-reference-configuration-properties.html) 和 PostgreSQL [这里](https://jdbc.postgresql.org/documentation/94/connect.html).

如果您不需要获取新生成的值（例如：自动递增的 ID），请将 `shouldReturnGeneratedValues` 参数设置为 false，这样可以通过将它们分批来提高批量插入的性能，而不是一直等待数据库同步新插入的对象状态.

如果要检查 `rewriteBatchedInserts` + `batchInsert` 是否正常工作，请检查如何为您的驱动程序启用 JDBC 日志记录，因为 Exposed 将始终显示未重写的多个插入。您可以找到有关如何在 PostgreSQL 中启用日志记录的文档 [这里](https://jdbc.postgresql.org/documentation/head/logging.html).
## 查询插入Insert From Select
如果您想使用 `INSERT INTO ... SELECT ` SQL 语句, 可以尝试 Exposed 的 `Table.insert(Query)`.
```kotlin
val substring = users.name.substring(1, 2)
cities.insert(users.slice(substring).selectAll().orderBy(users.id).limit(2))
```
默认情况下，它将尝试按照它们在 Table 实例中定义的顺序插入 non auto-increment “表”中的所有列。如果要指定某些列或更改顺序，请提供第二个参数columns(传入一个columns的list列表):
> By default it will try to insert into all non auto-increment `Table` columns in order they defined in Table instance. If you want to specify columns or change the order, provide list of columns as second parameter:
```kotlin
val userCount = users.selectAll().count()
users.insert(users.slice(stringParam("Foo"), Random().castTo<String>(VarCharColumnType()).substring(1, 10)).selectAll(), columns = listOf(users.name, users.id))
```