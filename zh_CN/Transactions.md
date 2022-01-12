## 概述

Exposed 中的 CRUD 操作必须在一个事务 _transaction_ 中调用. 事务封装了一组 DSL 操作. 要使用默认参数创建和执行事务，只需将函数代码块(function block)传递给事务函数(`transaction` function):
```kotlin
transaction {
    // DSL/DAO operations go here
}
```
多个事务在当前线程上同步执行, 所以他们会 _阻塞_ 你的应用程序的其他部分! 如果您需要异步执行事务，请考虑在单独的 _线程_ (Thread)上运行它.

### 访问返回值

尽管您可以在事务块(transaction block)中修改代码中的变量，但 `事务transaction` 支持直接返回值，从而实现不变性.

```kotlin
val jamesList = transaction {
    Users.select { Users.firstName eq "James" }.toList()
}
// jamesList is now a List<ResultRow> containing Users data
```
*注意:* 如果您不直接加载 `Blob` 和 `text` 字段, 那么它们在事务之外将不可用. 对与 `text` 字段, 在定义表的时，您可以使用 `eagerLoading` 参数, 来使得 `text`字段在事务之外可用.
```kotlin
// without eagerLoading
val idsAndContent = transaction {
   Documents.selectAll().limit(10).map { it[Documents.id] to it[Documents.content] }
}

// with eagerLoading for text fields
object Documents : Table() {
  ...
  val content = text("content", eagerLoading = true)
}

val documentsWithContent = transaction {
   Documents.selectAll().limit(10)
}
```

### 使用多个数据库
_从 0.10.1 版本开始支持此功能_

当您想使用多个不同的数据库时, 您必须存储 `Database.connect()` 返回的数据库引用(reference)并将其作为第一个参数提供给 `事务transaction` 函数.
```kotlin
val db1 = connect("jdbc:h2:mem:db1;DB_CLOSE_DELAY=-1;", "org.h2.Driver", "root", "")
val db2 = connect("jdbc:h2:mem:db2;DB_CLOSE_DELAY=-1;", "org.h2.Driver", "root", "")
transaction(db1) {
   ...
   val result = transaction(db2) {
      Table1.select{ }.map { it[Table1.name] }
   }
   
   val count = Table2.select { Table2.name inList result }.count()
}
```

实体 (见 [[DAO API|DAO]] 页面) `stick` 加载该实体的事务. 这意味着所有更改都保留在同一个数据库中和禁止哪些跨数据库引用并会抛出异常.

### 设置默认数据库
不带参数的 `事务块(transaction block)` 将使用默认数据库.
与 0.10.1 之前一样，这将是最新 _已连接的_ 数据库.
也可以显式设置默认数据库.
```kotlin
val db = Database.connect()
TransactionManager.defaultDatabase = db
```

### 使用嵌套事务
从 Exposed 0.16.1 开始，可以使用嵌套事务。要启用此功能，您应该将所需数据库实例上的 `useNestedTransactions` 设置为 `true`.

任何发生在 `事务块transaction block` 中的异常将不会回滚这个事务, 而只会回滚当前事务中的代码.
Exposed 使用 SQL `SAVEPOINT` 功能, 在 `事务块transaction block` 开始时标记当前事务，并在退出时释放它.

使用 savepoint 可能会影响性能，因此请阅读有关您使用的 DBMS 的文档以获取更多详细信息.

```kotlin
val db = Database.connect()
db.useNestedTransactions = true

transaction {
    FooTable.insert{ it[id] = 1 }
    
    var idToInsert = 0
    transaction { // nested transaction
        idToInsert++
        // On the first insert it will fail with unique constraint exception and will rollback to the `nested transaction` and then insert a new record with id = 2
        FooTable.insert{ it[id] = idToInsert } 
    }
}
```

### 使用协程 Coroutines
如今, 非阻塞和异步代码很流行.Kotlin 的协程[Coroutines](https://kotlinlang.org/docs/reference/coroutines-overview.html)为您提供了一种命令式的异步代码编写方式。大多数 Kotlin 框架（如 [ktor](https://ktor.io)）都内置了对 Coroutines 的支持，而 Exposed 主要是阻塞的.

为什么呢? 

因为 Exposed 使用 JDBC-api 与在阻塞 api 时代设计的数据库进行交互. 更重要的是，Exposed 将一些值存储在线程局部变量(thread-local)中，而协程可以（并且将）在不同的线程中执行. 

从 Exposed 0.15.1 开始，有一些桥接函数可以让您安全地在`suspend` blocks 中与 Exposed 进行交互: `newSuspendedTransaction/Transaction.suspendedTransaction` 具有与阻塞`事务transaction`函数相同的参数，但允许您提供 `CoroutineDispatcher` 函数将在其中执行. 如果未提供上下文，您的代码将在当前 `coroutineContext` 中执行.

示例用法看起来如下:
```kotlin
runBlocking {
    transaction {    
        SchemaUtils.create(FooTable) // Table表将在当前线程被创建
    
        newSuspendedTransaction(Dispatchers.Default) {
            FooTable.insert { it[id] = 1 } // 此插入操作，将在默认调度程序(Default dispatcher)的线程之一中执行 
    
            suspendedTransaction {
                val id = FooTable.select { FooTable.id eq 1 }.single()()[FooTable.id] // 此查询操作也将使用相同事务，在默认调度程序(Default dispatcher)的某个线程上执行
            }
        }
    
        val result = newSuspendedTransaction(Dispatchers.IO) {
            FooTable.select { FooTable.id eq 1 }.single()[H2Tests.Testing.id] // 此查询操作将使用同一事务，在 IO 调度程序(IO dispatcher)的某个线程上执行
        }
    }
}

```  

请注意这样的代码仍然是阻塞的（因为它仍然使用 JDBC）并且您不应该尝试在多个线程之间共享事务，因为它会导致不明确的行为.

如果您希望异步执行某些代码并稍后在代码中使用结果，请查看 `suspendedTransactionAsync` 函数。

```kotlin
val launchResult = suspendedTransactionAsync(Dispatchers.IO, db = db) {
    FooTable.insert{}

    FooTable.select { FooTable.id eq 1 }.singleOrNull()?.getOrNull(Testing.id)
}

println("Result: " + (launchResult.await() ?: -1))

```

此函数将接收与上面的 `newSuspendedTransaction` 相同的参数，但返回 `Deferred`，您可以 `等待await` 以获取您想要的结果.

有时候 `CoroutineDispatcher` 可以更改查询执行顺序，`suspendedTransactionAsync` 始终在新事务中执行，以防止并发问题。

### 高级参数及其用法

对于特定功能，可以使用以下附加参数创建事务: `transactionIsolation`, `repetitionAttempts` and `db`:

```kotlin
transaction (Connection.TRANSACTION_SERIALIZABLE, 2) {
    // DSL/DAO operations go here
}
```
**事务隔离Transaction Isolation:** 此参数是 SQL 标准中定义的参数，指定当多个事务在数据库上同时执行时应该发生什么. 该值不直接影响 Exposed 操作，而是发送到数据库，期望遵守. `java.sql.Connection` 定义了允许的值, 如下:
* **TRANSACTION_NONE**: 不支持事务.
* **TRANSACTION_READ_UNCOMMITTED**: 最宽松的设置。允许来自一个事务的未提交更改影响另一个事务中的读取（“脏读”）.
* **TRANSACTION_READ_COMMITTED**: 此设置可防止发生脏读，但仍允许发生不可重复读。不可重复读取是指一个事务（“事务 A”）从数据库中读取一行，另一个事务（“事务 B”）更改了该行，而事务 A 再次读取该行，导致不一致.
* **TRANSACTION_REPEATABLE_READ**: 公开事务的默认设置。防止脏读和不可重复读，但仍然允许幻读。幻读是指事务（“事务 A”）通过 WHERE 子句选择行列表，另一个事务（“事务 B”）对满足事务 A 的 WHERE 子句的行执行 INSERT 或 DELETE，事务 A 选择再次使用相同的 WHERE 子句，导致不一致.
* **TRANSACTION_SERIALIZABLE**: 最严格的设置。防止脏读、不可重复读和幻读.

> 不可重复读和幻读到底有什么区别呢？
> (1) 不可重复读是读取了其他事务更改的数据，针对update操作
> 解决：使用行级锁，锁定该行，事务A多次读取操作完成后才释放该锁，这个时候才允许其他事务更改刚才的数据。
> (2) 幻读是读取了其他事务新增的数据，针对insert和delete操作
> 解决：使用表级锁，锁定整张表，事务A多次读取数据总量之后才释放该锁，这个时候才允许其他事务新增数据。

**数据库db** 参数是可选的，用于选择应该结算事务的数据库（见上文）.

或者... 返回 [主页](Home.md)