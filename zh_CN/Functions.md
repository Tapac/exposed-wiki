* [如何使用函数](#如何使用函数)
* [字符串函数](#字符串函数)
* [聚合函数](#聚合函数)
* [自定义函数](#自定义函数)

## 如何使用函数
如果您想从查询中检索出函数的结果，则必须将函数声明为变量，例如:
```kotlin
val lowerCasedName = FooTable.name.lowerCase()
val lowerCasedNames = FooTable.slice(lowerCasedName).selecAll().map { it[lowerCasedName] }

``` 
此外，功能可以链接和组合:
```kotlin
val trimmedAndLoweredFullName = Concat(FooTable.firstName, stringLiteral(" "), FooTable.lastName).trim().lowerCase()
val fullNames = FooTable.slice(trimmedAndLoweredFullName).selecAll().map { it[trimmedAndLoweredFullName] }

```

## 字符串函数
### 大小写转换
返回小写大写字符串值.
```kotlin
val lowerCasedName = FooTable.name.lowerCase()
val lowerCasedNames = FooTable.slice(lowerCasedName).selecAll().map { it[lowerCasedName] }

```


## 聚合函数
这些函数应该在查询中使用 [[分组groupBy|DSL#分组group-by]].
### 最小值/最大值/平均值
返回 最小/最大/平均值, 可以应用于任何可比较的表达式
```kotlin
val minId = FooTable.id.min()
val maxId = FooTable.id.max()
val averageId = FooTable.id.avg()
val (min, max, avg) = FooTable.slice(minId, maxId, averageId).selecAll().map { 
    Triple(it[minId], it[maxId], it[averageId]) 
}

```

## 自定义函数
如果您在数据库中找不到您最喜欢的函数（因为 Exposed 仅对经典 SQL 函数提供基本支持），您可以定义自己的函数.

自 Exposed 0.15.1 以来，通过有多个选项来定义自定义函数:
1. 无参数函数:
```kotlin
val sqrt = FooTable.id.function("SQRT")
```
在 SQL 表示中，它将是 `SQRT(FooTable.id)`

2. 带有附加参数的函数:
```kotlin
val replacedName = CustomFunction<String?>("REPLACE", VarCharColumnType(), FooTable.name, stringParam("foo"), stringParam("bar"))

``` 
`CustomFunction` 类接收函数名作为第一个参数，结果列类型作为第二个参数，之后您可以提供任意数量的参数，并用逗号分隔.

还有一些快捷方式，针对string, long and datetime类型的函数:
* `CustomStringFunction`
* `CustomLongFunction`
* `CustomDateTimeFunction`

上面的代码可以简化为:
```kotlin
val replacedName = CustomStringFunction("REPLACE", FooTable.name, stringParam("foo"), stringParam("bar"))

``` 
