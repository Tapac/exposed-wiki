* [How to use functions](#how-to-use-functions)
* [String functions](#string-functions)
* [Aggregating functions](#aggregating-functions)
* [Custom functions](#custom-functions)

## How to use functions
In cases when you want to retrieve a function result from a query, then you have to declare the function as a variable like:
```kotlin
val lowerCasedName = FooTable.name.lowerCase()
val lowerCasedNames = FooTable.slice(lowerCasedName).selectAll().map { it[lowerCasedName] }

``` 
Also, functions could be chained and combined:
```kotlin
val trimmedAndLoweredFullName = Concat(FooTable.firstName, stringLiteral(" "), FooTable.lastName).trim().lowerCase()
val fullNames = FooTable.slice(trimmedAndLoweredFullName).selectAll().map { it[trimmedAndLoweredFullName] }

```

## String functions
### LowerCase/UpperCase
Returns a lower-cased/upper-cased string value.
```kotlin
val lowerCasedName = FooTable.name.lowerCase()
val lowerCasedNames = FooTable.slice(lowerCasedName).selectAll().map { it[lowerCasedName] }

```
### Substring
Returns a substring value from the specified start and with the specified length.
```kotlin
val shortenedName = FooTable.name.substring(start = 1, length = 3)
val shortenedNames = FooTable.slice(shortenedName).selectAll().map { it[shortenedName] }

```
### Concat
Returns a string value that concatenates the text representations of all non-null input values, separated by an optional separator.
```kotlin
val userName = concat(stringLiteral("User - "), FooTable.name)
val userNames = FooTable.slice(userName).selectAll().map { it[userName] }

```

## Aggregating functions
These functions should be used in queries with [[groupBy|DSL#group-by]].
### Min/Max/Average
Returns minimum/maximum/average value and can be applied to any comparable expression:
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
`CustomFunction` class accepts function name as a first parameter and resulting column type as second, after that you can provide any amount of parameters separated by a comma.

There are also shortcuts for string, long, and datetime functions:
* `CustomStringFunction`
* `CustomLongFunction`
* `CustomDateTimeFunction`

The code above could be simplified to:
```kotlin
val replacedName = CustomStringFunction("REPLACE", FooTable.name, stringParam("foo"), stringParam("bar"))

``` 
For example, the following could be used in SQLite to mimic its `date()` function:
```kotlin
val lastDayOfMonth = CustomDateFunction(
    "date",
    FooTable.dateColumn,
    stringLiteral("start of month"),
    stringLiteral("+1 month"),
    stringLiteral("-1 day")
)
```
3. Function that requires more complex query building:

All functions in Exposed extend the abstract class `Function`, which takes a column type and allows overriding `toQueryBuilder()`. This is what `CustomFunction` actually does, which can be leveraged to create more complex queries.

For example, Exposed provides a `trim()` function that removes leading and trailing whitespace from a String. In MySQL, this is just the default behavior as specifiers can be provided to limit the trim to either leading or trailing, as well as providing a specific substring other than spaces to remove. The custom function below supports this extended behavior:
```kotlin
enum class TrimSpecifier { BOTH, LEADING, TRAILING }

class CustomTrim<T : String?>(
    val expression: Expression<T>,
    val toRemove: Expression<T>?,
    val trimSpecifier: TrimSpecifier
) : Function<T>(TextColumnType()) {
    override fun toQueryBuilder(queryBuilder: QueryBuilder) {
        queryBuilder {
            append("TRIM(")
            append(trimSpecifier.name)
            toRemove?.let { +" $it" }
            append(" FROM ")
            append(expression)
            append(")")
        }
    }
}

fun <T : String?> Expression<T>.customTrim(
    toRemove: Expression<T>? = null,
    specifier: TrimSpecifier = TrimSpecifier.BOTH
): CustomTrim<T> = CustomTrim(this, toRemove, specifier)

transaction {
    FooTable.insert { it[name] = "xxxbarxxx" }

    val leadingXTrim = FooTable.name.customTrim(stringLiteral("x"), TrimSpecifier.LEADING)
    val trailingXTrim = FooTable.name.customTrim(stringLiteral("x"), TrimSpecifier.TRAILING)

    FooTable.slice(leadingXTrim).selectAll() // barxxx
    FooTable.slice(trailingXTrim).selectAll()  // xxxbar
}

```