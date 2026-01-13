---
title: "Lesson 2: Schema objects"
url: "/dev/schema-objects/"
date: 2025-04-10
description: >
categories: []
tags: []
weight: 1
toc: true
license_note: "For academic and non-commercial usage, licensed under CC BY-NC-SA 4.0 by MariaDB plc."
license: "CC BY-NC-SA 4.0"
license_url: "https://creativecommons.org/licenses/by-nc-sa/4.0/"
---

# Schema objects
 
## Learning objectives

- Gain an understanding of schema objects available within MariaDB
- Learn how to use and create databases and tables, including temporal data tables, and flexible and invisible columns
- Discuss data types and built-in functions, and their use cases
- Learn how to create and use views, triggers, events, and sequences

## Databases, tables and default schemas

### Case sensitivity

Depending on Operating System, File System and `lower_case_table_names`

Set before starting a project (strongly recommended)!

Usually case sensitive by default on Linux, not so on Windows and MacOS

Adopt a convention, such as always creating and referring to databases and tables using lowercase names

### Databases

**Database aka Schema**

- Highest Level Object
- Corresponds to directory within data directory
- All other objects reside within a database except user accounts, roles and plugins

```sql
CREATE DATABASE world;
```

### Tables

Stores rows of structured data organised by typed columns

Each corresponds to a metadata file (`.frm`) and data file(s), dependent on storage engine  
(i.e. InnoDB tablespaces)

Qualified by a database name  
(i.e. `database.table`)

### Columns

- Set of typed data values

- Qualified by a table name  
  *(i.e. table.column)*

- Stored together in each row (MariaDB Server) within pages, within tablespaces or data files  
  Virtual generated columns are not stored at all

- Index

### Column attributes

Columns have strict type definitions

Can specify DEFAULT value for column

Only Primary Key can be automatically incremented

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
  - Reduces storage in some Engines
  - Can also reduce execution time because there are more CPU cycles used to first check for NULL

### Indexes

- Exact copy of selected columns followed by primary key (e.g., name,address,ID)
  - For long columns the index might not be an "exact copy", but instead it might contain a prefix of the column
- Fast lookup of data within a table, without having to scan all columns for each row
- For InnoDB, primary key is silently appended to end of secondary indexes, unless specified elsewhere within index
- Attribute of a table

### Constraints

- Types: Primary Key, Foreign Key, Unique and Check
- Support and implementation differs per storage engine
- Allows data within column(s) to be limited or constrained to a set of values
- Usually defined with corresponding index
- Attribute of a table
- Has potential for contention at scale

### Views

- Virtual table defined by a SQL `SELECT` query
- Evaluated at each access
  - No materialized views
- Treated as a table for many purposes, shares namespace with tables
- Updateable in certain cases (`WITH CHECK OPTION`)
- Qualified by database (`database.view`)

### Stored routines

- Types: Functions, Triggers, Events, and Stored Procedures
- Has input and output parameters
- Reusable SQL Code
  - SQL/PSM (default)
  - PL/SQL (`sql_mode=ORACLE`)
- Can allow cursor loops
- Runtime script, not compiled or binary
- Lack of good debugging, can be hard to profile
- Can be bad for Statement based binary logging
- Stored routine uses the privileges of the user that defined it (user can be changed with the DEFINER clause)
- If SQL SECURITY INVOKER is set then the privileges of the user executing the stored routine are used
- Qualified by database (`database.my_stored_procedure`)

### Default schemas

#### information_schema
- The encyclopedia of your database
- Contains information on databases, tables, columns, procedures, indexes, statistics, partitions and views
- Plugins can install additional `information_schema` tables

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
- Will be covered in Tuning & optimization Module

### Creating your first table

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
- Column Definitions
- Define Primary Key, Secondary Indexes and Constraints
- Define Engine, Partitions, Character Set and other attributes

### Altering tables

`ALTER TABLE` is used to change a table's schema

**ADD COLUMN** to Add a Column

**DROP COLUMN** to Drop a Column — Deletes Data

**CHANGE COLUMN** and **MODIFY COLUMN** to Alter a Column

```sql
ALTER TABLE table1
ADD COLUMN col5 CHAR(8),
DROP COLUMN col3,
CHANGE COLUMN col4 col6 DATE,
MODIFY COLUMN col8 VARCHAR(10);
```

Basic Syntax Example for `ALTER TABLE` Statement

### Temporal tables

| Type                    | Tracks                           | Sample Use Cases                          |
|-------------------------|----------------------------------|-------------------------------------------|
| System-Versioned        | Change history                   | Audit, forensics, IoT temperature tracking |
| Application-Time Period | Time-limited values              | Sales offers, subscriptions               |
| Bitemporal              | Time-limited values with history | Schedules, decision support models        |

### Temporal tables

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
    PERIOD FOR
    valid_period(date_start, date_end)
);
```

#### Bitemporal Example

```sql
CREATE TABLE coupons_new (
    id INT UNSIGNED,
    name VARCHAR(255),
    date_start DATE,
    date_end DATE,
    PERIOD FOR
    valid_period(date_start, date_end)
) WITH SYSTEM VERSIONING;
```

## Data types and built-in functions

### Data types

Types: Binary, Numeric, String, Temporal, and User Defined

Use the most suitable data type to store all possible, required values  
Will truncate silently and rounds (`sql_mode`)

```shell
MariaDB [(none)]> help INT;
Name: 'INT'
Description: INT[(M)] [UNSIGNED] [ZEROFILL]
```

A normal-size integer.
The signed range is -2147483648 to 2147483647.
The unsigned range is 0 to 4294967295.

URL: https://mariadb.com/kb/en/mariadb/data-types-numeric-data-types/


### Numeric data types

- `FLOAT` and `DOUBLE` are approximate types
  - Uses 4 and 8 bytes IEEE storage format
- `DECIMAL (m, d)` maximum total number of digits, number of digits after decimal point
  - An Exact Value type, up to 65 digits precision, 4 bytes storage for each multiple of nine digits
- `NUMERIC` is a synonym for `DECIMAL`
- `REAL` is a synonym for `DOUBLE`
  - Unless in `REAL_AS_FLOAT` SQL mode

### Auto increment

- Use `LAST_INSERT_ID()` to get value generated for client connection
- `SERIAL` is a synonym for `BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE`
- In Aria the counter can be set back manually or if counter value wraps
- InnoDB
  - Prepares `AUTO_INCREMENT` counters when the MariaDB Server starts
  - Single mutex on a table, behavior changed with `innodb_autoinc_lock_mode`
 
#### 0 | default, Traditional
- Holds table-level lock for all `INSERTs` until end of statement

#### 1 Consecutive
- Holds table-level lock for all bulk `INSERTs` (such as `LOAD DATA` or `INSERT ... SELECT`) until end of statement
- For simple `INSERTs`, no table-level lock held

#### 2 Interleaved

- No table-level locks are held ever
- Fastest and most scalable
- Not safe for statement-based replication
- Generated IDs are not always consecutive

### Numeric data types

- Use `UNSIGNED` when appropriate
- `INT(n)` specifies display precision, not storage precision
- Size and precision is storage engine dependent
- Define handling of Out-of-Range Values with `sql_mode`
  - Default Mode: Values are Truncated Silently
  - Strict Mode: Errors are Generated
- `BIGINT` can enumerate more than all the ants on Earth and shouldn’t be your default choice
- `TINYINT(1)` is used for `BOOLEAN` values and is aliased by the `BOOLEAN` type

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

### Binary data types

- `BINARY`, `VARBINARY` and `BLOBs` can contain data with bytes from the whole range from 0 - 255
- Uses a special character set and collation called "binary"
- Blobs are often used to store files in a database
  - Files on disk are often faster
  - But no referential integrity is guaranteed
- Blobs are included in transactions, replication, and backups
- Blobs inflate `mysqld` memory usage
- Modern InnoDB has some improvements in storage and lookup of blobs

### Temporal data types

- `DATE` — from 1000-01-01 to 9999-12-31
  - YYYY-MM-DD
- `TIME` [(<microsecond precision>)]
  - from -838:59:59 to 838:59:59
- `DATETIME` [(<microsecond precision>)]
  - Same ranges as `DATE` and `TIME` above
  - YYYY-MM-DD HH:mm:ss
- `TIMESTAMP` — Unix timestamp, in seconds from 1970-01-01
  - Many Apps Store `UNIX_TIMESTAMP()`
    values in unsigned integer field
- `YEAR` — Accepts YYYY

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

### Special data types

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
      Language SET('English','French','Mandarin') 
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

### Built-in functions

Types: String, Date and Time, Aggregate, Numeric, Control Flow

Secondary functions such as Bit Functions and Operators, Encryption, Hashing and Compression, and Information Functions

Special Functions such as Dynamic Columns, Geographic, JSON, Spider and Window Functions

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

[Documentation on Date and Time Functions](https://mariadb.com/docs/server/reference/sql-functions/date-time-functions)

### Examples of date & time functions

#### Used in Queries and Data Manipulation Statements

```sql
SELECT NOW() + INTERVAL 1 DAY
         - INTERVAL 1 HOUR
       AS 'Day & Hour Earlier';
```

```shell
+---------------------+
| Day & Hour Earlier  |
+---------------------+
| 2020-06-02 08:32:44 |
+---------------------+
```

#### Used in WHERE Clauses

```sql
UPDATE table1
SET col3 = 'today', col4 = NOW()
WHERE col5 = CURDATE();
```

#### Used in Bulk Load

```sql
load data local infile '/tmp/test.csv' into
table test fields terminated by ','
ignore 1 lines (id,@dt1)
set dts=str_to_date(@dt1,'%d/%m/%Y');
```

### Manipulating strings

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

### An example of a string function

Used in queries and data manipulation statements

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

### Aggregate functions

Aggregate functions perform a calculation on a set of values and return a single result. Typically used in conjunction with the `GROUP BY` clause, these functions allow you to summarize data across multiple rows.

### Common MariaDB Aggregate Functions:

- **AVG()** - Returns the average value of the argument.
- **COUNT()** - Returns a count of the number of non-NULL values in the rows retrieved by a SELECT statement.
- **SUM()** - Returns the sum of the argument.
- **GROUP_CONCAT()** - Returns a string result with the concatenated non-NULL values from a group.
- **MAX()** - Returns the maximum value of the argument.
- **MIN()** - Returns the minimum value of the argument.
- **STD()** - Returns the population standard deviation of the argument.
- **STDDEV()** - An Oracle-compatible synonym for STD(), returning the population standard deviation.
- **VARIANCE()** - Returns the population standard variance of the argument.
### Aggregation function examples

**Without GROUP BY:**
Returns a single summary row for all selected rows.

```sql
SELECT COUNT(*), AVG(Population), MAX(Population) 
FROM City;
```

**With GROUP BY:**
Groups the outcome by one or more columns to provide summaries for each group.

```sql
SELECT CountryCode, SUM(Population) AS TotalPopulation
FROM City
GROUP BY CountryCode;
```


`DISTINCT` Removes Duplicate Values before Aggregation (e.g., with `COUNT()` and `GROUP_CONCAT()`)

```sql
SELECT COUNT(*)
FROM City;
```
```shell
+----------+
| COUNT(*) |
+----------+
| 4079     |
+----------+
```
```sql
SELECT COUNT(DISTINCT CountryCode)
FROM City;
```
```shell
+-----------------------------+
| COUNT(DISTINCT CountryCode) |
+-----------------------------+
| 232                         |
+-----------------------------+
```

### Aggregate multiplication

**Using logarithm logic:**

A chart shows the rate of return for a publicly traded company over 10 years

With an initial investment of $1,000 in 2009, calculate the value of the investment by 2019

- Possible solution:
  ```sql
  SELECT AVERAGE(ROR) FROM investment;
  ```
- Problem: This would result in an average return of approximately 9.9% over the 10 year span, but it does not take into account compound interest.

### Aggregate multiplication

Calculating compound interest: Find each year's ROR:

```shell
SELECT YEAR, 1+ROR AS multiplier, LOG(1+ROR) 
FROM investment;
+------+-----------+---------------------+
| YEAR | multiplier | LOG(1+ROR/100)     |
+------+-----------+---------------------+
| 2010 |     1.44  |     0.36464311193222|
| 2011 |     0.75  |    -0.28768207245178|
| 2012 |     1.08  |     0.07961013948044|
.. 
| 2017 |     1.11  |     0.10436001478726|
| 2018 |     0.92  |    -0.083381606995421|
| 2019 |     0.79  |    -0.23572232522169 |
+------+-----------+---------------------+
10 rows in set (#.## sec)
```

### Aggregate multiplication

Calculating compound interest:

1. Add all the logarithmic results together

   ```sql
   SELECT SUM(LOG(1+ROR))
   FROM investment;
   ```
   ```shell
   +--------------------+
   | SUM(LOG(1+ROR))    |
   +--------------------+
   | 0.68010598814117   |
   +--------------------+
   ```

2. Determine the return percentage with compound interest

   ```sql
   SELECT EXP(SUM(LOG(1+ROR)))
   FROM investment;
   ```
   ```shell
   +------------------------+
   | EXP(SUM(LOG(1+ROR)))   |
   +------------------------+
   |     1.9740869509493    |
   +------------------------+
   ```

### Aggregate multiplication

Calculating compound interest:

- Calculate the result

- Over ten years, even including the bad years, the initial investment of $1,000 nearly doubled

- Creative SQL can be used to solve perceived limitations in the language.

```sql
SELECT EXP(SUM(LOG(1+ROR)))*1000 FROM investment;
```
```shell
+-------------------------+
| EXP(SUM(LOG(1+ROR)))*1000 |
+-------------------------+
| 1974.0869509493          |
+-------------------------+
```

### Running total query

In the course of almost every analytical presentation, someone will request that a running total be presented!

The following SQL example uses a self-join on the investment table to create the running total

```sql
SELECT a.YEAR, a.ROR,
    EXP(SUM(LOG(1+b.ROR)))*1000 AS balance
FROM investment a
JOIN investment b ON (a.YEAR >= b.YEAR)
GROUP BY a.YEAR, a.ROR;
```

### Running total query

Notice how this statement works

```sql
SELECT a.YEAR, a.ROR,
EXP((LOG(1+b.ROR)))*1000 AS balance
FROM investment a
JOIN investment b ON (a.YEAR >= b.YEAR)
ORDER BY a.YEAR, a.ROR;
```

```shell
+------+----------+------------------+
| YEAR | ROR      | balance          |
+------+----------+------------------+
| 2000 |  0.0     | 1439.99999761581 |
| 2011 | -0.25    | 1439.99999761581 |
| 2011 | -0.25    | 1439.99999761581 |    750 |
| 2012 |  0.08    | 1439.99999761581 |
...
| 2019 | -0.21    | 1439.99999761581 |
| 2019 | -0.21    | 1439.9999821186 |    750 |
| 2019 | -0.21    | 1079.9999821186 |
+------+----------+------------------+
```

### Control flow functions

`IF(expr1,expr2,expr3)`
- If `expr1` is TRUE (`expr1 <> 0` and `expr1 <> NULL`) then `IF()` returns `expr2`; otherwise it returns `expr3`

`IFNULL(expr1,exprNULL)` returns `expr1`, or `exprNULL` if `expr1` is `NULL`

Do not mix up with `NULLIF(expr1,expr2)` which returns `expr1` if `expr1<>expr2` otherwise `NULL`

### Some numeric functions

- `CEIL(X)`
  - Returns the smallest integer value not less than `X`
  - Synonym for `CEILING()`
- `FLOOR(X)`
  - Returns the largest integer value not greater than `X`
- `ROUND(X, D)`
  - Rounds or round up to `D` decimal places
- `ABS(X)`
  - Returns the non-negative number of `X`

## Views, triggers and events

### Views

- Adapt and standardize table schema
- Simplify, split, or factor complex reporting queries
- Restrict visible table data to specific users and applications

### An example of a view

- A SQL Statement Represented as a Table
- Views are Not Materialised
- The `SELECT` Query always Re-Executes

```sql
CREATE VIEW emp_names AS
    SELECT emp_id, name_first, name_last
    FROM employees;

ALTER VIEW emp_names AS
    SELECT emp_id, name_first, name_middle, name_last,
    FROM employees;

SHOW FULL TABLES WHERE Table_type = 'VIEW';

SHOW CREATE VIEW emp_names \G

DROP VIEW emp_names;
```

## Lesson summary

- Gain an understanding of schema objects available within MariaDB
- Learn how to use and create databases and tables, including temporal data tables, and flexible and invisible columns
- Discuss data types and built-in functions, and their use cases
- Learn how to create and use views, triggers, events, and sequences

## Lab exercises

- 2-1 Creating a Database
- 2-2 Creating a Table
- 2-3 Creating Virtual Columns
- 2-4 Setting the Database Default Character Set
- 2-5 Creating and Comparing Tables for Multiple Storage Engines

