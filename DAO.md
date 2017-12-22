* [Overview](#overview)
* [Basic CRUD operations](#basic-crud-operations)
  * [Create](#create)
  * [Read](#read)
  * [Update](#update)
  * [Delete](#delete)




***


## Overview

The DAO (Data Access Object) API of Exposed, is similar to ORM frameworks like Hibernate with specific Kotlin API.  
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
An instance or a row in the table is defined as a class instance:
 ```kotlin
class StarWarsFilm(id: EntityID<Int>) : IntEntity(id) {
  companion object : IntEntityClass<StarWarsFilm>(StarWarsFilms)

  var sequelId by StarWarsFilms.sequelId 
  var name     by StarWarsFilms.name
  var director by StarWarsFilms.director
}
```
## Basic CRUD operations
### Create
```kotlin
val movie = StarWarsFilm.new {
  name = "The Last Jedi"
  sequelId = 8
  director = "Rian Johnson"
}
```
### Read
```kotlin

```

### Update
```kotlin

```
### Delete
```kotlin
movie.delete() 
```

