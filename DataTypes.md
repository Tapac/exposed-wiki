Currently Exposed support the following data types in table definition:  
* `integer` - translates to DB `INT`
* `long` - `BIGINT`
* `float` - `FLOAT`
* `decimal` - `DECIMAL` with scale and precision
* `bool` - `BOOLEAN`
* `char` - `CHAR`
* `varchar` - `VARCHAR` with length
* `text` - `TEXT`
* `enumeration` - `INT` ordinal value
* `enumerationByName` - `VARCHAR`
* `customEnumeration` - see [additional section](#how-to-use-database-enum-types)
* `date` - `DATETIME`
* `datetime` - `DATETIME`
* `blob` - `BLOB`
* `binary` - `VARBINARY` with length
* `uuid` - `BINARY(16)`
* `reference` - a foreign key

Note: some types are different for specific DB dialect.

## How to use database Enum types
Some of the databases (e.g. MySQL, PostgreSQL, H2) supports explicit ENUM types. Because keeping such columns in sync with kotlin enumerations using only jdbc metadata could be a huge challenge, Exposed doesn't provide a possibility to manage such columns in an automatic way, but that doesn't mean that you can't use such column types.
You have two options to work with ENUM database types:
1. Use existing ENUM column from your tables
2. Create column from Exposed by providing raw definition SQL
In both cases, you should use `customEnumeration` function (available since version 0.10.3)

As a jdbc-driver can provide/expect specific classes for Enum type, you must provide from/to transformation functions for them when defining a `customEnumeration`. 

For such enum `private enum class Foo { Bar, Baz }` you can use provided code for your database:

**H2**
```Kotlin
    val existingEnumColumn = customEnumeration("enumColumn", {Foo.values()[it as Int]}, {it.name})
    val newEnumColumn = customEnumeration("enumColumn", "ENUM('Bar', 'Baz')" {Foo.values()[it as Int]}, {it.name})
```

**MySQL**
```Kotlin
    val existingEnumColumn = customEnumeration("enumColumn", {Foo.valueOf(value as String)}, {it.name})
    val newEnumColumn = customEnumeration("enumColumn", "ENUM('Bar', 'Baz')" {Foo.valueOf(value as String)}, {it.name})
```

**PostgreSQL**

PostgreSQL requires that ENUM is defined as a separate type, so you have to create it before creating your table. Also postgresql jdbc driver returns PGobject instances for such values. The full working sample is provided below.
```Kotlin
class PGEnum<T:Enum<T>>(enumTypeName: String, enumValue: T?) : PGobject() {
    init {
        value = enumValue?.name
        type = enumTypeName
    }
}

object EnumTable : Table() {
    val enumColumn = customEnumeration("enumColumn", "FooEnum", {Foo.valueOf(value as String)}, { PGEnum("FooEnum", it})
}
...
transaction {
   exec("CREATE TYPE FooEnum AS ENUM ('Bar', 'Baz');")
   SchemaUtils.create(EnumTable)
   ...
}
```