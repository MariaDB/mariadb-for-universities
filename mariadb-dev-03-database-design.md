---
title: "Lesson 3: Database design"
url: "/dev/database-design/"
date: 2025-04-10
description: >
categories: []
tags: []
weight: 1
toc: true
---

# Database design

## Learning objectives

- Explain the purpose of data modeling and database design
- Describe how MariaDB data types affect performance
- Assign appropriate column options

## Data modeling and database design

### Database mindset

- **Redundant data is eliminated**  
  Easier updating and maintenance

- Atomicity, Isolation, Consistency, Durability (ACID)

- Field have explicit data types

- Flexible and easy to query programmatically

- Accessible by multiple remote, differing clients

- Multiple ways to view the same dataset

- Indexable

### Naming convention

Consistent, Descriptive Names Reduce Mistakes

Plural Form for Table Names (e.g., `products`)

Singular Form for Column Names (e.g., `name_last`, `email`)

Alphabetical Order for Link Tables (e.g., `posts_users`)

Long, descriptive names are better than short, cryptic ones. Use aliases.

Use consistent naming for an `id` column and its references as `table_id`.

### Table design

#### Normalization

- Process of Optimizing and Factoring a Schema
  - 1:1, 1:n and n:n
  - `1NF` / `2NF` / `3NF`
- Makes a Data Set Smaller by Eliminating Redundant or Duplicate Data
  - Problem if Data is Smaller than Key
- Ensures Data Set Consistency and Integrity
- Often Improves Concurrency by Reducing Locking Overhead - Not Always
- Increases Number of Tables and Associated Maintenance

#### Denormalization

- Complete Normalization can Slow Queries
- More Complex `JOIN` Queries are Drains and Harder to Maintain
  - Adding Redundant Data Back to Simplify
  - Problems with `JOIN` Queries
  - Combine 1:n Relationships
  - Add Pre-Computed Columns
  - Use Fake Materialized Views
- Denormalization Adds Redundant Data to Schema
  - Write Queries become More Complex or Numerous as Multiple Locations must be Maintained
- It's Easier to Normalize First, Then Denormalize when Appropriate

### Data relationships

**One-to-One or Zero-to-One**  
Stored generally in the same table  
May be split for performance  

**One-to-Many or Zero-to-Many**  
Many in separate table referenced by one primary key (i.e., a foreign key)  

**Many-to-Many**  
Several tables with one operating as a link  
Linked tables often have a composite primary key  

### Third normal form (3NF)

#### Remove horizontal redundancies
**First normal form**

No single column with more than a single item

No two columns with the same information

#### Each row must be unique
**First normal form**

Use a primary key: Natural or surrogate like `AUTO_INCREMENT`

#### Remove vertical redundancy
**Second normal form**

Same value should not repeat across rows

Data type can play a role (e.g., `ENUM`)

#### Remove vertical redundancy
**Third normal form**

Columns not dependent on primary key are removed

### Character set and collation

Character set may be global or for schema, table or column

Multi-byte character sets increase disk storage and working memory requirements

- UTF-8 Requires 3 or 4 bytes per character

Collations affect string comparison (character order)

Collations can be changed for a query:

```sql
SELECT * FROM table1 ORDER BY col1
COLLATE latin1_german2_ci;
```

### Indexing concept

**Poor Indexing #1 Reason For Poor Performance**

Like most RDBMS, MariaDB resolves queries faster with indexes

Without Indexes, MariaDB does slow Full Table Scans

Indexes can be for a Column or Multiple Columns (i.e., Composites)

In InnoDB, the primary key is added to the end of all indexes on disk

Avoid Indexing Excessively or Arbitrarily
- Read/Write Tradeoff
- Space/Performance Tradeoff

Regularly Remove Unused or Redundant Indexes

Avoid foreign keys due to performance overhead

### Index best practices

Every table should have a Primary Key

- This is particularly important for Replication

Foreign Key should have an index as it helps in the query joins

Create an index on the columns that are frequently used in the `WHERE` clause of the queries

Consider `ORDER BY` clause for possible composite Indexes

- This is very useful of the `WHERE` clause does very less filtering and the `ORDER BY` matches exactly with the `COMPOSITE` Index (Same order of keys, all `ASC`, all `DESC` etc.)

`INDEX` on an `EXPRESSION`, also known as `FUNCTION` based indexes

- `UPPER(First_Name)`, `SUBSTR(Account_No, 3, 5)`
- Not available directly, but can be achieved by defining a `VIRTUAL` column on the table and creating an index on it.

`EXPLAIN` / `ANALYZE` and `PROFILING` queries are the most important tools available when looking for possible Index candidates

### File storage

**Store Files in `BLOB` Useful**

- Included in Back-Up and Replication
- Guaranteed to be Consistent with Other Data

**Store Files in `BLOB` has Problems**

- Database gets Too Large
- MariaDB Can't do Partial Field Read — MyISAM Reads Entire File
- InnoDB can Skip if Field Not Referenced in Query

**Can Minimize by putting Files in Separate Table (1:1 relationship)**

**Store Files in File System and File Name and Path in Table**

- Database is Smaller and More Efficient
- Organize Separate Back-Up, possibly with External Locking for Consistency

### Database design summary

Adjust schema as needs change  
- Don’t over plan or over anticipate for unknown requirements  
- A well normalized schema is usually easy to change  

Consider data growth (a large dataset makes migrations slower and more difficult)  

Don’t optimize prematurely (makes future redesigns more difficult)  

Consider early on the limitations of the system  
- Storage engines may benefit from some non-3NF practices  

Minimize table size on disk and in memory  

Archive table data if possible and appropriate  

Remove duplicate or unused indexes  
- No need to index a column if it’s first field in another index  

Use appropriate data types — smaller is better  

Consider sharding large tables across multiple servers  

Use auto generated columns when possible  
- Act as function based indexes when indexed  

## Lesson summary

- Explain the purpose of data modeling and database design
- Describe how MariaDB data types affect performance
- Assign appropriate column options

## Lab exercises

- 3-1 Designing a Database in Third Normal Form