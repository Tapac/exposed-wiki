
## How to use functions
In cases when you want to retrieve function result from a query you have to declare function as a variable like:
```kotlin
val lowerCasedName = FooTable.name.lowerCase()
val lowerCasedNames = FooTable.slice(lowerCasedName).selecAll().map { it[lowerCasedName] }

``` 
Also, functions could be chained and combined:
```kotlin
val trimmedAndLoweredFullName = Concat(FooTable.firstName, stringLiteral(" "), FooTable.lastName).trim().lowerCase()
val fullNames = FooTable.slice(trimmedAndLoweredFullName).selecAll().map { it[trimmedAndLoweredFullName] }

```

## String functions
### LowerCase/UpperCase
Returns lower-cased/upper-cased string value.
```kotlin
val lowerCasedName = FooTable.name.lowerCase()
val lowerCasedNames = FooTable.slice(lowerCasedName).selecAll().map { it[lowerCasedName] }

```


## Aggregating functions
Those functions should be used in queries with [[groupBy|DSL#group-by]].
### Min/Max/Average
Returns minimum/maximum/average value, could be applied to any comparable expression
```kotlin
val minId = FooTable.id.min()
val maxId = FooTable.id.max()
val averageId = FooTable.id.avg()
val (min, max, avg) = FooTable.slice(minId, maxId, averageId).selecAll().map { 
    Triple(it[minId], it[maxId], it[averageId]) 
}

```