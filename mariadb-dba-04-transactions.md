---
title: "Lesson 4: Transactions"
date: 2025-04-07
description: >
categories: []
tags: []
weight: 4
toc: true
---

# Transactions

## Learning Objectives

- Define and describe what transactions are in a database environment
- Show how to perform transactions and when to use savepoints in MariaDB
- Explain and compare isolation levels and the types of isolation levels in MariaDB

## Introduction to Transactions

### Transactions

A transaction is a mechanism for grouping together multiple SQL statements into one atomic operation; an all or nothing proposition

**Atomicity**  
All statements are executed successfully or are cancelled as a unit

**Consistency**  
A database is in a consistent state when a transaction begins and when a transaction ends

**Isolation**  
One transaction does not affect another

**Durability**  
All changes made by a successfully completed transaction are properly recorded in the database. No changes are lost


### Savepoint

Using savepoints, you can roll back to certain points within the transaction without terminating the transaction.

Use the `SAVEPOINT` statement to set a name for the transaction

```
SAVEPOINT savepoint_name;
```

Use the `ROLLBACK TO` statement to roll back a transaction to the named savepoint without terminating the transaction

```
ROLLBACK TO savepoint_name;
```

Use the `RELEASE SAVEPOINT` statement to remove the specified savepoint from a transaction

```
RELEASE SAVEPOINT savepoint_name;
```

### Autocommit

DDL statements, such as `CREATE` or `DROP` for databases and `CREATE`, `DROP`, or `ALTER` for tables or stored routines cannot be rolled back.

By default, autocommit is ON, which means that all individual statements are committed as soon as they are finished unless they are in a `START TRANSACTION . . . COMMIT` block.

If autocommit is OFF, you need to explicitly issue a `COMMIT` statement to commit a transaction

```sql
COMMIT;
```

To disable autocommit execute `START TRANSACTION`, or `BEGIN`

```sql
START TRANSACTION;   /* or BEGIN */
```

### An Example of a Transaction

```sql
`START TRANSACTION;`  /* Start transaction with START TRANSACTION or BEGIN */
…  /* Perform your SQL statements */
`SAVEPOINT sp1;`  /* Set savepoints sp1 */
…  
`SAVEPOINT sp2;`  /* Set savepoints sp2 */
…  
`SAVEPOINT sp3;`  /* Set savepoints sp3 */

`RELEASE SAVEPOINT sp2;`  /* Deletes savepoint sp2 and all savepoints defined after which includes sp3 */

`ROLLBACK TO sp1;`  /* If anything goes wrong, undo only up to some savepoint */
`ROLLBACK;`  /* If anything goes wrong, undo all changes in the transaction */

`COMMIT;`  /* Else if everything is successful, commit the changes */

/* End of transaction */
```

## Isolation Levels

### Isolation Levels

When two or more transactions occur at the same time, the isolation level defines the degree at which a transaction is isolated from the resource or data modifications made by other transactions

The default isolation level is **REPEATABLE-READ**

To change the isolation level, you need to set the `tx_isolation` variable which is dynamic and has session level scope

```sql
SET SESSION tx_isolation = 'READ-COMMITTED';
```

The `transaction_isolation` option can be set in the configuration file

```ini
[mariadb]
transaction_isolation = READ-COMMITTED
```

### Types of Isolation Levels

**Read Uncommitted**  
Allows a transaction to see uncommitted changes made by other transactions (*dirty read*)

**Read Committed**  
Allows a transaction to see changes made by other transactions (different row content or fewer rows due to a `DELETE`, or rows no longer match due to an `UPDATE`) only if the changes have been committed (non-repeatable read)

**Repeatable Read**  
Ensures that if a transaction issues the same `SELECT` twice, no rows vanish or show different content. New rows (phantom rows) can still appear

Due to the implementation of row locks in InnoDB there are no phantom rows in InnoDB

**Serializable**  
Completely isolates a transaction's effects from other transactions

In InnoDB this causes read locks after `SELECT` until `COMMIT` or `ROLLBACK` (like `WITH LOCK IN SHARE MODE`)

## Lesson Summary

- Define and describe what transactions are in a database environment
- Show how to perform transactions and when to use savepoints in MariaDB
- Explain and compare isolation levels and the types of isolation levels in MariaDB

