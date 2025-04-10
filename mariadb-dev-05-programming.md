# Programming

## Learning objectives

- Differentiate between stored procedures and stored functions using the native MariaDB SQL/PSM language
- Use JSON document stores
- Understand how to access MariaDB via APIs using both native and non-native MariaDB connectors
- Set SQL modes to affect error output and use the SHOW WARNINGS and SHOW ERRORS statements to interpret error messages

# JSON

## Relational + JSON

**How to use hybrid data models**

**Relational**  
data integrity, transactions and reliability

**JSON**  
flexibility, simplicity and ubiquity
- JSON data type since 10.2
- Many JSON functions available besides schema support
- Indexing for faster retrieval is possible
- Stored as `LONGTEXT` in `mysqldump`

**A data model can be comprised of structured and semi-structured data**
- Structured data is externally described
- Semi-structured data is self described

## Relational + JSON

| Name   | Format | Price | Video                                           |
|--------|--------|-------|-------------------------------------------------|
| Aliens | Blu-ray| 13.99 | `{"resolution": "1080p", "aspectRatio": "1.85:1"}` |

**Structured**

Structure described by the schema

**Semi-structured**

Structure described by the data

## Define

### Creating and Querying JSON Documents

```sql
CREATE TABLE IF NOT EXISTS products (
    id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
    type VARCHAR(1) NOT NULL,
    name VARCHAR(40) NOT NULL,
    format VARCHAR(20) NOT NULL,
    price FLOAT(5, 2) NOT NULL,
    attr JSON NOT NULL);
```

## Create

### Creating and querying JSON documents

```sql
INSERT INTO products (type, name, format, price, attr) VALUES
  ('M', 'Aliens', 'Blu-ray', 13.99,'{"video": {"resolution": "1080p", "aspectRatio": "1.85:1"}, "cuts":
    [{"name": "Theatrical", "runtime": 138}, {"name": "Special Edition", "runtime": 155}], "audio":
    ["DTS HD", "Dolby Surround"]}');

INSERT INTO products (type, name, format, price, attr) VALUES
  ('B', 'Foundation', 'Paperback', 7.99, '{"author": "Isaac Asimov", "page_count": 296}');
```

## Read field

Creating and Querying JSON Documents

```sql
SELECT name, format, price,
    JSON_VALUE(attr, '$.video.aspectRatio') AS aspect_ratio
FROM products
WHERE type = 'M';
```

| Name   | Format | Price | Aspect_ratio |
|--------|--------|-------|--------------|
| Aliens | Blu-ray| 13.99 | 1.85:1       |

## Null field

Creating and Querying JSON Documents

```sql
SELECT type, name, format, price,
    JSON_VALUE(attr,$.video.aspectRatio') AS aspect_ratio
FROM products;
```

| Type | Name       | Format    | Price | Aspect_ratio |
|------|------------|-----------|-------|--------------|
| M    | Aliens     | Blue-ray  | 13.99 | 1.85:1       |
| B    | Foundation | Paperback | 7.99  | NULL         |

## Contains field

Creating and Querying JSON Documents

```sql
SELECT name, format, price,
    JSON_VALUE(attr, '$.video.aspectRatio') AS aspect_ratio
FROM products
WHERE type = 'M' AND
    JSON_CONTAINS_PATH(attr, 'one', '$.video.resolution') = 1;
```

| Name  | Format | Price | Aspect_ratio |
|-------|--------|-------|--------------|
| Aliens | Blu-ray | 13.99 | 1.85:1 |

## Contains value

Creating and Querying JSON Documents

```sql
SELECT id, name, format, price
FROM products
WHERE type = 'M' AND
  JSON_CONTAINS(attr, '\"DTS HD\"', '$.audio') = 1;
```

| Name   | Format | Price | Aspect_ratio |
|--------|--------|-------|--------------|
| Aliens | Blu-ray| 13.99 | 1.85:1       |

## Read array

Creating and Querying JSON Documents

```sql
SELECT name, format, price,
   JSON_QUERY(attr, '$.audio') AS audio
FROM products
WHERE type = 'M';
```

| Name   | Format | Price | Audio                          |
|--------|--------|-------|--------------------------------|
| Aliens | Blu-ray| 13.99 | [ "DTS HD", "Dolby Surround" ] |

## Read array element

Creating and Querying JSON Documents

```sql
SELECT name, format, price,
  JSON_VALUE(attr, '$.audio[0]') AS default_audio
FROM products
WHERE type = 'M';
```

| Name  | Format | Price | Audio  |
|-------|--------|-------|------- |
| Aliens | Blu-ray | 13.99 | DTS HD |

## Read object

### Creating and querying JSON documents

```sql
SELECT name, format, price,
    JSON_QUERY(attr, '$.video') AS video
FROM products
WHERE type = 'M';
```

| Name  | Format | Price | Video                                    |
|-------|--------|-------|------------------------------------------|
| Aliens| Blu-ray| 13.99 | { "resolution": "1080p", "aspectRatio": "1.85:1" } |

## Read objects

Creating and Querying JSON Documents

```sql
SELECT name,
    JSON_QUERY(attr, '$.audio') AS audio,
    JSON_QUERY(attr, '$.video') AS video
FROM products
WHERE type = 'M';
```

| Name   | Audio                           | Video                                    |
|--------|---------------------------------|------------------------------------------|
| Aliens | [ "DTS HD", "Dolby Surround" ]  | { "resolution": "1080p", "aspectRatio": "1.85:1" } |

## Field equals

Creating and Querying JSON Documents

    SELECT id, name, format, price
    FROM products
    WHERE type = 'M' AND
        JSON_VALUE(attr, '$.video.resolution') = '1080p';

| Name  | Format | Price | Aspect_ratio |
|-------|--------|-------|--------------|
| Aliens | Blu-ray | 13.99 | 1.85:1 |

## Indexing

Creating and Querying JSON Documents

```sql
ALTER TABLE products ADD COLUMN
    video_resolution VARCHAR(5) AS (JSON_VALUE(attr, '$.video.resolution')) VIRTUAL;
EXPLAIN SELECT name, format, price FROM products WHERE video_resolution = '1080p';
```

| ID | Select_type | Table    | Type | Possible_keys |
|----|-------------|----------|------|---------------|
| 1  | SIMPLE      | Products | ALL  | NULL          |

## Indexing

### Creating and Querying JSON Documents

```sql
CREATE INDEX resolutions ON products(video_resolution);

EXPLAIN SELECT name, format, price FROM products WHERE video_resolution = '1080p';
```

| ID  | Select_type | Table    | Ref | Possible_keys |
| --- | ----------- | -------- | --- | ------------- |
| 1   | SIMPLE      | Products | ref | resolutions   |

## Insert field

### Modifying JSON documents

```sql
UPDATE products SET attr = JSON_INSERT(attr, '$.disks', 1) WHERE id = 1;
SELECT name, format, price,
    JSON_VALUE(attr, '$.disks') AS disks
FROM products
WHERE type = 'M';
```

| Name   | Format | Price | Disks |
|--------|--------|-------|-------|
| Aliens | Blu-ray| 13.99 | 1     |

## Insert array

Modifying JSON Documents

```sql
UPDATE products SET attr = JSON_INSERT(attr, '$.languages',
JSON_ARRAY('English', 'French')) WHERE id = 1;

SELECT name, format, price, JSON_QUERY(attr, '$.languages') AS languages
FROM products WHERE type = 'M';
```

| Name   | Format | Price | Languages      |
|--------|--------|-------|----------------|
| Aliens | Blu-ray| 13.99 | ["English", "French"] |

## Add array element

### Modifying JSON Documents

```sql
UPDATE products SET attr = JSON_ARRAY_APPEND(attr, '$.languages', 'Spanish')
WHERE id = 1;
SELECT name, format, price, JSON_QUERY(attr, '$.languages') AS languages
FROM products WHERE type = 'M';
```

| Name   | Format | Price | Languages                           |
|--------|--------|-------|-------------------------------------|
| Aliens | Blu-ray | 13.99 | [ "English", "French", "Spanish" ] |

## Remove array element

### Modifying JSON Documents

```sql
UPDATE products SET attr = JSON_REMOVE(attr, '$.languages[0]') WHERE id = 1;

SELECT name, format, price, JSON_QUERY(attr, '$.languages') AS languages
FROM products WHERE type = 'M';
```

| Name   | Format | Price | Languages                |
|--------|--------|-------|--------------------------|
| Aliens | Blu-ray| 13.99 | [ "French", "Spanish" ]  |

## Relational to JSON

### Creating JSON Documents From Relational Data

```sql
SELECT JSON_OBJECT('name', name, 'format', format, 'price', price) AS DATA
FROM products
WHERE type = 'M';
```

### Data

```json
{
  "name": "Tron",
  "format": "Blu-ray",
  "price": 29.99
}
```

## Relational + JSON

### Creating JSON Documents From Relational Data

```sql
SELECT JSON_OBJECT('name', name, 'format', format, 'price', price, 'resolution',
    JSON_VALUE(attr, '$.video.resolution')) AS DATA
FROM products
WHERE type = 'M';
```

### Data

```json
{
    "name": "Aliens",
    "format": "Blu-ray",
    "price": 13.99,
    "resolution": "1080p"
}
```

## Relational + JSON (merge)

### Creating JSON Documents From Relational Data

```sql
SELECT JSON_MERGE(
    JSON_OBJECT(
      'name', name,
      'format', format),
    attr) AS DATA
FROM products
WHERE type = 'M';
```

### Data

```json
{
  "name": "Tron",
  "format": "Blu-ray",
  "video": {
    "resolution": "1080p",
    "aspectRatio": "1.85:1"},
  "cuts": [
    {"name": "Theatrical", "runtime": 138},
    {"name": "Special Edition", "runtime": 155}],
  "audio": [
    "DTS HD",
    "Dolby Surround"]
}
```

## Constraints

Enforcing Data Integrity With JSON Documents

```sql
ALTER TABLE products ADD CONSTRAINT check_attr
  CHECK (
    type != 'M' or (type = 'M' and
      JSON_TYPE(JSON_QUERY(attr, '$.video')) = 'OBJECT' and
      JSON_TYPE(JSON_QUERY(attr, '$.cuts')) = 'ARRAY' and
      JSON_TYPE(JSON_QUERY(attr, '$.audio')) = 'ARRAY' and
      JSON_TYPE(JSON_VALUE(attr, '$.disks')) = 'INTEGER' and
      JSON_EXISTS(attr, '$.video.resolution') = 1 and
      JSON_EXISTS(attr, '$.video.aspectRatio') = 1 and
      JSON_LENGTH(JSON_QUERY(attr, '$.cuts')) > 0 and
      JSON_LENGTH(JSON_QUERY(attr, '$.audio')) > 0));
```

## Constraints

Enforcing Data Integrity With JSON Documents

```sql
INSERT INTO products (type, name, format, price, attr) VALUES 
('M', 'Tron', 'Blu-ray', 29.99, '{"video": {"aspectRatio": "2.21:1"}, "cuts": 
[{"name": "Theatrical", 
  "runtime": 96}], "audio": ["DTS HD", "Dolby Digital"], "disks": 1}');
```

```sql
ERROR 4025 (23000): CONSTRAINT `check_attr` failed for `test`.`products`
```

NO RESOLUTION FIELD!

## Constraints

### Enforcing Data Integrity With JSON Documents

```sql
INSERT INTO products (type, name, format, price, attr) VALUES 
  ('M', 'Tron', 'Blu-ray', 29.99, '{"video": {"resolution": "1080p", "aspectRatio": "2.21:1"}, "cuts": 
  [{"name": "Theatrical", "runtime": 96}], "audio": ["DTS HD", "Dolby Digital"], "disks": "one"}');
```

`ERROR 4038 (HY000): Syntax error in JSON text in argument 1 to function 'json_type' at position 1`

“ONE” IS NOT A NUMBER!

# Accessing MariaDB with APIs

## Application program interface (API)

Database API allows an Application Server to interface easily with a MariaDB Server:

- Standardised communication interface
- Serves as bottom-most layer in the communications tack on top of which SQL runs

End Users → Web Page → Application Server → Database Server

Browser App → Backend / API → MariaDB

## Minimal steps for an application

**Preparation**

- Install and configure API software on server
- Create a MariaDB user account for application

**Basic components of application**

- Establish a connection to server
- Query the database
- Parse the data and display to user

## Api database user account

### Create User Account for API

Set Host to Localhost

Limit Privileges (e.g., Only `SELECT`)

```sql
CREATE USER 'api_user'@'localhost' IDENTIFIED BY 'mariadb';
GRANT SELECT ON api.* TO 'api_user'@'localhost';
```

## Installing and configuring - Java

Connector/J is the standard connector for Java

Built on top of JDBC

Available from MariaDB web site as binary download (compiled classes in JAR)

Just place the JAR somewhere in your CLASSPATH

## Connecting to MariaDB - Java

Connect a Java Application to MariaDB

- Import required packages
- Authentication Variables
- New Connection Variable
- Error Handling via try… catch (throws SQLException)

```java
import java.sql.Connection;
import java.sql.DriverManager;
import import java.sql.SQLException;

String host = 'localhost';
String user = 'api_user';
String pw = 'mariadb';
String db = 'api';

Connection conn = DriverManager.getConnection("jdbc:mysql://" 
+ host + "/" + db + "?" + "user=" + user + "&password=" + pw);
```

## Querying MariaDB - Java

**Import required packages**

**Query Server**

- Declare prepared statement
- Init prepared statement
- Bind variable to statement
- Execute statement
- Fetch Results (Loop through Results & Display)

```java
import java.sql.PreparedStatement;
import java.sql.ResultSet;

String sql = "SELECT common_name, saber_color
              FROM jedis
              WHERE LOWER(common_name) LIKE LOWER(?)";

PreparedStatement stmt = connect.prepareStatement(sql);

stmt.setString(1, "some_value");
stmt.executeQuery();
ResultSet rs = stmt.getResultSet();
while (rs.next()) {
    System.out.println(rs.getString("common_name")
    + " - " + rs.getString("saber_color"))
}
```

## Installing and configuring - R2DBC

Standard connector for Java using the Reactive Relational Database Connectivity (R2DBC) API

R2DBC operations are non-blocking, which makes the R2DBC API more scalable than Java’s standard JDBC API.

Capable of being used with a native R2DBC implementation or in combination with an R2DBC Client Library (e.g. Spring Data R2DBC).

Get it from MariaDB web site as binary download (compiled classes in JAR)

Just place the JAR somewhere in your `CLASSPATH`

The R2DBC specification is currently pre-GA and is planned to be released GA (v 1.0) by end of 2021.

## Connecting to MariaDB - R2DBC

Connect an R2DBC application to MariaDB

```java
import org.mariadb.r2dbc.MariadbConnectionConfiguration;
import org.mariadb.r2dbc.MariadbConnectionFactory;
import io.r2dbc.spi.Connection;

MariadbConnectionConfiguration conf = 
    MariadbConnectionConfiguration.builder()
        .host("192.0.2.1")
        .port(3306)
        .username("connr2dbc_test")
        .password("passwd")
        .database("test")
        .build();

MariadbConnectionFactory connFactory = new MariadbConnectionFactory(conf);

Connection conn = connFactory.create().block();
```

## Querying MariaDB - R2DBC

### Import required packages from R2DBC

### Query Server
- Declare a statement
- Execute the statement and map back the result to an iterator
- Define the action inside the iterator
- Define the action outside the iterator

```java
import io.r2dbc.spi.Statement;
import io.r2dbc.spi.Result;
import io.r2dbc.spi.Row;
import io.r2dbc.spi.RowMetadata;
import reactor.core.publisher.Flux;

Statement stmt = conn.createStatement("SELECT name, email FROM contacts");

for (String contact_entry : 
    Flux.from(stmt.execute()).flatMap( res -> res.map( (row, metadata) -> {
            return String.format("%s: %s", 
            row.get(0, String.class), 
            row.get(1, String.class) 
    ))).toIterable()) { 
    System.out.println(contact_entry); 
}
```

## Installing and configuring - Node.js

- Multiple legacy connectors exists like `mysql` and `mysql2`
- MariaDB provides its own connector `mariadb`
  - Higher performance
  - Modern API (promise) and legacy support (callbacks)
- Available in NPM
  - `# npm install mariadb`

## Connecting to MariaDB - Node.js

Connect a Node.js Program to MariaDB

- Include the mariadb module
- Authentication Variables
- New Connection Variable
- Error Handling
- Disconnecting

```javascript
const mariadb = require('mariadb');

const host = 'localhost';
const user = 'api_user';
const pw = 'mariadb';
const db = 'api';

var conn = mariadb.createConnection({host: host, user: user, password: pw, database: db})
  .catch(err => {
    console.log(err);
  });

conn.end();
```

## Querying MariaDB - Node.js

Declare a SQL statement

Query Server
- Use the Promise interface
- Fetch Results
- Handle potential errors

Close Statement Handle and Connection

```javascript
const sql = "SELECT common_name, saber_color
             FROM jedis
             WHERE LOWER(common_name) LIKE LOWER('%some_value%')";
             
conn.query(sql)
.then(row => {
    console.log(row[0].common_name + ' - ' + row[0].saber_color);
})
.catch(err => {
    //handle connection error
});
.finally(() => {
    conn.end();
});
```

## Installing and configuring - Python

Native connector for MariaDB exists under the name `mariadb`

Open-source project, created and maintained by the community

Available in the Python Package Index (PyPI) repository

`# pip install mariadb`

## Connecting to MariaDB - Python

Connect a Python Program to MariaDB

- Import the mariadb module
- Set the connection parameters
- Errors will generate exceptions, so use try/catch to handle any of them
- Get a connection cursor to run your queries

```python
import mariadb

# Connect to MariaDB Platform
try:
    conn = mariadb.connect(user="db_user", password="db_user_passwd", host="192.0.2.1", port=3306, database="employees")
except mariadb.Error as e:
    print(f"Error connecting to MariaDB Platform: {e}")

# Get Cursor
cur = conn.cursor()
```

## Querying MariaDB - Python

Queries are run through a cursor

Statements will be prepared on-the-fly

AUTOCOMMIT is on by default

Connection should be closed when not needed any more

```python
# Get data
some_name = "George"
cur.execute("SELECT first_name, last_name FROM employees WHERE first_name=?", (some_name,))
for first_name, last_name in cur:
    print(f"First name: {first_name}, Last name: {last_name}")

# Add data
try:
    cur.execute("INSERT INTO employees (first_name, last_name) VALUES (?, ?)", ("Maria","DB"))
except mariadb.Error as e:
    print(f"Error: {e}")

# Close connection
conn.close()
```

## Installing and configuring - C++

MariaDB Connector/C++ is a native C++ driver that can be used within C++ projects

The standard C client library can also be used in C++ projects

MariaDB Connector/C++ can be downloaded and installed for use on Linux and Windows

For more information on getting started with Connector/C++ see the official documentation -  
[https://mariadb.com/docs/clients/connector-cpp/#connector-cpp-installation](https://mariadb.com/docs/clients/connector-cpp/#connector-cpp-installation)

## Connecting to MariaDB - C++

Connect a C++ Program to MariaDB Connector/C++

- Including required headers
- Defining connection properties
- Creating an account
- Connecting to the database
- Disconnecting

```cpp
#include <mariadb/conncpp.hpp>

sql::Driver* driver = 
    sql::mariadb::get_driver_instance();

sql::SQLString
url("jdbc:mariadb://192.0.2.1:3306/employees")
sql::Properties properties({{"user",
    "db_user"}, {"password", "db_user_password"}});

std::unique_ptr<sql::Connection>
    conn(driver->connect(url, properties));

conn->close();
```

## Querying MariaDB - C++

### Query Server

Include required headers

Define a prepared statement

Prepare the statement

Bind variables

Execute the query and obtain result set

Loop over the result set to read it

```cpp
#include <mariadb/conncpp.hpp>

#define SQL "SELECT common_name, saber_color FROM jedis WHERE LOWER(common_name) LIKE LOWER(?)"

std::unique_ptr<sql::Statement> stmt(conn->createStatement());

stmt->setString(1, "some_value");

sql::ResultSet* res = stmt->executeQuery("SELECT common_name, saber_color FROM jedis WHERE LOWER(common_name) LIKE LOWER(?)");

while (res->next()) {
    cout << res->getString(0);
    cout << res->getString(1);
}
```

## Installing and configuring - C

MariaDB provides a client shared object library with header files

Use `yum` or similar to install the development package:

```
# yum install MariaDB-devel
```

Alternatively, if you compile MariaDB from source, client library and headers will be installed locally

## Connecting to MariaDB - C

Connect a C Programme to MariaDB

Including the required header  
Defining connection properties  
Creating a new connection handler  
Connecting to server  
Handling errors  
Disconnecting from server  

```c
#include <mysql.h>

#define HOST "localhost"
#define USER "api_user"
#define PW "mariadb"
#define DB "api"

MYSQL *conn = mysql_init(NULL);
if (!mysql_real_connect(conn, HOST, USER, PW, DB, 3306, NULL, 0))
    printf("%s\n", mysql_error(conn));

mysql_close(conn);
```

## Querying MariaDB - C

Define a SQL query

Query Server
- Execute the query
- Retrieve a result set
- Iterate over the result set to read values

Free resources

```c
#define SQL "SELECT common_name, saber_color FROM jedis WHERE LOWER(common_name) LIKE LOWER('some_value')"

mysql_query(conn, sql);

MYSQL_RES *rs = mysql_store_result(conn);

while (MYSQL_ROW row = mysql_fetch_row(rs))
    printf("%s = %s\n", row[0], row[1]);

mysql_free_result(rs);
```

# Non-native MariaDB connectors

## Installing and configuring - .NET

Multiple connectors exist, both free to use and commercial ones.

MySql Connector is a truly async MySQL ADO.NET provider, supporting MariaDB

It has a permissive license (MIT)

It is available from the NuGet repository

```
C:\> Install-Package MySqlConnector -Version 1.3.8
```

## Connecting to MariaDB - .NET

Connect a .NET Program to MariaDB

- Define the necessary connection properties and pass them to a new instance of the connection
- Open the connection

```csharp
using (var connection = new MySqlConnection("Server=myserver;User ID=mylogin;Password=mypass;Database=mydatabase"))
{
    connection.Open();
}
```

## Querying MariaDB - .NET

Use an existing connection.

Define the query string as a new instance of the MySqlCommand class.

Pass this new instance to the connection and it will execute it.

Get back a reader from the command class.

Loop around the reader to fetch the result set row by row.

```csharp
using (var connection = new MySqlConnection("Server=myserver;User ID=mylogin;Password=mypass;Database=mydatabase"))
{
    connection.Open();

    using (var command = new MySqlCommand("SELECT field FROM table;", connection))
    using (var reader = command.ExecuteReader())
    while (reader.Read())
    Console.WriteLine(reader.GetString(0));
}
```

## Installing and configuring - PHP

Three PHP APIs (*mysql*, *mysqli*, and PDO Extensions)  
- MySQL Improved (*mysqli*) is Recommended

Use `yum` or similar Utility to Install PHP and *mysqli*

```
# yum install php php-mysql
```

The web server should be configured to render PHP pages properly; if PHP is installed from a distribution package, this is usually included in the package.

## Connecting to MariaDB - PHP

Connect a Web Page or Program to MariaDB

- PHP Tags
- Authentication Variables
- New Connection Variable
- Error Handling

```php
<?php
$host = 'localhost';
$user = 'api_user';
$pw = 'mariadb';
$db = 'api';

$connect = new mysqli($host, $user, $pw, $db);

if (mysqli_connect_errno()) {
    printf("Connect failed: %s\n",
    mysqli_connect_error());
    exit();
}
?>
```

## Querying MariaDB - PHP

- **Capture & Parse User Input**

- **Query Server**
  - Store SQL Statement with Placeholder
  - Prepare Connection with SQL Statement
  - Prepare Variables to Store Fields from Results
  - Fetch Results (Loop through Results & Display)

- **Close Statement Handle and Connection**

```php
<?php
$search_parameter = $_REQUEST['jediname'];
$search_parameter = "%" . $search_parameter . "%";
$sql = "SELECT common_name, saber_color
        FROM jedis
         WHERE LOWER(common_name) LIKE LOWER(?)";
$sth = $connect->prepare($sql);
$sth->bind_param('s', $search_parameter);
$sth->execute();
$sth->bind_result($common, $saber_color);
while($sth->fetch()) {
     print "$common - <i>$saber_color</i><br/>";
}
$sth->close();
$connect->close();
?>
```

## Other database connectors

Many other programming languages provide their own connectors for MariaDB Server

Some could be native, others are often based on the MariaDB client shared library

- For Perl, look at the `DBD::MySQL` module, available in CPAN.
- For Ruby, there is a package available in the Rubygem repository.
- For R, use the `RMariaDB` package.
- For Erlang, there is a native implementation named `MySQL/OTP` which can be used with MariaDB Server.
- For dotNet, there are commercially supported connectors from various vendors which may offer advanced features, compared to the open-source one.

## Object-relational mapping (ORM)

Object-Relational Mapping (ORM) is technique that lets you query and manipulate data from a database using an object-oriented paradigm.

There are many ORM libraries/frameworks that exist across a variety of languages. A few examples that have been tested to work with native MariaDB connectors are:

- Python: SQLAlchemy
- Java: Hibernate, jOOQ, JPA
- Node.js: Sequelize

Before incorporating an ORM into a project, make sure you understand how it works with the database and how this relates to your projected workload.

# Error and warning handling

## SQL errors and warnings

**SHOW WARNINGS** Statement returns errors, warnings, and notes

Relevant to the last statement that used a table

Each new **INSERT** that uses a table clears the list first

```sql
CREATE TABLE table1 (col1 TINYINT);

INSERT INTO table1 VALUES (1000);
Query OK, 1 row affected, 1 warning (0.00 sec)

SHOW WARNINGS;
+---------+------+----------------------------------------+
| Level   | Code | Message                                |
+---------+------+----------------------------------------+
| Warning | 1264 | Out of range value for column 'c1' at row 1 |
+---------+------+----------------------------------------+
```

## Operating system errors

Low level errors may affect MariaDB's operation

Things like low disk space, file system permissions, etc

OS level error codes are passed back up the chain

MariaDB includes the `perror` utility for displaying errors

```sql
LOAD DATA INFILE '/root/test' INTO TABLE t1;
ERROR 13 (HY000):
Can't get stat of '/root/test' (Errcode: 13)

# perror 13
Error code 13: Permission denied
```

## Domain name service

MariaDB uses DNS and Reverse DNS for IP and Hostname with Client Connections

A Slow DNS Server can Slow MariaDB Client Connection

Running a Local DNS Server Helps with Lookup Speed and Security

Use `--skip-name-resolve` to Tell MariaDB Not to do DNS Lookup  
Then Use Only IP Addresses in Permissions Tables

## Persistent connections

Some Client Languages and Connection Pool Solutions use Persistent Connections (PHP `mysql_pconnect()`)

Not very Effective  
A New Client Connection is Simpler

Client Session Buffers can Stay Allocated too Long if the Client Pool of Application is not Efficient, Increasing Memory Usage

Transactions can be Held Open if Application is Built Poorly, Reducing Concurrency

## Avoiding divide by zero

MariaDB Fails Silently and Produces a `NULL` Response for Divide-by-zero Error

- Unless SQL Mode `ERROR_FOR_DIVISION_BY_ZERO` is Active

The `COALESCE` and `NULLIF` functions can be used to provide a solution for divide-by-zero errors:

```sql
SET @INPUT1 = 0;
SELECT COALESCE(50 /NULLIF(@INPUT1,0),0) AS response;
+----------+
| response |
+----------+
| 0.0000   |
+----------+
```

```sql
SET @INPUT1 = 5;
SELECT COALESCE(50 /NULLIF(@INPUT1,0),0) AS response;
+----------+
| response |
+----------+
| 10.0000  |
+----------+
```

# Lesson summary

- Differentiate between stored procedures and stored functions using the native MariaDB SQL/PSM language
- Use JSON document stores
- Understand how to access MariaDB via APIs using both native and non-native MariaDB connectors
- Set SQL modes to affect error output and use the `SHOW WARNINGS` and `SHOW ERRORS` statements to interpret error messages

# Lab exercises

- 7-1 Connecting to MariaDB with a PHP-Based Web Form
- 7-2 Connecting to MariaDB with a Python Script
- 7-3 Creating a View
- 7-4 Creating a Stored Procedure in SQL/PSM

