CREATE DATABASE world;view---
title: "Lesson 2: Schema Objects"
url: "/dba/schema-objects/"
date: 2025-04-07
description: >
categories: []
tags: []
weight: 2
toc: true
license_note: "For academic and non-commercial usage, licensed under CC BY-NC-SA 4.0 by MariaDB plc."
license: "CC BY-NC-SA 4.0"
license_url: "https://creativecommons.org/licenses/by-nc-sa/4.0/"
---

# Schema Objects

## Learning Objectives

- Identify and describe the schema objects available within MariaDB Enterprise Server
- Demonstrate how to create and use databases, tables, and columns in MariaDB Enterprise Server
- Compare and contrast the data types and built-in functions in MariaDB Enterprise Server, and explain their use cases

### Collations and Character Sets

- A collation is a set of rules that defines how to compare and sort strings

- A character set defines how and which characters are stored to support a particular language or languages

- Only applicable to text fields such as `VARCHAR` or `TEXT`

- The default character set is `latin1` and the default collation is `latin1_swedish_ci`

- Collation and character set are specified at table or column level

### Listing Character Sets

```sql
MariaDB [(none)]> SHOW CHARACTER SET;

| Charset | Description                 | Default collation | Maxlen |
|---------|-----------------------------|-------------------|--------|
| big5    | Big5 Traditional Chinese    | big5_chinese_ci   | 2      |
| dec8    | DEC West European           | dec8_swedish_ci   | 1      |
| cp850   | DOS West European           | cp850_general_ci  | 1      |
| hp8     | HP West European            | hp8_english_ci    | 1      |
| koi8r   | KOI8-R Relcom Russian       | koi8r_general_ci  | 1      |
| latin1  | cp1252 West European        | latin1_swedish_ci | 1      |
| latin2  | ISO 8859-2 Central European | latin2_general_ci | 1      |
| swe7    | 7bit Swedish                | swe7_swedish_ci   | 1      |
| ascii   | US ASCII                    | ascii_general_ci  | 1      |
[...]
```

### Listing Collations

```sql
MariaDB [(none)]> SHOW COLLATION LIKE 'utf8%';
+-----------------------+---------+-----+---------+----------+---------+
| Collation             | Charset | Id  | Default | Compiled | Sortlen |
+-----------------------+---------+-----+---------+----------+---------+
| utf8mb3_general_ci    | utf8mb3 |  33 |    Yes  |    Yes   |       1 |
| utf8mb3_bin           | utf8mb3 |  83 |         |    Yes   |       1 |
| utf8mb3_unicode_ci    | utf8mb3 | 192 |    Yes  |    Yes   |       8 |
| utf8mb3_icelandic_ci  | utf8mb3 | 193 |         |    Yes   |       8 |
| utf8mb3_latvian_ci    | utf8mb3 | 194 |         |    Yes   |       8 |
| utf8mb3_romanian_ci   | utf8mb3 | 195 |         |    Yes   |       8 |
| utf8mb3_slovenian_ci  | utf8mb3 | 196 |         |    Yes   |       8 |
| utf8mb3_polish_ci     | utf8mb3 | 197 |         |    Yes   |       8 |
| utf8mb3_estonian_ci   | utf8mb3 | 198 |         |    Yes   |       8 |
| utf8mb3_spanish_ci    | utf8mb3 | 199 |         |    Yes   |       8 |
...
+-----------------------+---------+-----+---------+----------+---------+
```

## Databases, Tables, and Default Schemas

### Case Sensitivity

Depends on operating system, file system and `lower_case_table_names` configuration

Set before starting a project (strongly recommended!)

Usually case sensitive by default on Linux, but not on Windows and MacOS

Adopt a convention such as always creating and referring to databases and tables using lowercase names

Most are limited to 64 characters

### Databases

**Database aka Schema**

Highest level object

Corresponds to a directory within the data directory

All other objects reside within user-created databases however certain objects such as user accounts, roles, stored procedures and plugins reside within system databases.

```sql
-- create new database with name 'world'
CREATE DATABASE `world`;

-- list of exists databases
SHOW DATABASES;
```

### Tables

- Stores rows of structured data organized by typed columns

- Each corresponds to a metadata file (`.frm`) and data file(s), dependent on the storage engine  
  *(i.e. InnoDB tablespaces)*

- Qualified by a database name  
  *(i.e. database.table)*

### Columns

Set of typed data values

Qualified by a table name  
(i.e. table.column)

Stored together in each row (MariaDB Server) within pages, within tablespaces or data files

Generated columns are stored if they are created as `PERSISTENT` | `STORED` though `VIRTUAL` columns are not stored

Index

### Indexes

- Exact copy of selected columns followed by primary key (e.g., name, address, ID)
  - For long columns the index might not be an "exact copy", but instead it might contain a prefix of the column
- Fast lookup of data within a table without having to scan all columns for each row
- For InnoDB the primary key is silently appended to the end of secondary indexes, unless specified elsewhere within an index
- Attribute of a table

### Constraints

- Types: Primary Key, Foreign Key, Unique, and Check
- Support and implementation differs per storage engine
- Allows data within column(s) to be limited or constrained to a set of values
- Usually defined with a corresponding index
- Attribute of a table
- Has potential for contention at scale

### AUTO_INCREMENT

- Use `LAST_INSERT_ID()` to obtain the value generated for the client connection
- `SERIAL` is a synonym for `BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE`
- In Aria the counter can be set back manually if the counter value wraps
- InnoDB prepares `AUTO_INCREMENT` counters when MariaDB Server starts
  - Single mutex on a table, behavior changed with `innodb_autoinc_lock_mode`

| 0 | 1 | 2 |
| :--- | :--- | :--- |
| **Traditional** | **Consecutive** | **Interleaved** |
| Default | | |
| Holds table-level lock for all `INSERT`s until end of statement | Holds table-level lock for all bulk `INSERT`s (such as `LOAD DATA` or `INSERT ... SELECT`) until end of statement | No table-level locks are held ever |
| | For simple `INSERT`s, no table-level lock held | Fastest and most scalable |
| | | Not safe for statement-based replication |
| | | Generated IDs are not always consecutive |

### Column Attributes

Columns have strict type definitions

Can specify DEFAULT value for column

Only the primary key can be automatically incremented

```sql
CREATE TABLE people (
    id INT AUTO_INCREMENT KEY,
    name VARCHAR(20) DEFAULT 'unknown'
);
```

Columns can be NULL, unless defined NOT NULL
- NULL means "No Value", "Not Applicable", or "Unknown"
- Use NULL when value is not an empty string
- NOT NULL
  - Reduces storage in some storage engines
  - Can also reduce execution time because there are more CPU cycles used to first check for NULL

### Views

- Virtual table defined by a SQL `SELECT` query
- Evaluated at each access
  - No materialized views
- Treated as a table for many purposes, shares namespace with tables
- Updateable in certain cases (`WITH CHECK OPTION`)
- Qualified by a database (`database.view`)

### Using Views

Customers Table

```shell
| ID | FirstName | LastName |
|----|-----------|----------|
| 1  | Alice     | Evans    |
| 2  | Bob       | Smith    |
```

Addresses Table

```shell
| CustomerID | Address1          | City    |
|------------|-------------------|---------|
| 1          | 123 Main Street   | Anytown |
| 2          | 456 Spruce Street | Anyburg |
```

Create CustomerAddresses View as select from Customers and Addresses tables:

```sql
CREATE VIEW CustomerAddresses AS
SELECT
    CONCAT(c.FirstName, ' ', c.LastName) as FullName,
    CONCAT(a.Address1, ', ', a.City) as FullAddress
FROM Customers c
JOIN Addresses a ON c.id = a.CustomerId;
```

Fetch data from CustomerAddresses View

```sql
SELECT FullName, FullAddress FROM CustomerAddresses;
```
```shell
| FullName    | FullAddress               |
|-------------|---------------------------|
| Alice Evans | 123 Main Street, Anytown  |
| Bob Smith   | 456 Spruce Street, Anyburg|
```

### Stored Routines

- Types: Functions, Triggers, Events, and Stored Procedures
- Has input and output parameters
- Reusable SQL Code
  - SQL/PSM (default)
  - SQL/PL which is a compatible subset of PL/SQL (`sql_mode=ORACLE`)
- Can allow cursor loops
- Runtime script, not compiled or binary
- Lack of good debugging, can be hard to profile
- Can be bad for statement based binary logging
- Stored routine uses the privileges of the user that defined it (user can be changed with the `DEFINER` clause)
- If `SQL SECURITY INVOKER` is set then the privileges of the user executing the stored routine are used
- Qualified by database (`database.my_stored_procedure`)

### Creating Your First Table

```sql
CREATE TABLE city (
    ID INT(11) NOT NULL AUTO_INCREMENT,
    Name CHAR(35) NOT NULL DEFAULT '',
    CountryCode CHAR(3) NOT NULL DEFAULT '',
    District CHAR(20) NOT NULL DEFAULT '',
    Population INT(11) NOT NULL DEFAULT '0',
    PRIMARY KEY (ID),
    KEY CountryCode (CountryCode),
    CONSTRAINT FOREIGN KEY (CountryCode)
        REFERENCES country (Code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

- Name and type of object being created
- Column definitions
- Define primary key, secondary indexes and constraints
- Define engine, partitions, character set and other attributes

### Altering Tables

`ALTER TABLE` IS USED TO CHANGE A TABLE'S SCHEMA

**ADD COLUMN** to add a column

**DROP COLUMN** to drop a column *(deletes data!)*

**CHANGE COLUMN** and **MODIFY COLUMN** to alter a column

```sql
ALTER TABLE table1
ADD COLUMN col5 CHAR(8),
DROP COLUMN col3,
CHANGE COLUMN col4 col6 DATE,  # new column name col6
MODIFY COLUMN col8 VARCHAR(10);
```

Basic syntax example for `ALTER TABLE` statement

### Temporal Tables

```sql
| Type                   | Tracks                          | Sample Use Cases                         |
|------------------------|---------------------------------|------------------------------------------|
| System-Versioned       | Change history                  | Audit, forensics, IoT temperature tracking |
| Application-Time Period| Time-limited values             | Sales offers, subscriptions              |
| Bitemporal             | Time-limited values with history| Schedules, decision support models       |
```

*mariadb-dump* does not read historical rows from versioned tables, and so historical data will not be backed up.

### Temporal Tables

#### System-Versioned Example

```sql
CREATE TABLE accounts (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255),
    amount INT
) WITH SYSTEM VERSIONING;
```

#### Application-Time Period Example

```sql
CREATE TABLE coupons (
    id INT UNSIGNED,
    date_start DATE,
    date_end DATE,
    PERIOD FOR valid_period(date_start, date_end)
);
```

#### Bitemporal Example

```sql
CREATE TABLE coupons_new (
    id INT UNSIGNED,
    name VARCHAR(255),
    date_start DATE,
    date_end DATE,
    PERIOD FOR valid_period(date_start, date_end)
) WITH SYSTEM VERSIONING;
```

### Default Schemas

#### information_schema
- The encyclopedia of your database
- Contains information on databases, tables, columns, procedures, indexes, statistics, partitions and views
- Plugins can install additional information_schema tables
- Thread pool information tables
- Keyword and SQL functions

#### mysql
Stores information on:
- Server configuration
- Grants and timezones
- User accounts
- Roles
- Stored procedure definitions
- Stored function definitions
- Event definitions
- Plugins installed by `INSTALL SONAME`/`INSTALL PLUGIN`

#### performance_schema
- Metrics and performance data
- Covered in detail in our Performance Tuning class

### Default Schemas

#### `sys_schema`

- Added in MariaDB 10.6
- Tracks configuration changes in `sys_config` table
- Has views, functions, and stored procedures for getting detailed metrics

### Information Schema Details

- Pseudo database that holds metadata on schemas
- Generated as needed
- No on-disk presence
- Read-only tables
- Plugins can install additional information schema tables (e.g. `METADATA_LOCK_INFO`)
- Useful for schema design, redesign and migration
- Provides much more information than `SHOW` statements and is ANSI/ISO SQL:2003
- Do NOT automate queries in production as this can affect performance
  
```sql
  CHARACTER_SETS  
  COLLATION_CHARACTER_SET_APP  
  LICABILITY  
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

### Methods of Accessing the Information Schema

**Accessing Directly (Using SELECT from relevant tables)**

```sql
SELECT
    TABLE_SCHEMA,
    ENGINE,
    COUNT(*)
FROM information_schema.TABLES
WHERE TABLE_SCHEMA NOT IN('mysql','information_schema','performance_schema')
GROUP BY TABLE_SCHEMA, ENGINE;

SELECT *
FROM information_schema.global_status
WHERE VARIABLE_NAME LIKE '%qcache%';
```

**Using Show Statements**

```sql
SHOW STATUS LIKE '%qcache%';
```

### Crash Safe System Tables

#### mysql Schema

`SELECT ENGINE, TABLE_NAME FROM information_schema.tables WHERE TABLE_SCHEMA="mysql";`

- Most system tables will show an ENGINE of ARIA
- However some system tables will show an ENGINE as CSV
  - `slow_log`
  - `general_log`
- Some INNODB related tables will remain INNODB tables
  - `mysql.gtid_slave_pos`
  - `mysql.innodb_index_stats`
  - `mysql.innodb_table_stats`
  - `mysql.transaction_registry`

## Data Types

### Data Types

Types: Binary, Numeric, String, Temporal, and User Defined

Use the most suitable data type to store all possible, required values  
Will truncate silently and round depending on `sql_mode`

```sql
MariaDB [(none)]> help INT;
Name: 'INT'
Description: INT[(M)] [UNSIGNED] [ZEROFILL]

A normal-size integer. 
The signed range is -2,147,483,648 to 2,147,483,647.
The unsigned range is 0 to 4,294,967,295.
```

### Numeric Data Types

- TINYINT
- SMALLINT
- MEDIUMINT
- INTEGER,INT
- BIGINT

- Use `UNSIGNED` when appropriate
- `INT(n)` specifies display precision, not storage precision
- Size and precision is storage engine dependent
- Define handling of out-of-range values with `sql_mode`
  - Default Mode: values are truncated silently
  - Strict Mode: errors are generated
- `BIGINT` can enumerate more than all the ants on Earth and shouldn’t be your default choice
- `TINYINT(1)` is used for `BOOLEAN` values and is aliased by the `BOOLEAN` type

- FLOAT
- DOUBLE
- DECIMAL
- NUMERIC
- REAL

- `FLOAT` and `DOUBLE` are approximate types
  - Uses 4 and 8 bytes IEEE storage format
- `DECIMAL (m, d)` maximum total number of digits, number of digits after decimal point
  - An Exact Value type, up to 65 digits precision, 4 bytes storage for each multiple of nine digits
- `NUMERIC` is a synonym for `DECIMAL`
- `REAL` is a synonym for `DOUBLE`
  - Unless in `REAL_AS_FLOAT` SQL mode

### String data types

MariaDB provides several string data types to store character data efficiently. These types are used for everything from single-character codes to large-scale text documents, and they are defined by their maximum length, character set, and collation.

- `CHAR(n)` - Number of characters, not bytes, wide
  - Always stores n characters
  - Automatically pads with spaces for shorter strings
- `VARCHAR(n)` - Variable length up to maximum n characters
  - Changes to `CHAR` in Implicit Temporary Tables and mysqld internal buffers
  - 256 characters and longer treated as `TEXT`
  - For InnoDB, this maximum will depend on the row format
- `TEXT` - Large text object
  - Up to 65,535 (2^16 - 1) characters
  - Not supported by the `MEMORY` Storage Engine
  - MariaDB uses `ARIA` for implicit on-disk temporary tables
- `TINYTEXT` - Text type limited up to 255 (2^8 - 1) characters
- `MEDIUMTEXT` - Text type limited up to 16,777,215 (2^24 - 1) characters
- `LONGTEXT` - Text type limited up to 4,294,967,295 (2^32 - 1) characters

#### String data attributes

String columns are defined by a **Character Set** and a **Collation**. These can be inherited from the server or database, or set specifically at the table or column level.

- **Character Sets**: Determine how characters are encoded. Modern MariaDB (10.6+) defaults to `utf8mb4` for full Unicode support. Multi-byte sets provide broader character support but increase disk storage and memory requirements.
- **Collations**: A set of rules for comparing and sorting strings. Modern MariaDB uses UCA (Unicode Collation Algorithm) based collations, such as `utf8mb4_uca1400_ai_ci`, which offer improved linguistic accuracy and can be overridden within specific queries.

Column table and column collation may be defined on table creation. Example below shows couple of variants:
```sql
CREATE TABLE t (
    -- colums with charset latin1 and defalult collation
    latin_name text CHARSET latin1,
    -- column with default charset and utf8mb4_general_ci collation
    utf8mb4_name text COLLATE utf8mb4_general_ci,
    -- column with certain charset and collation
    utf8mb3_name text CHARSET utf8mb3 COLLATE utf8mb3_general_ci  
);
```

How to inspect Character Set and Collation? One of methods - call `CREATE TABLE <table name>;` command.
```shell
|-------|---------------------------------------------------------------------------------------|
| Table | Create Table                                                                          |
|-------|---------------------------------------------------------------------------------------|
| t     | CREATE TABLE `t` (                                                                    |
|       |   `latin_name` text CHARACTER SET latin1 COLLATE latin1_swedish_ci DEFAULT NULL,      |
|       |   `utf8mb4_name` text CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,  |
|       |   `utf8mb3_name` text CHARACTER SET utf8mb3 COLLATE utf8mb3_general_ci DEFAULT NULL   |
|       | ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_uca1400_ai_ci                 |
```

Try this on [SQLize.online](https://sqlize.online/sql/mariadb118/7a04936ecb50e04de1745033909387e4/)

### Binary Data Types

- BINARY
- VARBINARY
- TINYBLOB
- BLOB
- MEDIUMBLOB
- LONGBLOB

- `BINARY`, `VARBINARY` and `BLOBs` can contain data with bytes from the whole range from 0 - 255
- Uses a special character set and collation called "binary"
- Blobs are often used to store files in a database
  - Files on disk are often faster
  - But referential integrity is not guaranteed
- Blobs are included in transactions, replication, and backups
- Blobs inflate `mysqld` memory usage
- Modern InnoDB has some improvements in storage and lookup of blobs

### Temporal Data Types

- DATE
- TIME
- DATETIME
- TIMESTAMP
- YEAR

```sql
SELECT CURTIME(4);
```

```shell
+---------------+
| CURTIME(4)    |
+---------------+
| 05:33:09.1061 |
+---------------+
```

- `DATE` — from 1000-01-01 to 9999-12-31
  - YYYY-MM-DD
- `TIME` [(<microsecond precision>)]
  - from -838:59:59 to 838:59:59
- `DATETIME` [(<microsecond precision>)]
  - Same ranges as `DATE` and `TIME` above
  - YYYY-MM-DD HH:mm:ss
- `TIMESTAMP` — Unix timestamp, in seconds from 1970-01-01
  - Many applications store `UNIX_TIMESTAMP()` values in unsigned integer field
- `YEAR` — Accepts YYYY

### JSON Data Type

- JSON is a language-independent data format.
- JSON documents can be validated for correct syntax with `JSON_VALID()` used as a `CHECK` constraint.
- MariaDB supports a full set of JSON functions.
- JSON columns can be indexed by values of an attribute.
- The indexes are created by using virtual generated columns.

```sql
CREATE TABLE city (
  Name VARCHAR(35) NOT NULL,
  Info JSON DEFAULT NULL
);

INSERT INTO city VALUES (
  'New York',
  JSON_OBJECT(
    'Population','8008278',
    'Country', 'USA'
  )
);
```

```sql
SELECT
  Name, JSON_VALUE(Info,'$.Population') AS Population FROM city;

+---------+------------+
| Name    | Population |
+---------+------------+
| New York|  8008278   |
+---------+------------+
```

```sql
SELECT * FROM city;

+----------+--------------------------------------------+
| Name     | Info                                       |
+----------+--------------------------------------------+
| New York | {"Population": "8008278", "Country":"USA"} |
+----------+--------------------------------------------+
```

### Special Data Types

- **ENUM** is an enumerated list of string values
  - Holds one of the values listed
  - Stored as 2-byte integer index, presented as value

  ```sql
  CREATE TABLE country (
      Continent ENUM('Asia','Europe','N America','Africa','Oceania','Antarctica','S America')
  );
  ```

- **SET** is a specified list of string values
  - Can hold one or more values from the defined set

  ```sql
  CREATE TABLE countrylanguage (
      CountryCode CHAR(3),
      Language SET ('English','French','Mandarin','Spanish')
  );

  INSERT INTO countrylanguage VALUES
      ('CHN','Mandarin'),
      ('CAN','English,French');
  ```

- **INET6** is a data type for storing IPv6 IP addresses as well as IPv4
  - Stores as a BINARY(16)
  
  ```sql
  CREATE TABLE ipaddress (address INET6);
  ```

## Built-in Functions

### MariaDB Date and Time Functions:

- **ADDDATE()** - Adds a time interval to a date value.
- **ADDTIME()** - Adds a specialized time interval (hours, minutes, seconds, microseconds) to a time or datetime.
- **CONVERT_TZ()** - Converts a datetime value from one time zone to another.
- **CURDATE()** - Returns the current date in 'YYYY-MM-DD' or YYYYMMDD format.
- **CURTIME()** - Returns the current time in 'HH:MM:SS' or HHMMSS format.
- **DATE()** - Extracts the date part of a date or datetime expression.
- **DATE_ADD()** - Adds a time interval (INTERVAL) to a date or datetime value.
- **DATE_FORMAT()** - Formats a date value according to a specified format string.
- **DATE_SUB()** - Subtracts a time interval (INTERVAL) from a date or datetime value.
- **DATEDIFF()** - Returns the number of days between two date values.
- **DAYNAME()** - Returns the name of the weekday for a given date.
- **DAYOFMONTH()** - Returns the day of the month (1 through 31) for a date.
- **DAYOFWEEK()** - Returns the weekday index (1 = Sunday, 2 = Monday, ..., 7 = Saturday).
- **DAYOFYEAR()** - Returns the day of the year (1 through 366) for a date.
- **EXTRACT()** - Extracts a specific part (e.g., YEAR, MONTH, DAY) from a date or datetime.
- **FROM_DAYS()** - Converts a numeric day count (from year 0) into a date value.
- **FROM_UNIXTIME()** - Converts a Unix timestamp into a date or datetime string.
- **GET_FORMAT()** - Returns a format string for various date/time standards (ISO, USA, etc.).
- **HOUR()** - Returns the hour part (0 through 23) of a given time or datetime.
- **LAST_DAY()** - Returns the last day of the month for a given date value.
- **MAKEDATE()** - Returns a date value generated from a year and a day of the year.
- **MAKETIME()** - Returns a time value generated from hour, minute, and second components.
- **MICROSECOND()** - Returns the microseconds (0 through 999999) from a time or datetime.
- **MINUTE()** - Returns the minute part (0 through 59) of a given time or datetime.
- **MONTH()** - Returns the month part (1 through 12) of a given date.
- **MONTHNAME()** - Returns the full name of the month for a given date.
- **NOW()** - Returns the current date and time when the statement began executing.
- **PERIOD_ADD()** - Adds a specified number of months to a period (formatted as YYMM or YYYYMM).
- **PERIOD_DIFF()** - Returns the number of months between two periods (formatted as YYMM or YYYYMM).
- **QUARTER()** - Returns the quarter of the year (1 through 4) for a given date.
- **SEC_TO_TIME()** - Converts a value in seconds into a 'HH:MM:SS' time format.
- **SECOND()** - Returns the second part (0 through 59) of a given time or datetime.
- **STR_TO_DATE()** - Converts a string into a date or datetime based on a format string.
- **SUBDATE()** - Subtracts a time interval from a date value (synonym for DATE_SUB).
- **SUBTIME()** - Subtracts a time interval from a time or datetime value.
- **SYSDATE()** - Returns the current date and time at the exact moment the function is called.
- **TIME()** - Extracts the time part of a given datetime or time expression.
- **TIME_FORMAT()** - Formats a time value according to a specified format string.
- **TIME_TO_SEC()** - Converts a time value into the total number of seconds.
- **TIMEDIFF()** - Returns the difference between two time or datetime values.
- **TIMESTAMP()** - Returns a datetime value from a date/datetime or adds a time to an expression.
- **TIMESTAMPADD()** - Adds a specified interval (unit) to a date or datetime expression.
- **TIMESTAMPDIFF()** - Returns the difference between two date or datetime expressions in a specified unit.
- **TO_DAYS()** - Returns the number of days from year 0 to the given date.
- **UNIX_TIMESTAMP()** - Returns a Unix timestamp (seconds since '1970-01-01 00:00:00' UTC).
- **UTC_DATE()** - Returns the current UTC date.
- **UTC_TIME()** - Returns the current UTC time.
- **UTC_TIMESTAMP()** - Returns the current UTC date and time.
- **WEEK()** - Returns the week number for a given date.
- **WEEKDAY()** - Returns the weekday index (0 = Monday, 1 = Tuesday, ..., 6 = Sunday).
- **WEEKOFYEAR()** - Returns the calendar week of the date (1 through 53).
- **YEAR()** - Returns the year part of a given date.
- **YEARMONTH()** - Returns the year and month value (often used as an extraction unit or in periodic calculations).

### Examples of Date and Time Functions

**Used in Queries and Data Manipulation Statements**

```sql
SELECT NOW() + INTERVAL 1 DAY
           - INTERVAL 1 HOUR
       AS 'Day & Hour Earlier';
```

```sql
+---------------------+
| Day & Hour Earlier  |
+---------------------+
| 2020-06-02 08:32:44 | 
+---------------------+
```

**Used in WHERE Clauses**

```sql
UPDATE table1
SET col3 = 'today', col4 = NOW()
WHERE col5 = CURDATE();
```

**Used in Bulk Load**

```sql
load data local infile '/tmp/test.csv' into
table test fields terminated by ','
ignore 1 lines (id,@dt1)
set dt=str_to_date(@dt1,'%d/%m/%Y');
```

### Manipulating Strings

### MariaDB Functions For String Manipulation

- **ASCII()** - Returns the numeric ASCII value of the leftmost character.
- **BIN()** - Returns a string representation of the binary value of a number.
- **BINARY** - Casts a string to a binary string.
- **BIT_LENGTH()** - Returns the length of a string in bits.
- **CAST()** - Converts a value from one data type to another.
- **CHAR()** - Returns the character for each integer passed.
- **CHARACTER_LENGTH()** - Returns the length of a string in characters.
- **CHAR_LENGTH()** - Synonym for `CHARACTER_LENGTH()`.
- **COALESCE()** - Returns the first non-NULL value in a list.
- **CONCAT()** - Concatenates two or more strings.
- **CONCAT_WS()** - Concatenates strings with a separator.
- **CONVERT()** - Converts a value to a different data type or character set.
- **ELT()** - Returns the string at the specified index from a list.
- **EXPORT_SET()** - Returns a string where bits in a value determine the inclusion of "on" or "off" strings.
- **EXTRACTVALUE()** - Extracts a value from an XML string using XPath.
- **FIELD()** - Returns the index of a string within a list of strings.
- **FIND_IN_SET()** - Returns the index of a string within a comma-separated list.
- **FORMAT()** - Formats a number with a specific number of decimal places and locale.
- **FROM_BASE64()** - Decodes a base64-encoded string.
- **HEX()** - Returns a hexadecimal representation of a value.
- **INSERT()** - Inserts a substring into a string at a specific position, replacing a number of characters.
- **INSTR()** - Returns the position of the first occurrence of a substring.
- **LCASE()** - Synonym for `LOWER()`.
- **LEFT()** - Returns a specified number of characters from the left of a string.
- **LENGTH()** - Returns the length of a string in bytes.
- **LIKE** - Simple pattern matching using `%` and `_`.
- **LOAD_FILE()** - Reads a file and returns the content as a string.
- **LOCATE()** - Returns the position of the first occurrence of a substring.
- **LOWER()** - Converts a string to lowercase.
- **LPAD()** - Left-pads a string with another string to a certain length.
- **LTRIM()** - Removes leading spaces from a string.
- **MAKE_SET()** - Returns a comma-separated list of strings based on bits set in a value.
- **MATCH AGAINST** - Performs full-text searches.
- **MID()** - Synonym for `SUBSTRING()`.
- **NOT LIKE** - Negation of the `LIKE` pattern matching.
- **NOT REGEXP** - Negation of regular expression pattern matching.
- **OCTET_LENGTH()** - Synonym for `LENGTH()`.
- **ORD()** - Returns the character code for the leftmost character if it is a multi-byte character.
- **POSITION()** - Synonym for `LOCATE()`.
- **QUOTE()** - Escapes a string for use in a SQL statement.
- **REGEXP()** - Pattern matching using regular expressions.
- **REPEAT()** - Repeats a string a specified number of times.
- **REPLACE()** - Replaces all occurrences of a substring with another.
- **REVERSE()** - Reverses the characters in a string.
- **RIGHT()** - Returns a specified number of characters from the right of a string.
- **RPAD()** - Right-pads a string with another string to a certain length.
- **RTRIM()** - Removes trailing spaces from a string.
- **SOUNDEX()** - Returns a Soundex string for phonetically matching words.
- **SOUNDS LIKE** - Compares two strings using Soundex.
- **SPACE()** - Returns a string containing a specified number of spaces.
- **STRCMP()** - Compares two strings and returns 0 if they are identical.
- **SUBSTR()** - Synonym for `SUBSTRING()`.
- **SUBSTRING()** - Extracts a substring from a string starting at a specific position.
- **SUBSTRING_INDEX()** - Returns a substring from a string before a count of occurrences of a delimiter.
- **TO_BASE64()** - Encodes a string into base64 format.
- **TRIM()** - Removes leading and trailing spaces or other specified characters.
- **UCASE()** - Synonym for `UPPER()`.
- **UNHEX()** - Converts hexadecimal data into a binary string.
- **UPPER()** - Converts a string to uppercase.
- **WEIGHT_STRING()** - Returns the binary weight string of a value used for sorting.

Documentation on String Functions: [https://mariadb.com/kb/en/library/string-functions/](https://mariadb.com/kb/en/library/string-functions/)

### An Example of a String Function

Used in Queries and Data Manipulation Statements

```sql
SELECT domain, domain_count
FROM (
    SELECT
        SUBSTRING(email_address, LOCATE('@', email_address) +1 ) AS domain,
        COUNT(*) AS domain_count
    FROM clients_email
    GROUP BY domain
) AS derived1
WHERE domain_count > 200
LIMIT 100;
```

### Working with the JSON Data Type

### MariaDB JSON Functions:

- **JSONPath Expressions** - Notation used to address specific parts of a JSON document.
- **JSON_ARRAY** - Creates a JSON array from a list of values.
- **JSON_ARRAYAGG** - Aggregates a result set into a single JSON array.
- **JSON_ARRAY_APPEND** - Appends values to the end of the indicated arrays within a JSON document.
- **JSON_ARRAY_INSERT** - Inserts values into a JSON array at specified positions.
- **JSON_COMPACT** - Removes all unnecessary whitespace from a JSON document to reduce its size.
- **JSON_CONTAINS** - Returns 1 if a JSON document contains a specific object at a specific path, otherwise 0.
- **JSON_CONTAINS_PATH** - Returns 1 if a JSON document contains data at the specified paths.
- **JSON_DEPTH** - Returns the maximum depth of a JSON document.
- **JSON_DETAILED** - Formats a JSON document with indentation and line breaks for human readability.
- **JSON_EQUALS** - Returns 1 if two JSON documents are equivalent, otherwise 0.
- **JSON_EXISTS** - Checks if a specific path exists within a JSON document.
- **JSON_EXTRACT** - Returns data from a JSON document at the specified paths.
- **JSON_INSERT** - Inserts values into a JSON document at specified paths without replacing existing values.
- **JSON_KEYS** - Returns the keys of a JSON object as a JSON array.
- **JSON_LENGTH** - Returns the number of elements in a JSON array or the number of members in a JSON object.
- **JSON_LOOSE** - Normalizes a JSON document and adds minimal whitespace for a "loose" compact format.
- **JSON_MERGE** - Alias for `JSON_MERGE_PRESERVE`; merges multiple JSON documents.
- **JSON_MERGE_PATCH** - Merges documents following the RFC 7396 merge patch algorithm (replaces existing keys).
- **JSON_MERGE_PRESERVE** - Merges multiple JSON documents, preserving all values and keys.
- **JSON_NORMALIZE** - Normalizes a JSON document by removing extra whitespace and sorting keys.
- **JSON_OBJECT** - Creates a JSON object from a provided list of key-value pairs.
- **JSON_OBJECTAGG** - Aggregates key-value pairs from a result set into a single JSON object.
- **JSON_OVERLAPS** - Returns 1 if two JSON documents share any common elements or key-value pairs.
- **JSON_QUERY** - Extracts an object or internal array from a JSON document.
- **JSON_QUOTE** - Quotes a string as a JSON value by adding surrounding double quotes and escaping characters.
- **JSON_REMOVE** - Removes data from a JSON document at the specified paths.
- **JSON_REPLACE** - Replaces existing values at specified paths in a JSON document.
- **JSON_SEARCH** - Returns the path to a specific string within a JSON document.
- **JSON_SET** - Inserts or updates values in a JSON document at the specified paths.
- **JSON_TABLE** - A table function that converts JSON data into a relational table format.
- **JSON_TYPE** - Returns the data type of a JSON value as a string.
- **JSON_UNQUOTE** - Unquotes a JSON value and returns the result as a regular string.
- **JSON_VALID** - Returns 1 if a string is a valid JSON document, otherwise 0.
- **JSON_VALUE** - Extracts a scalar value (string, number, or boolean) from a JSON document.

### Examples of JSON Functions

#### `JSON_VALUE` Used to pull a scalar value from JSON data

```sql
SELECT 
    name, 
    latitude, 
    longitude, 
    JSON_VALUE(attr, '$.details.foodType') AS food_type 
FROM locations 
WHERE type = 'R';
```
```shell
| name   | latitude   | longitude   | food_type |
|--------|------------|-------------|-----------|
| Shogun | 34.1561131 | -118.131943 | Japanese  |
```

#### `JSON_QUERY` Used to return entire JSON object data

```sql
SELECT 
    name, 
    latitude, 
    longitude, 
     JSON_QUERY(attr, '$.details') AS details 
FROM locations 
WHERE type = 'R';
```
```shell
| name   | latitude   | longitude   | details                                                                                   |
|--------|------------|-------------|-------------------------------------------------------------------------------------------|
| Shogun | 34.1561131 | -118.131943 | {"foodType": "Japanese", "menu": "https://www.restaurantshogun.com/menu/teppan-1-22.pdf"} |
```

#### `JSON_VALID` Used to validate JSON values

```sql
CREATE TABLE locations (
    id INT NOT NULL AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    type CHAR(1) NOT NULL,
    latitude DECIMAL(9,6) NOT NULL,
    longitude DECIMAL(9,6) NOT NULL,
    attr LONGTEXT CHARACTER SET utf8mb4
    COLLATE utf8mb4_bin DEFAULT NULL
    CHECK (JSON_VALID('attr'))
    PRIMARY KEY (id)
);
```

#### `JSON_INSERT` Used to insert fields

```sql
UPDATE locations
    SET attr = JSON_INSERT(attr,'$.nickname','The Bean') WHERE id = 8;
```

#### `JSON_ARRAY` Used to create new arrays

```sql
UPDATE locations
    SET attr = JSON_INSERT(attr, '$.foodTypes',
    JSON_ARRAY('Asian', 'Mexican'))
    WHERE id = 1;
```

#### `JSON_TABLE` Used to convert data

```sql
SELECT l.name, d.food_type, d.menu
FROM locations AS l,
     JSON_TABLE(l.attr,
       '$' COLUMNS(
         food_type VARCHAR(25) PATH '$.foodType',
         menu VARCHAR(200) PATH '$.menu')
     ) AS d
WHERE id = 2;
```
```shell
| name   | food_type | menu                                                  |
|--------|-----------|-------------------------------------------------------|
| Shogun | Japanese  | https://www.restaurantshogun.com/menu/teppan-1-22.pdf |
```

## Lesson Summary

- Identify and describe the schema objects available within MariaDB Enterprise Server
- Demonstrate how to create and use databases, tables, and columns in MariaDB Enterprise Server
- Compare and contrast the data types and built-in functions in MariaDB Enterprise Server, and explain their use cases


