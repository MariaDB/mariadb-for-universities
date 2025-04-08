---
title: "Lesson 3: Storage Engines"
url: "/dba/storage-engines/"
date: 2025-04-07
description: >
categories: []
tags: []
weight: 3
toc: true
---

# Storage Engines

## Learning Objectives

- List the available storage engines in MariaDB Enterprise Server, and explain their use cases
- Analyze and evaluate the features of the Aria and InnoDB storage engines

### Storage Engines

```sql
`SHOW ENGINES;`

`CREATE TABLE table1 (col1 INT) ENGINE = Aria;`

`ALTER TABLE table1 ENGINE = InnoDB;`
```

- Multiple storage engines on a server and within a query are allowed
- Determine storage medium (disk, memory, etc.)
- Atomicity Consistency Isolation Durability (ACID)
- Lock at certain levels (table, page, row)
- Some offer special features (foreign keys, GIS, columnar data)
- Some provide optimization

### ACID Transactions

#### Atomicity

All transactions are done completely or rolled back

#### Consistency

Transactions preserve integrity constraints like data types and foreign keys

#### Isolation

Transactions do not interfere; intermediate results are hidden

#### Durability

Completed transactions are permanently recorded

### MariaDB Storage Engines

Optimize for a wide variety of workloads with a single database platform

- MariaDB Enterprise Server
- SQL API
- Pluggable Storage Engine APIs

|                | Aria                   | InnoDB                    | ColumnStore                      | Spider                  | MyRocks                |
|----------------|------------------------|---------------------------|----------------------------------|-------------------------|------------------------|
| **Target**     | Read-Heavy             | Mixed Read/Write          | Analytics                        | Federation              | Write-Heavy            |
| **Added**      | ES 10.2+               | ES 10.4+                  | ES 10.2+                         | ES 10.3+                | ES 10.3+               |
| **Optimizes**  | Used by ES for System Tables | General Purpose Mixed read/writes| Primary Option for Analytics, Mixed loads | Sharding, Interlink     | I/O Reduction, SSD     |
| **Replaces**   | Not recommended, internal use by ES | MySQL, SQL Server, Oracle, etc. | RedShift, Vertica, Snowflake, etc. | ETL tools in many cases | Cassandra, HBase, etc. |

## Aria

### Aria Features

- Designed as replacement for MyISAM
- Non-transactional
- High read, low write traffic
- Many column data types
- Concurrent INSERTs possible
- Supports data-at-rest encryption
- Buffers Pages before Writing Rows
- Index & Pages Cached (`aria_pagecache_buffer_size`)
- Crash Safe — Statement Commit & Rollback (notes a log file)
- Used for Internal On-Disk Temporary Tables
  - Variable `aria_used_for_temp_tables` should be ON (default)

## InnoDB

### InnoDB

#### Features
- Fully transactional, ACID compliant
- Data and indexes cached by `mysqld` in buffer pool
- Supports foreign keys and multi-version concurrency control
- Row-level locking for high concurrency
- Encryption for tables
- Supports four isolation levels
  - Read uncommitted
  - Read committed
  - Repeatable read (no phantom rows; default)
  - Serializable
- Page compression
- Online DDL
- Online backups
- Incremental backups
- Reliable crash recovery
- Typically faster overall performance

![/mariadb-dba-03-09.svg](/mariadb-dba-03-09.svg)

## Lesson Summary

- List the available storage engines in `MariaDB Enterprise Server`, and explain their use cases
- Analyze and evaluate the features of the `Aria` and `InnoDB` storage engines

