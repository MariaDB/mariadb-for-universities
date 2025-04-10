# Data manipulation and extraction

## Learning objectives

- Know how to select, insert, load, change and delete data
- Explain how to execute outer and inner joins, and join a table to itself
- Know how to use correct `SQL` syntax when creating subqueries within a statement, and demonstrate how to convert subqueries into joins
- Describe and use window functions and common table expressions (`CTEs`)

# Selecting data

## Select statement

**Retrieves data from the table for the client**

**Can calculate and use functions**

Be careful of the overuse of functions in OLTP use cases where it may be better to add a column such as a virtual column that contains calculations already rendered

```sql
SELECT CONCAT(name_first, SPACE(1), name_last) AS 'Name',

(DAYOFYEAR(birthdate) - DAYOFYEAR(CURDATE())) / 12 AS 'Months to Birthday'

FROM clients

LIMIT 1;
```

An Example of Retrieving Data, Calculating, and Using Functions with a `SELECT` Statement

## Where clause

Used with `SELECT`, `UPDATE`, and `DELETE` Statements

Columns with Operators can be Specified to Filter Results

```sql
SELECT Name, Population
FROM City
WHERE Population > 1000000;

+-------------+------------+
| Name        | Population |
+-------------+------------+
| Kabul       | 1780000    |
| Alger       | 2168000    |
| Luanda      | 2022000    |
| Buenos Aires| 2982146    |
| La Matanza  | 1266461    |
| ...         |            |
```

Basic Example for `SELECT` Statement using `WHERE` Clause

## Operators

The `WHERE` Clause Allows For Several Operators

| Arithmetic | Comparison |
|------------|------------|
| -          | AND, &&    | IS, =         | LIKE       |
| +          | OR, \|\|   | <             | IS         |
| *          | XOR        | <=            | IS NOT     |
| /          | >=         | IS NULL, ISNULL| IS NOT NULL|
| %          | >          | <> , <>, !=     | IN         |
|            |            | INTERVAL      |
|            |            | BETWEEN ... AND|
|            |            | NOT BETWEEN ... AND|

## Limit clause

Used to limit the offset and number of rows returned

Values must be positive integer constants

```sql
SELECT Name, District
FROM City
WHERE CountryCode = 'FIN'
LIMIT 2, 3;
```

```sql
+-------------------+-----------+
| Name              | District  |
+-------------------+-----------+
| Helsinki          | Newmaa    |
| Espoo             | Newmaa    |
| Tampere           | Pirkanmaa |
+-------------------+-----------+
```

## Order by and limit clause

Data can be Ordered and Broken into Blocks

`SELECT cols`
`FROM table`
`WHERE clause`
`ORDER BY col`
`LIMIT offset, count;`

Basic Syntax Example for `SELECT` Statement with `ORDER BY` Clause

`SELECT Name, Population`
`FROM City`
`WHERE Population > 1000000`
`ORDER BY Population`
`LIMIT 0, 3;`

| Name    | Population |
|---------|------------|
| Zapopan | 1,012,563  |
| Napoli  | 1,002,079  |
| Perm    | 1,070,162  |

## Ascending and descending order

Add the `ASC` or `DESC` Options to Order Results in Ascending or Descending Order

Multiple Columns Allowed

No descending indexes however B+ Tree indexes can be used to optimize sort or range based queries

```sql
SELECT Name, District FROM City
WHERE CountryCode = 'FIN'
ORDER BY District DESC, Name ASC;
```

| Name           | District         |
|----------------|------------------|
| Turku [Åbo]    | Varsinais-Suomi  |
| Lahti          | Päijät-Häme      |
| Oulu           | Pohjois-Pohjanmaa|
| Tampere        | Pirkanmaa        |
| Espoo          | Newmaa           |
| Helsinki [Helsingfors]| Newmaa     |
| Vantaa         | Newmaa           |


## Group by clause

Useful with Aggregate Functions to Create Sub-Groups

```sql
SELECT Continent, SUM(Population)
FROM Country
GROUP BY Continent;
```

```sql
+---------------+------------------+
| Continent     | SUM(Population)  |
+---------------+------------------+
| Asia          |    3750025700    |
| Europe        |     730074600    |
| North America |     482993000    |
| Africa        |     784475000    |
| Oceania       |      30401150    |
| Antarctica    |              0    |
| South America |     345780000    |
+---------------+------------------+
```

## Select into outfile statement

Exports Results to a File on Server  
File is Saved to datadir by Default  
Fails if Another File with Same Name  
`FILE` Privilege Required  
One Record per Line  
Fields Separated by Tabs (`Ascii #9\t`) by Default  

```sql
SELECT INTO OUTFILE '/path/City.txt'
FROM City;
```

Basic Example of `SELECT INTO OUTFILE` Syntax

```sql
SELECT INTO OUTFILE '/path/City.csv'
FROM City
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
ESCAPED BY '\\'
LINES TERMINATED BY '\r\n'
STARTING BY 'Record: ';
```

Examples of More Precise Format Options

# Inserting and loading data

## Single versus multi-row insert

Single Row `INSERT` Repeats Full Write Process for Each

- Increases Traffic
- Parsing is Repeated (Unlike Query Cache for `SELECT`)
- Index Rebuilt for Each `INSERT`

Multi-Row `INSERT` Minimizes Overhead

- Rows are Inserted in Batches
- Requires only One Index Rebuild

```sql
INSERT INTO table1 VALUES ('test',1,2);

INSERT INTO table1 VALUES ('test',1,2);

INSERT INTO table1 VALUES ('test',1,2), ('test',3,4);
```

Write Process

- CONNECT
- SEND
- PARSE
- INSERT
- INDEX
- DISCONNECT

## Multi-row insert caveats

**Multi-Row `INSERT` Statements are Transactions**
- All Rows must be Written to Commit Transaction with a Transactional Engines
- Prior Rows Inserted if One fails with Non-Transactional Engine

**Size is Set by `max_allowed_packet` Variable**

**Faster Loading of Large Data Sets**
- Used by Default by `mysqldump`

## Insert...select statement

- Useful in copying data between tables
  - Fields of `SELECT` must match `INSERT` column list
- No separate parse step required to insert data

```sql
CREATE TABLE employees_personal LIKE employees;

INSERT INTO employees_personal
(emp_id, gender, birth_date)
SELECT emp_id, gender, birth_date
FROM employees;
```

## Duplicate keys statement

`DUPLICATE KEY` does a select then decides on insert versus update

`REPLACE` does a delete then an insert

Existing Rows are Updated not Duplicated when using `ON DUPLICATE KEY UPDATE`

Duplicates are Rows with same `PRIMARY` or `UNIQUE` Key Values

Only One is Updated even if Many Exist

```sql
INSERT INTO parts (part_code, part)
VALUES ('bt-45','45mm Bolt')
ON DUPLICATE KEY 
UPDATE part_code = part_code + '-1';
```

```sql
UPDATE parts 
SET part_code = part_code + '-1'
WHERE part_code = 'bt-45';
```

```sql
INSERT INTO parts (part_code, part)
VALUES ('bt-45','45mm Bolt');
```

This DUPLICATE KEYS statement is the equivalent of these statements ====>


## Load data infile statement

Imports Data Directly from File to Server

Put File in Data Directory or give

Full Path

`FILE` Privilege Required

Use `LOAD DATA LOCAL INFILE` for File on Client Instead of Server

```sql
LOAD DATA INFILE '/tmp/City.csv'
INTO TABLE City
FIELDS
    TERMINATED BY ','
    ENCLOSED BY '"'
    ESCAPED BY '\\'
LINES TERMINATED BY '\r\n'
STARTING BY 'Record: ';
```

## Importing SQL files

Import SQL Text Files using `mysql` Client from Command-Line

Fast Option and No Screen Output

Aborts on Errors by Default

```
# mysql -u devuser -p world < /path/to/world.sql
```

# Changing and deleting data

## Update statement

Use `SET` Clause to Change Column Values

Several Optional Clauses (i.e., `JOIN`, `WHERE`, `ORDER BY`, `LIMIT`)

All Rows Changed Without `WHERE` or `LIMIT` Clause

`ORDER BY` and `LIMIT` Are Not Compatible with `JOIN`

```sql
UPDATE table1
JOIN table2 USING(id)
SET table1.col1 = 'expression'
WHERE table2.field4 = 'whatever';
```

Basic Syntax Example for `UPDATE` Statement

```sql
UPDATE City
SET Population = Population + 1
WHERE CountryCode = 'FIN';

Query OK, 7 rows affected (0.00 sec)
Rows matched: 7 Changed: 7 Warnings: 0
```

## Replace statement

**`REPLACE` can be used instead of `INSERT` (MariaDB Extension to SQL)**

Deletes Duplicates in Table

Inserts New Rows

Duplicates are Rows with Same Value for `PRIMARY` or `UNIQUE` Key

```sql
REPLACE INTO table (columns) VALUES (values);
```

Basic Syntax Example for `REPLACE` Statement

## Delete statement

Specify Table or Tables from which to Delete Rows

- `JOIN` Causes Deletions in Joined Tables

Several Optional Clauses (i.e., `JOIN`, `WHERE`, `ORDER BY`, `LIMIT`)

All Rows Changed Without `WHERE` or `LIMIT` Clause

`ORDER BY` and `LIMIT` Are Not Compatible with `JOIN`

```sql
DELETE FROM table1
JOIN table2 USING(id)
WHERE condition;
```

Basic Syntax Example for `DELETE` Statement

```sql
DELETE FROM City
WHERE CountryCode = 'FIN';

Query OK, 7 rows affected (0.02 sec)
Rows matched: 7  Changed: 7  Warnings: 0
```

## Truncate statement

Empties a table completely

`TRUNCATE tablename [WAIT n|NOWAIT];`

The `TRUNCATE <tablename>` statement will fail for an InnoDB table if any `FOREIGN KEY` constraints from other tables reference the table, returning the following error

`ERROR 1701 (42000): Cannot truncate a table referenced in a foreign key constraint`

Faster than a `DELETE` statement because it drops and re-creates the table

Will return a row count of 0 so if you need to know how many rows were removed, use the `DELETE` statement

Drops all historical records from a system-versioned table

# Joins

## Joins overview

Joins combine records from two or more tables, using values common within each one

Rows from Table A are *joined* with rows from Table B

The result *set* has rows containing fields from both tables

Result may be returned to the client, placed into a new table, or used as an *implicit* table for further operations

`INNER` `OUTER` `NATURAL`  
`CROSS` `STRAIGHT_JOIN` `RIGHT`  
`LEFT` `USING` `ON`

## Implicit join syntax

Join and Other `WHERE` Conditions Mingled

Without Constraints, the Result is a Full Cartesian Product

`SELECT` cols  
`FROM` table, table2  
`WHERE` join-condition  
`AND` other-condition;

Basic Syntax Example for Joining Tables

`SELECT` City.Name, Country.Name, City.Population  
`FROM` Country, City  
`WHERE` CountryCode = Code  
`AND` City.Population < 100000;

Practical Example for Joining Tables within `WHERE` Clause

## Explicit join syntax

With a `JOIN` Clause the Join Conditions are Separate

Other `WHERE` Clause Conditions are Not Mixed with Join Points

```sql
SELECT cols
FROM table JOIN table2
ON join-condition
WHERE other-condition;
```

Basic Syntax Example for `JOIN` Clause

```sql
SELECT City.Name, Country.Name, City.Population
FROM Country
JOIN City ON CountryCode = Code
WHERE City.Population < 100000;
```

Practical Example for Joining Tables with `JOIN` Clause

## Inner joins

- `JOIN` defaults to an `INNER JOIN`
  - `INNER` keyword is optional
  - In MariaDB a `CROSS JOIN` is the same as an `INNER JOIN`
  
- Without filtering, result is a cartesian product
  - Every matched row in table1 is joined to every matched row in table2
  - `ON` clause is optional; without it, every row in both table matches
  - Can quickly become a large result set!
  
- Only returns rows when there is a match in both tables

```sql
SELECT cols FROM
table INNER JOIN
table2
ON join-condition
WHERE
other-conditions;
```

## Outer joins

`LEFT JOIN` really means `LEFT OUTER JOIN`

- `OUTER` keyword is optional
- `RIGHT JOIN` is also available

An `OUTER JOIN` does not require matches in both tables

- For `LEFT JOIN` every matching row from the left table is returned
- If no corresponding match in the right table, fields are `NULL`
- Vice versa for `RIGHT JOIN`

MariaDB does not support full outer join

- It can be simulated using a `UNION` between `LEFT` and `RIGHT` joins

```sql
SELECT cols FROM
table LEFT JOIN
table2
ON join-condition
WHERE
other-conditions;
```

## Outer join example

`SELECT cols FROM table LEFT JOIN table2 ON join-condition WHERE other-conditions;`

`SELECT Country.Name as Country, City.Name as Capital FROM Country LEFT JOIN City ON Capital = City.Id WHERE Country.Name LIKE 'B%';`

```sql
+-------------------------------+--------------+
| Country                       | Capital      |
+-------------------------------+--------------+
| Bulgaria                      | Sofija       |
| Burkina Faso                  | Ouagadougou  |
| Burundi                       | Bujumbura    |
| Belarus                       | Minsk        |
| Bouvet Island                 | NULL         | <- no capital
| British Indian Ocean Territory| NULL         | <- no capital
+-------------------------------+--------------+
```

## Natural joins

- Compare columns common to table a and b
- Column names must match exactly
- Result set holds one column for each matching pair
- Generally considered dangerous!
- The join conditions are not obvious or easy to debug
- What happens when the schema changes?

  - If fields with the same names are added to both tables?
  - If a field is dropped from one table, but not the other?
  - There will be no explicit error, but the result set will change!

## General tips

Cartesian products of table subsets, with variations

**The optimizer does a lot for you such as:**

- Orders tables for minimum cost (fewest rows examined)
- Converts `OUTER` to `INNER` when possible

**Other performance considerations to consider:**

- Ensure columns in `ON` and `USING` clauses are indexed
- Try to keep `GROUP BY` and `ORDER BY` columns in one table, so that an index may be used (if available)
- Consider *join decomposition* (very application specific)

## General tips

### Watch for ambiguous column references

Example: If two tables contain a `Name` field

Use full notation such as: `tableA.Name = tableB.Name`

### Use table aliases to shorten and simplify queries

An alias is just an alternative name for a table, within this query

The `AS` keyword is optional

Makes for easier reading and easier typing!

Less work required when tables are renamed

Ensures column names are never ambiguous

```sql
SELECT ... FROM TableA AS a, TableB AS b
WHERE a.Name = b.Name;
```

## General tips

Tables in joins are not necessarily processed in the same order they are listed in the query  
MariaDB's Optimizer will move them around for efficiency  
This does not affect the result set from a relational point of view  
Use `STRAIGHT_JOIN` to control table join order, but generally only for debugging! The optimizer is really very good at its job

Beware that cartesian products can grow very large  
Particularly when more than two tables are involved

Use inner joins over outer joins, where possible  
Unless outer is explicitly required by the application  
As a rule the working and result sets are smaller for inner joins  
Unexpected `NULLs` make applications fall over (ie, when data changes)

Avoid using comma joins in queries to attain readability, flexibility and portability

# Subqueries

## Subqueries

Queries within Other Queries Entered within Parentheses

```sql
SELECT Language FROM CountryLanguage
WHERE CountryCode = (SELECT Code FROM Country WHERE Name = 'Finland');
```

## Scalar subquery

- Returns a Single Value
  - Result Set must be One Row with One Column

- NULL is Returned for an Empty Result Set

```sql
SELECT Country.Name, 100 * CountryPopulation / 
(SELECT SUM(Population) FROM Country) AS pct_world_pop 
FROM Country;

+---------------------+--------------+
| Name                | pct_world_pop|
+---------------------+--------------+
| Afghanistan         |      0.3738  |
| Netherlands         |      0.2610  |
| Netherlands Antilles|      0.0036  |
...
```

## Row subqueries

Row subqueries return a single row

The subquery result set must be _one row with 2+ cols_

Use with equality operators
=, <>, !=, <=>  

Use with comparison operators
<, >, >=, <=  

Column order is important here!

```sql
SELECT ('London', 'GBR') = 
(SELECT Name, CountryCode 
FROM City WHERE ID = 456) AS is_london;

+-----------+
| is_london |
+-----------+
|         0 |
+-----------+
```

## Table subquery result set

Table subqueries return an _implicit_ table

The subquery result may contain zero or more rows

Can be used in a `FROM` clause with a table alias

Use with `WHERE` and `IN`, `EXISTS`, `ANY`, `ALL` or `SOME`

```sql
SELECT * FROM
(SELECT Code, Name
 FROM Country
 WHERE IndepYear IS NOT NULL)
 AS independent_countries;

+------+---------------+
| Code | Name          |
+------+---------------+
| AFG  | Afghanistan   |
| NLD  | Netherlands   |
...
```

## Rewriting to joins

Why rewrite a perfectly good subquery into a join?

In some cases, joins still outperform subqueries  
The MariaDB Optimizer is more mature for joins...though it is getting better all the time!  

Non-correlated subqueries generally perform well  
They do not reference the outer query and are executed just once  
Can be executed alone  

Correlated subqueries can lead to performance traps  
They depend on the outer query and may be executed repeatedly  
Cannot be executed alone  

## Rewriting to joins

**IN** subqueries can be an **INNER JOIN**  
Same concept: An inner join only returns rows that match  
May require use of `DISTINCT`

**NOT IN** subqueries can be an **OUTER JOIN**  
`LEFT JOIN` and `WHERE ... IS NULL`

Sometimes, the equivalent join is much more complex  
For example using subqueries in the `FROM` clause for easy aggregation  
Or using subqueries used for accessing hierarchical data

# Lesson summary

- Know how to select, insert, load, change and delete data
- Explain how to execute outer and inner joins, and join a table to itself
- Know how to use correct SQL syntax when creating subqueries within a statement, and demonstrate how to convert subqueries into joins
- Describe and use window functions and common table expressions (CTEs)

# Lab exercises

- 5-1 Exporting and Importing Data Using mariadb-dump
- 5-2 Exporting and Importing Data Using MariaDB Backup
- 5-3 Exporting and Importing Data Using Database Statements
- 5-4 Using Different Types of Joins in Queries
- 5-5 Using and Optimizing Different Types of Subqueries
- 5-6 Using Common Table Expressions
- 5-7 Using Window Functions
- 5-8 Rolling Back a Transaction
- 5-9 Observing a Deadlock