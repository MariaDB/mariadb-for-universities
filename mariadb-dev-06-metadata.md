---
title: "Lesson 6: Metadata"
url: "/dev/metadata/"
date: 2025-04-10
description: >
categories: []
tags: []
weight: 1
toc: true
---

# Metadata

## Learning objectives

- List the various metadata access methods available
- Understand the structure of the `INFORMATION_SCHEMA`, and list the differences between the `SHOW` statements and the `INFORMATION_SCHEMA`
- Use query profiling to improve query performance
- Describe features of the performance schema

## Information schema

### Information schema details

- Pseudo database that holds metadata on schemas
- Generated as needed
- No on-disk presence
- Read-only tables
- Plugins can install additional information_schema tables (e.g. `METADATA_LOCK_INFO`)
- Useful for schema design, redesign and migration
- Provides much more information than `SHOW` statements and is ANSI/ISO SQL:2003
- Do NOT automate queries in production as this can affect performance

```sql
        CHARACTER_SETS
        COLLATION_CHARACTER_SET_APPL
        LICALBILITY
        COLLATIONS
        COLUMN_PRIVILEGES
        COLUMNS
        ENGINES
        EVENTS
        FILES
        GLOBAL_STATUS
        GLOBAL_VARIABLES
        KEY_COLUMN_USAGE
        PARTITIONS
        PLUGINS
        PROCESSLIST SCHEMATA
        PROFILING
        REFERENTIAL_CONSTRAINTS
        ROUTINES
        SCHEMA_PRIVILEGES
        SESSION_STATUS
        SESSION_VARIABLES
        STATISTICS
        TABLE_CONSTRAINTS
        TABLE_PRIVILEGES
        TABLES
        TRIGGERS
        USER_PRIVILEGES
        VIEWS
```

### Methods of accessing the information schema

#### Accessing directly

```sql
SELECT TABLE_SCHEMA,ENGINE,COUNT(*) FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA NOT IN('mysql','information_schema','performance_schema') GROUP BY TABLE_SCHEMA,ENGINE;
```

```sql
SELECT * FROM information_schema.global_status WHERE VARIABLE_NAME LIKE '%qcache%';
```

#### Using show statements

```sql
SHOW STATUS LIKE '%qcache%';
```

## Query profiling

### Query profiling

The `SHOW PROFILE` and `SHOW PROFILES` statements display profiling information that indicates resource usage for statements executed during the current session

Profiling is enabled by

`SET profiling=1;`

`SHOW PROFILES` displays a list of the most recent statements

List size is controlled by profiling_history_size session variable (default: 15 / max. : 100)

### Query profiling example

Enable profiling and `SELECT ... WHERE` for non-indexed column

```sql
MariaDB [(none)]> SET profiling = 1;
MariaDB [(none)]> USE test;
Database changed

MariaDB [test]> select id, pad from test.sbtest1 where id=1000000;

MariaDB [test]> select id,pad from test.sbtest1 where pad='96658133632-61067815236-70200144435-65878686334-75374878237';
```

### Query profiling example

Without an index execution time becomes much longer  
So an index is added to the pad column

```sql
MariaDB [test]> SHOW PROFILES;

+----------+-----------+-----------------------------------------------------+
| Query_ID | Duration  | Query                                               |
+----------+-----------+-----------------------------------------------------+
|        1 | 0.00042851| SELECT DATABASE()                                   |
|        2 | 0.00283556| show databases                                      |
|        3 | 0.00033463| show tables                                         |
|        4 | 0.00012415| select id, pad from test.sbtest1 where id=1000000   |
|        5 | 0.32615275| select id,pad from test.sbtest1 where               |
|          |           | pad='9656133362-61067815236-70200144435-6587866334-75374878237' |
+----------+-----------+-----------------------------------------------------+
```

### Query profiling example

```sql
MariaDB [test]> ALTER TABLE sbtest1 ADD INDEX(pad);
MariaDB [test]> select id, pad from test.sbtest1 where pad='96558133562-61076...
```

Adding an index for the pad column to avoid a full scan, speeds up the SELECT query

```sql
MariaDB [test]> MariaDB [test]> show profiles;
+----------+------------+--------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query
|
+----------+------------+--------------------------------------------------------------------------------------+
|        1 | 0.00048251 | SELECT DATABASE()
|
|        2 | 0.00283556 | show databases
|
|        3 | 0.00034363 | show tables
|
|        4 | 0.00042145 | select id, pad from test.sbtest1 where id=1000000
|
|        5 | 0.32615275 | select id,pad from test.sbtest1 where
pad='96658133632-61067815236-70200144435-65878686334-75374878237' |
|        6 | 3.95204536 | ALTER TABLE sbtest1 ADD INDEX(pad)
|
|        7 | 0.00070418 | select id,pad from test.sbtest1 where
pad='96658133632-61067815236-70200144435-65878686334-75374878237' |
+----------+------------+--------------------------------------------------------------------------------------+
```

## Lesson summary

- List the various metadata access methods available
- Understand the structure of the `INFORMATION_SCHEMA`, and list the differences between the `SHOW` statements and the `INFORMATION_SCHEMA`
- Use query profiling to improve query performance
- Describe features of the performance schema

## Lab exercises

- 8-1 Collecting Server Information
- 8-2 Using the Information Schema’s Profiling Table