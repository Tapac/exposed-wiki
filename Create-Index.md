* [Single Column Index](#single-column-index)
* [Multiple Column Index](#multiple-column-index)
* [Customizing An Index](#customizing-an-index)
  - [Index With Type](#index-with-type)
  - [Functional Index](#functional-index)
  - [Partial Index](#partial-index)
* [Accessing Index Statements](#accessing-index-statements)

## Single Column Index

The simplest way to create an index is to use an extension function directly on a column:
```kotlin
object FooTable : Table("foo") {
    val id = integer("id")
    val bar = varchar("bar", 32).index()
    val baz = varchar("baz", 32).uniqueIndex()

    override val primaryKey = PrimaryKey(id)
}
```
Both functions have the parameter `customIndexName` that accepts a String argument if the index name needs to be set; otherwise, the name of the index is determined by the table and column names. In the example above, the first index would be `foo_bar` and the unique index `foo_baz`.

**Note:** `index()` also has a parameter `isUnique` that defaults to `false`, allowing it to be used as `uniqueIndex()` if needed.

## Multiple Column Index

An index can also be created using multiple columns:
```kotlin
object FooTable : Table("foo") {
    val id = integer("id")
    val bar = varchar("bar", 32)
    val baz = varchar("baz", 32)
    val qux = varchar("qux", 32)

    override val primaryKey = PrimaryKey(id)

    init {
        index(isUnique = false, bar, baz)
        uniqueIndex(baz, qux)
    }
}
```

Both functions have the same parameters as their respective extension functions, allowing a custom name to be provided; otherwise, the name is derived from the table and column names. In the example above, the first index would be `foo_bar_baz` and the unique index `foo_baz_qux`.

## Customizing An Index

The functions `index()` and `uniqueIndex()` have multiple parameters that allow the created index to be customized to most common specific needs.

### Index With Type

A String argument specifying the index type can be provided:
```kotlin
init {
    index("btree_bar_index", false, bar, indexType = "HASH")
}
```

### Functional Index

Some databases support functional key parts that index expressions instead of columns directly:
```kotlin
init {
    index(functions = listOf(bar.lowerCase(), baz.substring(1, 5)))
    // CREATE INDEX foo_lower_substring ON foo (LOWER(foo.bar), SUBSTRING(foo.baz, 1, 5))

    uniqueIndex(columns = arrayOf(bar), functions = listOf(Coalesce(baz, stringLiteral("*"))))
    // CREATE UNIQUE INDEX foo_bar_coalesce_unique ON foo (bar, COALESCE(foo.baz, '*'))
}
```
Operator expressions, like `plus()`, are also accepted by the `functions` parameter.

### Partial Index

Some databases support creating a partial index by defining a filter expression to improve querying performance. The created index will only contain entries for the table rows that match this predicate.
```kotlin
init {
    index(columns = arrayOf(flag, title)) { flag eq true }
    index(columns = arrayOf(amount, title)) { (title like "A%") and (amount greaterEq 10) }
}
```

## Accessing Index Statements

Once a table has been created, the list of its indices can be accessed using the property `Table.indices`. Table indices are represented by the data class `Index`, so its properties can be checked in the following manner, for example:
```kotlin
FooTable.indices.map { it.indexName to it.createStatement().first() }
```
**Note:** An instance of the `Index` data class can be created directly using its public constructor, for the purpose of evaluating or using  create/modify/drop statements, for example. Doing so will not add the instance to an existing table's list of indices in the way that using `index()` would. Also, if an instance is created with arguments provided to the `functions` parameter, a `functionsTable` argument must also be provided.