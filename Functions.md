* [How to use functions](#how-to-use-functions)
* [String functions](#string-functions)
* [Aggregating functions](#aggregating-functions)
* [Custom functions](#custom-functions)

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

## Custom functions
If you can't find your most loved function used in your database (as Exposed provides only basic support for classic SQL functions) you can define your own functions.

Since Exposed 0.15.1 there multiple options to define custom functions:
1. Function without parameters:
```kotlin
val sqrt = FooTable.id.function("SQRT")
```
In SQL representation it will be `SQRT(FooTable.id)`

2. Function with additional parameters:
```kotlin
val replacedName = CustomFunction<String?>("REPLACE", VarCharColumnType(), FooTable.name, stringParam("foo"), stringParam("bar"))

``` 
`CustomFunction` class accept function name as a first parameter and resulting column type as second, after that you can provide any amount of parameters with will be separated by a comma.

There are also shortcuts for string, long and datetime functions:
* `CustomStringFunction`
* `CustomLongFunction`
* `CustomDateTimeFunction`

The code above could be simplified to:
```kotlin
val replacedName = CustomStringFunction("REPLACE", FooTable.name, stringParam("foo"), stringParam("bar"))

``` 
