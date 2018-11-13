* [Overview](#overview)
* [Basic CRUD operations](#basic-crud-operations)
  * [Create](#create)
  * [Read](#read)
  * [Update](#update)
    * [Sort](#sort-order-by)
  * [Delete](#delete)
* [Referencing](#referencing)
  * [many-to-one reference](#many-to-one-reference)
  * [Optional reference](#optional-reference)
  * [many-to-many reference](#many-to-many-reference)
* [Advanced CRUD operations](#advanced-crud-operations)
  * [Read entity with a join to another table](#read-entity-with-a-join-to-another-table)
  * [Auto-fill created and updated columns on entity change](#auto-fill-created-and-updated-columns-on-entity-change)

***


## Overview

The DAO (Data Access Object) API of Exposed, is similar to ORM frameworks like Hibernate with a Kotlin-specific API.  
A DB table is represented by an `object` inherited from `org.jetbrains.exposed.sql.Table` like this:
```kotlin
object StarWarsFilms : Table() {
  val id: Column<Int> = integer("id").autoIncrement().primaryKey()
  val sequelId: Column<Int> = integer("sequel_id").uniqueIndex()
  val name: Column<String> = varchar("name", 50)
  val director: Column<String> = varchar("director", 50)
}
```
Tables that contain an `Int` id with the name `id` can be declared like this:
```kotlin
object StarWarsFilms : IntIdTable() {
  val sequelId: Column<Int> = integer("sequel_id").uniqueIndex()
  val name: Column<String> = varchar("name", 50)
  val director: Column<String> = varchar("director", 50)
}
```
Note that these Column types will be defined automatically, so you can also just leave them out. This would produce the same result as the example above:
```kotlin
object StarWarsFilms : IntIdTable() {
  val sequelId = integer("sequel_id").uniqueIndex()
  val name = varchar("name", 50)
  val director = varchar("director", 50)
}
```
An entity instance or a row in the table is defined as a class instance:
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
To get entities use one of the following
```kotlin
val movies = StarWarsFilm.all()
val movies = StarWarsFilm.find { StarWarsFilms.sequelId eq 8 }
val movie = StarWarsFilm.findById(5)
```
* For a list of available predicates see [DSL Where expression](https://github.com/JetBrains/Exposed/wiki/DSL#where-expression).  

Read a value from a property similar to any property in a Kotlin class:
```kotlin
val name = movie.name
```
#### Sort (Order-by)
Ascending order:
```kotlin
val movies = StarWarsFilm.all().sortedBy { it.sequelId }
```
Descending order:
```kotlin
val movies = StarWarsFilm.all().sortedByDescending{ it.sequelId }
```
### Update
Update a value of a property similar to any property in a Kotlin class:
```kotlin
movie.name = "Episode VIII – The Last Jedi"
```
* Note: Exposed doesn't make an immediate update when you set a new value for Entity, it just stores it on the inner map. "Flushing" values to the database occurs at the end of the transaction or before next `select *` from the database.
### Delete
```kotlin
movie.delete() 
```

## Referencing
### many-to-one reference
Let's say you have this table:
```kotlin
object Users: IntIdTable() {
    val name = varchar("name", 50)
}

class User(id: EntityID<Int>): IntEntity(id) {
    companion object : IntEntityClass<User>(Users)

    var name by Users.name
}
```
And now you want to add a table referencing this table (and other tables!):
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
Now you can get the film for a rating in the same way you would get any other field:
```kotlin
filmRating.film // returns a StarWarsFilm object
```
Now if you wanted to get all the ratings for a film, you could do that by using the `FilmRating.find` function, but what is much easier is to just add a `referrersOn` field to the StarWarsFilm class:
```kotlin
class StarWarsFilm(id: EntityID<Int>) : IntEntity(id) {
    companion object : IntEntityClass<StarWarsFilm>(StarWarsFilms)

    ...
    val ratings by UserRating referrersOn UserRatings.film // make sure to use val and referrersOn
    ...
}
```
You can call:
```kotlin
movie.ratings // returns all UserRating objects with this movie as film
```
### Optional reference
You can also add an optional reference:
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
Now `secondUser` will be a nullable field.
Of course you can still use `referrersOn`.

### many-to-many reference
In some cases a many-to-many reference may be required.
Let's assume you want to add a reference to the following Actors table to the StarWarsFilm class:
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
Create an additional intermediate table to store the references:
```kotlin
object StarWarsFilmActors : Table() {
    val starWarsFilm = reference("starWarsFilm", StarWarsFilms).primaryKey(0)
    val actor = reference("actor", Actors).primaryKey(1)
}
```
Add a reference to `StarWarsFilm`:
```kotlin
class StarWarsFilm(id: EntityID<Int>) : IntEntity(id) {
    companion object : IntEntityClass<StarWarsFilm>(StarWarsFilms)

    ...
    var actors by Actor via StarWarsFilmActors
    ...
}
```
Note: Creating the entity and the reference in the same `transaction` does only work if you set the
id column manually. If you're using `UUIDTables` and `UUIDEntity` you can do it like this:
```kotlin
transaction {
 //only works with UUIDTable and UUIDEntity
 StarWarsFilm.new (UUID.randomUUID()){
    ...
    actors = SizedCollection(listOf(actor))
  }
}
```
If you don't want to set the ID column manually, you have to create the entity in it's own `transaction` and set the relation afterward in another `transaction`:
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

## Advanced CRUD operations
### Read entity with a join to another table
Lets imagine that you want to find all users who rated second SW film with more then 5.
First of all we should write that query using Exposed DSL.
```kotlin
val query = Users.innerJoin(UserRatings).innerJoin(StarWarsFilm)
  .slice(Users.columns)
  .select {
    StarWarsFilms.sequelId eq 2 and (UserRatings.value gt 5) 
  }.withDistinct()
```
After that all we have to do is to "wrap" a result with User entity:
```kotlin
val users = User.wrapRows(query).toList()
```

### Auto-fill created and updated columns on entity change
See example by @PaulMuriithi [here](https://github.com/PaulMuriithi/ExposedDatesAutoFill/blob/master/src/main/kotlin/app/Models.kt).