## Overview

CRUD operations in Exposed must be called from within a _transaction._ Transactions encapsulate a set of DSL operations. To create and execute a transaction with default parameters, simply pass a function block to the `transaction` function:
```kotlin
transaction {
    // DSL/DAO operations go here
}
```
Transactions are executed synchronously on the current thread, so they _will block_ other parts of your application! If you need to execute a transaction asynchronously, consider running it on a separate `Thread`.

### Accessing returned values

Although you can modify variables from your code within the transaction block, `transaction` supports returning a value directly, enabling immutability:

```kotlin
val jamesList = transaction {
    Users.select { Users.firstName eq "James" }.toList()
}
// jamesList is now a List<ResultRow> containing Users data
```
*Note:* `Blob` and `text` fields wont be available without transaction if you don't load them directly like:
```kotlin
   val idsAndContent = transaction {
       Documents.selectAll().limit(10).map { it[Documents.id] to it[Documents.content] }
   }
```

### Working with a multiple databases
_This functionality supported since 0.10.1 version_

When you want to work with different databases then you have to store database reference returned by `Database.connect()` and provide it to `transaction` function as first parameter. As before 0.10.1, `transaction` block without parameters will work with the latest _connected_ database.
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

Entities (see [[DAO API|DAO]] page) `stick` to a transaction which was used to load that entity. That means that all changes persist to the same database and what cross-database references are prohibited and will throw exceptions. 

### Advanced parameters and usage

For specific functionality, transactions can be created with the additional parameters: `transactionIsolation`, `repetitionAttempts` and `db`:

```kotlin
transaction (Connection.TRANSACTION_SERIALIZABLE, 2) {
    // DSL/DAO operations go here
}
```
**Transaction Isolation:** This parameter, defined in the SQL standard, specifies what is supposed to happen when multiple transactions execute concurrently on the database. This value does NOT affect Exposed operation directly, but is sent to the database, where it is expected to be obeyed. Allowable values are defined in `java.sql.Connection` and are as follows:
* **TRANSACTION_NONE**: Transactions are not supported.
* **TRANSACTION_READ_UNCOMMITTED**: The most lenient setting. Allows uncommitted changes from one transaction to affect a read in another transaction (a "dirty read").
* **TRANSACTION_READ_COMMITTED**: This setting prevents dirty reads from occurring, but still allows non-repeatable reads to occur. A _non-repeatable read_ is when a transaction ("Transaction A") reads a row from the database, another transaction ("Transaction B") changes the row, and Transaction A reads the row again, resulting in an inconsistency.
* **TRANSACTION_REPEATABLE_READ**: The default setting for Exposed transactions. Prevents both dirty and non-repeatable reads, but still allows for phantom reads. A _phantom read_ is when a transaction ("Transaction A") selects a list of rows through a `WHERE` clause, another transaction ("Transaction B") performs an `INSERT` or `DELETE` with a row that satisfies Transaction A's `WHERE` clause, and Transaction A selects using the same WHERE clause again, resulting in an inconsistency.
* **TRANSACTION_SERIALIZABLE**: The strictest setting. Prevents dirty reads, non-repeatable reads, and phantom reads.

**db** parameter is optional and used to select database where transaction should be settled (see section above).