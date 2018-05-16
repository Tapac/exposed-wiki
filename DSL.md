* [Overview](#overview)
* [Basic CRUD operations](#basic-crud-operations)
  * [Create](#create)
  * [Read](#read)
  * [Update](#update)
  * [Delete](#delete)
* [Where expression](#where-expression)
* [Count](#count)
* [Order-by](#order-by)
* [Group-by](#group-by)
* [Limit](#limit)
* [Join](#join)
* [Alias](#alias)
* [Batch Insert](#batch-insert)
* [Insert From Select](#insert-from-select)



***


## Overview

The DSL (Domain Specific Language) API of Exposed, is similar to actual SQL statements with type safety that Kotlin offers.  
A DB table is represented by an `object` inherited from `org.jetbrains.exposed.sql.Table` like that:
```kotlin
object StarWarsFilms : Table() {
  val id: Column<Int> = integer("id").autoIncrement().primaryKey()
  val sequelId: Column<Int> = integer("sequel_id").uniqueIndex()
  val name: Column<String> = varchar("name", 50)
  val director: Column<String> = varchar("director", 50)
}
```
Tables that contains `Int` id with the name `id` can be declared like that:
```kotlin
object StarWarsFilms : IntIdTable() {
  val sequelId: Column<Int> = integer("sequel_id").uniqueIndex()
  val name: Column<String> = varchar("name", 50)
  val director: Column<String> = varchar("director", 50)
}
``` 
## Basic CRUD operations
### Create
```kotlin
val id = StarWarsFilms.insertAndGetId {
  it[name] = "The Last Jedi"
  it[sequelId] = 8
  it[director] = "Rian Johnson"
}
```
### Read
```kotlin
val query: Query = StarWarsFilms.select { StarWarsFilms.sequelId eq 8 }
```
`Query` inherit `Iterable` so it is possible to traverse it with map/foreach etc'. For example:
```kotlin
StarWarsFilms.select { StarWarsFilms.sequelId eq 8 }.forEach {
  println(it[StarWarsFilms.name])
}
```
There is `slice` function which allows you to select specific columns or/and expressions.
```kotlin
val filmAndDirector = StarWarsFilms.
   slice(StarWarsFilms.name, StarWarsFilms.director).
   selectAll().map {
      it[StarWarsFilms.name] to it[StarWarsFilms.director]
   }
```

If you want to select only distinct value then use `withDistinct()` function:
```kotlin
val directors = StarWarsFilms.
   slice(StarWarsFilms.director).
   select { StarWarsFilms.sequelId < 5 }.
   withDistinct().map {
      it[StarWarsFilms.director]
   }
```

### Update
```kotlin
StarWarsFilms.update ({ StarWarsFilms.sequelId eq 8 }) {
  it[name] = "Episode VIII – The Last Jedi"
}
```

If you want to update column value with some expression like increment:
```kotlin
StarWarsFilms.update({ StarWarsFilms.sequelId eq 8 }) {
    with(SqlExpressionBuilder) {
       it.update(StarWarsFilms.sequelId, StarWarsFilms.sequelId + 1)
    }
} 
```

### Delete
```kotlin
StarWarsFilms.deleteWhere { StarWarsFilms.sequelId eq 8 }
```

## Where expression
Query expression (where) expects a boolean operator (ie: `Op<Boolean>`).  
Allowed conditions are:  
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
regexp
notRegexp
inList
notInList
between
match (MySQL MATCH AGAINST) 
```
Allowed logical conditions are:
```
and
or
```
## Count
`count()` is a method of `Query` that is used like below example:
```kotlin
val count = StarWarsFilms.select { StarWarsFilms.sequelId eq  8 }.count()
```
## Order-by
Order-by accepts a list of columns mapped to boolean indicates if sorting should be ascending or descending.
Example:
```kotlin
StarWarsFilms.selectAll().orderBy(StarWarsFilms.sequelId to true)
```
## Group-by
In group-by, define fields and their functions (such as `count`) by the `slice()` method.
```kotlin
StarWarsFilms
  .slice(StarWarsFilms.sequelId.count(), StarWarsFilms.director)
  .selectAll()
  .groupBy(StarWarsFilms.director)
```
Available functions are:
```
count
sum
max
min
average
...
``` 
## Limit
You can use limit function to prevent loading large data sets or use it for pagination with second `offset` parameter.
```kotlin
// Take 2 films after the first one.
StarWarsFilms.select { StarWarsFilms.sequelId eq Players.sequelId }.limit(2, offset = 1)
```
## Join
For join example consider the following tables:
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
Join to count how many players play in each movie:
```kotlin
(Players innerJoin StarWarsFilms)
  .slice(Players.name.count(), StarWarsFilms.name)
  .select { StarWarsFilms.sequelId eq Players.sequelId }
  .groupBy(StarWarsFilms.name)
``` 
* In case there is a foreign key it is possible to replace `select{}` with `selectAll()`

## Alias
Aliases allow preventing ambiguity between field names and table names.
Use the aliased var instead of original one:
```
val filmTable1 = StarWarsFilms.alias("ft1")
filmTable1.selectAll() // can be used in joins etc'
```

## Batch Insert
Batch Insert allow mapping a list of entities into DB raws in one sql statement. It is more efficient than inserting one by one as it initiates only one statement. Here is an example:
```kotlin
val cityNames = listOf("Paris", "Moscow", "Helsinki")
val allCitiesID = cities.batchInsert(cityNames) { name ->
  this[cities.name] = name
}
```

## Insert From Select
If you want to use `INSERT INTO ... SELECT ` SQL clause try Exposed analog `Table.insert(Query)`.
```kotlin
val substring = users.name.substring(1, 2)
cities.insert(users.slice(substring).selectAll().orderBy(users.id).limit(2))
```
By default it will try to insert into all non auto-autoincrement `Table` columns in order they defined in Table instance. If you want to specify columns or change the order, provide list of columns as second paramter:
```kotlin
val userCount = users.selectAll().count()
users.insert(users.slice(stringParam("Foo"), Random().castTo<String>(VarCharColumnType()).substring(1, 10)).selectAll(), columns = listOf(users.name, users.id))
```