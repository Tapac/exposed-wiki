* [概览](#概览)
* [基本的增删改查操作](#基本的增删改查操作)
  * [新增](#新增)
  * [查询](#查询)
    * [排序](#排序sort-order-by)
  * [修改](#修改)
  * [删除](#删除)
* [关联Referencing](#关联Referencing)
  * [多对一many-to-one reference](#多对一many-to-one-reference)
  * [Optional reference](#optional-reference)
  * [多对多many-to-many reference](#多对多many-to-many-reference)
  * [Parent-Child reference](#parent-child-reference)
  * [Eager Loading](#eager-loading)
* [高级的增删改查操作](#高级的增删改查操作)
  * [Read entity with a join to another table](#read-entity-with-a-join-to-another-table)
  * [Auto-fill created and updated columns on entity change](#auto-fill-created-and-updated-columns-on-entity-change)
* [实体映射mapping](#实体映射mapping)
  * [Fields transformation](#fields-transformation)
***
## 概览
Exposed 的 DAO（数据访问对象）API，类似于带有 Kotlin 特定 API 的 Hibernate 等 ORM 框架.  
一个数据库table表, 由继承自 `org.jetbrains.exposed.sql.Table` 的一个 `object`表示, 如下所示:
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
请注意，以下这些 Column 类型将自动定义，因此您也可以去掉对它们的定义。产生与上面示例相同的结果:
```kotlin
object StarWarsFilms : IntIdTable() {
  val sequelId = integer("sequel_id").uniqueIndex()
  val name = varchar("name", 50)
  val director = varchar("director", 50)
}
```
一个实体实例或表中的一行row被定义为一个类实例:
 ```kotlin
class StarWarsFilm(id: EntityID<Int>) : IntEntity(id) {
  companion object : IntEntityClass<StarWarsFilm>(StarWarsFilms)
  var sequelId by StarWarsFilms.sequelId 
  var name     by StarWarsFilms.name
  var director by StarWarsFilms.director
}
```
## 基本的增删改查操作
### 新增
```kotlin
val movie = StarWarsFilm.new {
  name = "The Last Jedi"
  sequelId = 8
  director = "Rian Johnson"
}
```
### 查询
要获取实体entities，请使用以下方法之一
```kotlin
val movies = StarWarsFilm.all()
val movies = StarWarsFilm.find { StarWarsFilms.sequelId eq 8 }
val movie = StarWarsFilm.findById(5)
```
* 有关可用谓词的列表，请参见 [DSL Where expression](https://github.com/JetBrains/Exposed/wiki/DSL#where-expression).  
  从类似于 Kotlin 类中的任何属性的属性中读取值:
```kotlin
val name = movie.name
```
#### 排序sort-order-by
升序:
```kotlin
val movies = StarWarsFilm.all().sortedBy { it.sequelId }
```
降序:
```kotlin
val movies = StarWarsFilm.all().sortedByDescending{ it.sequelId }
```
### 修改
更新类似于 Kotlin 类中任何属性的属性值:
```kotlin
movie.name = "Episode VIII – The Last Jedi"
```
* 注意：当您为 Entity 设置新值时，Exposed 不会立即更新，它只是将其存储在内部映射中。向数据库“刷新Flushing”值发生在事务结束时或从数据库中的下一个“select *”之前.
### 删除
```kotlin
movie.delete() 
```
## 关联Referencing
### 多对一many-to-one reference
假如您有一张表:
```kotlin
object Users: IntIdTable() {
    val name = varchar("name", 50)
}
class User(id: EntityID<Int>): IntEntity(id) {
    companion object : IntEntityClass<User>(Users)
    var name by Users.name
}
```
现在您要添加一个reference 来引用当前表(和其他表！):
```kotlin
object UserRatings: IntIdTable() {
    val value = long("value")
    val film = reference("film", StarWarsFilms)
    val user = reference("user", Users)
}
class UserRating(id: EntityID<Int>): IntEntity(id) {
    companion object : IntEntityClass<UserRating>(UserRatings)
    var value by UserRatings.value
    var film by StarWarsFilm referencedOn UserRatings.film // use referencedOn for normal references
    var user by User referencedOn UserRatings.user
}
```
现在您可以像获取其他字段属性一样获取电影的评分rating:
```kotlin
filmRating.film // returns a StarWarsFilm object
```
现在如果您想获取一部电影的所有评分rating, 您可以使用 `FilmRating.find` 函数来实现, 但是更简单的是可以在 StarWarsFilm类中添加一个 `referrersOn` 字段属性:
```kotlin
class StarWarsFilm(id: EntityID<Int>) : IntEntity(id) {
    companion object : IntEntityClass<StarWarsFilm>(StarWarsFilms)
    ...
    val ratings by UserRating referrersOn UserRatings.film // make sure to use val and referrersOn
    ...
}
```
你可以这样调用:
```kotlin
movie.ratings // returns all UserRating objects with this movie as film
```
### Optional reference
您也可以添加一个 optional reference:
```kotlin
object UserRatings: IntIdTable() {
    ...
    val secondUser = reference("second_user", Users).nullable() // this reference is nullable!
    ...
}
class UserRating(id: EntityID<Int>): IntEntity(id) {
    companion object : IntEntityClass<UserRating>(UserRatings)
    ...
    var secondUser by User optionalReferencedOn UserRatings.secondUser // use optionalReferencedOn for nullable references
    ...
}
```
现在 `secondUser` 将是一个可空字段, 你应该使用 `optionalReferrersOn` 而不是 `referrersOn` 来获取 `secondUser` 的所有评分rating.

```kotlin
class User(id: EntityID<Int>): IntEntity(id) {
    companion object : IntEntityClass<User>(Users)
    ...
    val secondRatings by UserRating optionalReferrersOn UserRatings.secondUser // make sure to use val and optionalReferrersOn
    ...
}
```

### 多对多many-to-many reference
在某些情况下，可能需要多对多引用 many-to-many reference.
假设您想向 StarWarsFilm 类添加对以下 Actors 表的引用:
```kotlin
object Actors: IntIdTable() {
    val firstname = varchar("firstname", 50)
    val lastname = varchar("lastname", 50)
}
class Actor(id: EntityID<Int>): IntEntity(id) {
    companion object : IntEntityClass<Actor>(Actors)
    var firstname by Actors.firstname
    var lastname by Actors.lastname
}
```
创建一个额外的中间表来存储引用:
```kotlin
object StarWarsFilmActors : Table() {
    val starWarsFilm = reference("starWarsFilm", StarWarsFilms)
    val actor = reference("actor", Actors)
    override val primaryKey = PrimaryKey(starWarsFilm, actor, name = "PK_StarWarsFilmActors_swf_act") // PK_StarWarsFilmActors_swf_act is optional here
}
```
添加对 `StarWarsFilm` 的引用:
```kotlin
class StarWarsFilm(id: EntityID<Int>) : IntEntity(id) {
    companion object : IntEntityClass<StarWarsFilm>(StarWarsFilms)
    ...
    var actors by Actor via StarWarsFilmActors
    ...
}
```
注意: 仅当您手动设置 id 列时，才能在同一个`事务transaction`中创建实体和引用。如果你正在使用 `UUIDTables` 和 `UUIDEntity` 你可以这样做:
```kotlin
transaction {
 //only works with UUIDTable and UUIDEntity
 StarWarsFilm.new (UUID.randomUUID()){
    ...
    actors = SizedCollection(listOf(actor))
  }
}
```
如果您不想手动设置 id 列，则必须在其自己的`事务transaction`中创建实体，然后在另一个`事务transaction`中设置关系:
```kotlin
// create film
val film = transaction {
   StarWarsFilm.new {
    name = "The Last Jedi"
    sequelId = 8
    director = "Rian Johnson"
  }
}
//create actor
val actor = transaction {
  Actor.new {
    firstname = "Daisy"
    lastname = "Ridley"
  }
}
//add reference
transaction {
  film.actors = SizedCollection(listOf(actor))
}
```
### Parent-Child reference
Parent-child reference父子引用与多对多many-to-many版本非常相似，但中间表包含对同一个表的两个引用.
假设您要构建一个具有 parents 和 children 的分层实体. 我们的表和实体映射看起来会是这样
```kotlin
object NodeTable : IntIdTable() {
    val name = varchar("name", 50)
}
object NodeToNodes : Table() {
    val parent = reference("parent_node_id", NodeTable)
    val child = reference("child_user_id", NodeTable)
}
class Node(id: EntityID<Int>) : IntEntity(id) {
    companion object : IntEntityClass<Node>(NodeTable)
    var name by NodeTable.name
    var parents by Node.via(NodeToNodes.child, NodeToNodes.parent)
    var children by Node.via(NodeToNodes.parent, NodeToNodes.child)
}
```
如您所见，`NodeToNodes` 列仅针对 `NodeTable`，并且使用了另一个版本的 `via` 函数.
现在您可以创建一个有层次结构的nodes.
```kotlin
val root = Node.new { name = "root" }
val child1 = Node.new {
    name = "child1" 
}
child1.parents = SizedCollection(root) // assign parent
val child2 = Node.new { name = "child2" }
root.children = SizedCollection(listOf(child1, child2)) // assign children
```
请注意，您不能在 `new` block块内设置引用，因为尚未创建实体并且未定义要引用的 id.
### Eager Loading
**自 0.13.1 起可用**.
Exposed 中的引用是延迟加载的，这意味着获取引用数据的查询是在首次使用引用时进行的。对于您知道提前需要引用的场景，Exposed 可以在父查询时预先加载它们，这可以防止经典的“N+1”问题，因为引用可以在单个查询中聚合和加载.
To eager load a reference you can call the "load" function and pass the DAO's reference as a KProperty:
```kotlin
StarWarsFilm.findById(1).load(StarWarsFilm::actors)
```
This works for references of references also, 例如, 如果 Actors 有一个 评分rating 的 reference 引用, 你可以:
```kotlin
StarWarsFilm.findById(1).load(StarWarsFilm::actors, Actor::rating)
```
同样, 您可以在DAO集合(Lists、SizedIterables)上使用 eager load references, 对于集合，您可以像以前一样使用 with 函数, 将 DAO's reference 作为 KProperty 的引用传递.
```kotlin
StarWarsFilm.all().with(StarWarsFilm::actors)
```
注意：通过eager loaded 的引用存储在事务缓存中, 这意味着它们在其他事务中不可用，因此必须在同一事务中加载和引用.

#### Eager loading for Text Fields
某些数据库驱动程序不会立即加载文本内容（出于性能和内存原因），这意味着您只能在打开的事务中获取列值. 

如果您希望在事务之外使得内容可用，您可以在定义数据库表时使用 eagerLoading 参数.
> If you desire to make content available outside the transaction, you can use the eagerLoading param when defining the DB Table.
```kotlin
object StarWarsFilms : Table() {
  ...
  val description = text("name", eagerLoading=true)
}
```
## 高级的增删改查操作
### Read entity with a join to another table
假设您要查找对第二部 SW 电影评分超过 5 的所有用户.
首先，我们应该使用 Exposed DSL 编写查询语句.
```kotlin
val query = Users.innerJoin(UserRatings).innerJoin(StarWarsFilm)
  .slice(Users.columns)
  .select {
    StarWarsFilms.sequelId eq 2 and (UserRatings.value gt 5) 
  }.withDistinct()
```
之后我们要做的就是用User entity “包装”一个结果:
```kotlin
val users = User.wrapRows(query).toList()
```
### Auto-fill created and updated columns on entity change
参见示例 @PaulMuriithi [这里](https://github.com/PaulMuriithi/ExposedDatesAutoFill/blob/master/src/main/kotlin/app/Models.kt).
### Use queries as expressions
想象一下，您想根据每个city拥有的user数量对city进行排序。为此，您可以编写一个子查询来计算每个city的用户user并按该数字排序。虽然为了这样做，您必须将 `Query` 转换为 `Expression`。这可以使用 `wrapAsExpression` 函数来完成:
```kotlin
val expression = wrapAsExpression<Int>(Users
  .slice(Users.id.count())
  .select {
      Cities.id eq Users.cityId
  })
val cities = Cities
  .selectAll()
  .orderBy(expression, SortOrder.DESC)
  .toList()
```
## 实体映射mapping
### Fields transformation
由于数据库只能存储整数和字符串等基本类型，因此在 DAO 级别保持相同的简单性并不总是很方便.
有时您可能想要进行一些转换，例如从 varchar 列解析 json 或根据数据库中的值从缓存中获取一些值.
在这种情况下，首选方法是使用列转换。假设我们要在 Entity 上定义无符号整数字段，但是 Exposed 还没有这样的列类型.
```kotlin
object TableWithUnsignedInteger : IntIdTable() {
    val uint = integer("uint")
}
class EntityWithUInt : IntEntity() {
    var uint: UInt by TableWithUnsignedInteger.uint.transform({ it.toInt() }, { it.toUInt() })
    
    companion object : IntEntityClass<EntityWithUInt>()
}
```
`transform` 函数接受两个 lambdas 来将值与原始列类型进行转换.
之后，在您的代码中，您将只能将 `UInt` 实例放入 `uint` 字
仍然可以通过 DAO 插入带有负整数的更新值，但是您的业务代码会变得更加简洁.
请记住，每次对字段的访问都会产生什么这样的转换，这意味着您应该在此处避免大量转换.