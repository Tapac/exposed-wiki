The DSL (Domain Specific Language) API of Exposed, is similar to actual SQL statements with type safety that Kotlin offers.  
A DB table is represented by an `object` inherited from `org.jetbrains.exposed.sql.Table` like that:
```kotlin
object StarWarsFilms : Table() {
  val id: Column<Int> = integer("id").autoIncrement().primaryKey()
  val sequelId: Column<Int> = integer("sequel_id").uniqueIndex()
  val name: Column<String> = varchar("name", 50)
}
```
Tables that contains `Int` id with the name `id` can be declared like that:
```kotlin
object StarWarsFilms : IntIdTable() {
  val sequelId: Column<Int> = integer("sequel_id").uniqueIndex()
  val name: Column<String> = varchar("name", 50)
}
``` 
## Basic CRUD operations
### Create
```kotlin
val id = StarWarsFilms.insert {
  it[name] = "The Last Jedi"
  it[sequelId] = 8
} get StarWarsFilms.id
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
match
```

## Advanced operations
### Join
