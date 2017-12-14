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
val id = StarWarsFilms. insertAndGetId {
  it[name] = "The Last Jedi"
  it[sequelId] = 8
  it[director] = "Rian Johnson"
}
```
### Read
```kotlin
val query: Query = StarWarsFilms.select { StarWarsFilms.sequelId eq 8 }
```
`Query` inherit `Iterable` so it is possible to traverse it with map/foreach etc'
### Update
```kotlin
StarWarsFilms.update ({ StarWarsFilms.sequelId eq 8 }) {
  it[name] = "Episode VIII – The Last Jedi"
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
`count()` is a method of `Query` and executed like that:
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
In group by, define fields and their functions (such as `count`) by the `slice()` method.
```kotlin
StarWarsFilms
  .slice(StarWarsFilms.sequelId.count(), StarWarsFilms.director)
  .selectAll()
  .groupBy(StarWarsFilms.director)
```
Availbe functions are:
```
count
sum
max
min
average
...
``` 
## Join
For the join example consider the following tables:
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
Join to count how many players plays in each movie:
```kotlin
(Players innerJoin StarWarsFilms)
  .slice(Players.name.count(), StarWarsFilms.name)
  .select { StarWarsFilms.sequelId eq Players.sequelId }
  .groupBy(StarWarsFilms.name)
``` 