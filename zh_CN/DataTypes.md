目前 Exposed 在表table的定义中支持以下数据类型:  
* `integer` - 转换成数据库的 `INT`
* `long` - `BIGINT`
* `float` - `FLOAT`
* `decimal` - `DECIMAL` 具有规模和精度
* `bool` - `BOOLEAN`
* `char` - `CHAR`
* `varchar` - `VARCHAR` 有长度
* `text` - `TEXT`
* `enumeration` - `INT` 序数值(ordinal value)
* `enumerationByName` - `VARCHAR`
* `customEnumeration` - 详见 [附加部分](#如何使用数据库枚举类型)
* `blob` - `BLOB`
* `binary` - `VARBINARY` 有长度
* `uuid` - `BINARY(16)`
* `reference` - 外键

其中 `exposed-java-time` 扩展模块 (`org.jetbrains.exposed:exposed-java-time:$exposed_version`) 提供了额外的类型:

* `date` - `DATETIME`
* `time` - `TIME`
* `datetime` - `DATETIME`
* `timestamp` - `TIMESTAMP`
* `duration` - `DURATION`

注意：某些类型对于特定的数据库方言是不同的.

## 如何使用数据库枚举类型
一些数据库（例如 MySQL、PostgreSQL、H2）支持显式 ENUM 类型。因为仅使用 jdbc 元数据使这些columns列与 kotlin enumerations枚举保持同步, 可能是一个巨大的挑战, 所以 Exposed 不提供以自动方式管理这些columns列的可能性，但这并不意味着您不能使用这些columns列类型.
您有两个选项可以使用 ENUM 数据库类型:
1. 使用表中现有的 ENUM column列
2. 通过提供原始定义 SQL 从 Exposed 创建column列
   在这两种情况下，您都应该使用 `customEnumeration` 函数（从 0.10.3 版开始可用）

由于jdbc-driver 提供了一个 provide/expect 特殊类给 Enum 类型, 您必须在定义一个 `customEnumeration`时为它提供 from/to 转换函数. 

例如这样的枚举类 `private enum class Foo { Bar, Baz }` 您可以使用如下为数据库提供的代码:

**H2**
```Kotlin
val existingEnumColumn = customEnumeration("enumColumn", { Foo.values()[it as Int] }, { it.name })
val newEnumColumn = customEnumeration("enumColumn", "ENUM('Bar', 'Baz')", { Foo.values()[it as Int] }, { it.name })
```

**MySQL**
```Kotlin
val existingEnumColumn = customEnumeration("enumColumn", { value -> Foo.valueOf(value as String) }, { it.name })
val newEnumColumn = customEnumeration("enumColumn", "ENUM('Bar', 'Baz')", { value -> Foo.valueOf(value as String) }, { it.name })
```

**PostgreSQL**

PostgreSQL 要求将 ENUM 定义为单独的类型，因此您必须在创建表之前创建它。 postgresql jdbc 驱动程序也会为这些值返回 PGobject 实例。下面提供了完整的工作示例.
```Kotlin
class PGEnum<T : Enum<T>>(enumTypeName: String, enumValue: T?) : PGobject() {
    init {
        value = enumValue?.name
        type = enumTypeName
    }
}

object EnumTable : Table() {
    val enumColumn = customEnumeration("enumColumn", "FooEnum", {value -> Foo.valueOf(value as String)}, { PGEnum("FooEnum", it) }
}
...
transaction {
   exec("CREATE TYPE FooEnum AS ENUM ('Bar', 'Baz');")
   SchemaUtils.create(EnumTable)
   ...
}
```


或者... 返回 [主页](Home.md)