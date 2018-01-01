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
    return@transaction Users.select { Users.firstName eq "James" }
}
// jamesList is now a List<ResultRow> containing Users data
```
(In the above example, `return@transaction` is unneccessary, but added for clarity.)

### Advanced parameters and usage

For specific functionality, transactions can be created with the additional parameters `transactionIsolation` and `repetitionAttempts`:

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