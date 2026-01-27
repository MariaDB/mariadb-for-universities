---
title: "Lesson 1: Getting Started"
url: "/dev/getting-started/"
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

# Getting started

## Learning objectives

- Discuss MariaDB's client / server architecture
- Gain an understanding of isolation levels and MariaDB's storage engine concept
- Utilize MariaDB connectors and utilities
- Learn basic MariaDB administration tasks relating to server logs, user management, and backing up and restoring data

## Architecture

### Installation methods and packages

#### Linux
- MariaDB repositories
- Distribution repositories
- Binary tarballs
- Docker Container Image
- Source tarballs

#### Windows
- MariaDB installer
- Docker Container Image
- Source tarballs

#### MacOS
- Homebrew
- Macports
- Docker Container Image
- Source tarballs

### Isolation levels

When two or more transactions occur at the same time, the isolation level defines the degree at which a transaction is isolated from the resource or data modifications made by other transactions.

The default isolation level is REPEATABLE-READ.

To change the isolation level, you need to set the `tx_isolation` variable which is dynamic and has session level scope.

```sql
SET SESSION tx_isolation = 'READ-COMMITTED';
```

The `transaction_isolation` option can be set in the configuration file.

```ini
[mariadb]
transaction_isolation = READ-COMMITTED
```

### Types of isolation levels

Read Uncommitted  
Allows a transaction to see uncommitted changes made by other transactions (*dirty read*).

Read Committed  
Allows a transaction to see changes made by other transactions (different row content or fewer rows due to a DELETE, or rows no longer match due to an UPDATE) only if the changes have been committed (*non-repeatable read*).

Repeatable Read  
Ensures that if a transaction issues the same SELECT twice, no rows vanish or show different content. New rows (*phantom rows*) can still appear.  
Due to the implementation of row locks in InnoDB there are no phantom rows in InnoDB.

Serializable  
Completely isolates a transaction’s effects from other transactions.  
In InnoDB this causes read locks after SELECT until COMMIT or ROLLBACK (like WITH LOCK IN SHARE MODE).

### Storage engine concept

```sql
SHOW ENGINES;

CREATE TABLE table1 (col1 INT) ENGINE = Aria;

ALTER TABLE table1 ENGINE = InnoDB;
```

Multiple Storage Engines on a Server and within a Query are Allowed  
They Determine Storage Medium (disk, memory, etc.)

Atomicity Consistency Isolation Durability (ACID)
- A: Transactions are All or None
- C: Transactions preserve the integrity of the data
- I: Transactions are Not Visible to Other Sessions
- D: Data is saved and persists, dependable

Lock at certain Levels (table, page, row)

Some offer Special Features (Foreign Keys, GIS, Columnar Data)

Some Provide optimization

## Connectors and utilities

### Connectors

Various Libraries for connecting to MariaDB

**Connectors**

- MariaDB Connector/C
- MariaDB Connector/Node.js
- * MariaDB Connector/C++
- * MariaDB Connector/ODBC
- MariaDB Connector/J (JDBC)
- * MariaDB Connector/Python
- MariaDB Connector/R2DBC
- ** .NET, Perl DBI, PHP, Ruby

\* Wrapper for MariaDB C API, which uses the MariaDB Network Protocol

\** Non-native (unsupported) Connectors to use with MariaDB

### Client connections

- TCP/IP Connections available on All Platforms (`skip_networking` disables this)
- Socket Files available on Unix Platforms *(fastest choice)*
- Named Pipe (`named_pipe`) and Shared Memory (`shared_memory`) available on Windows
- MariaDB Connections require few resources and easy to open
- Most use External Connection Pools *(not needed usually)*
- Set Global Client Connection Limit (`max_connections=n`)

### Mariadb client - command-line tool

- Shell or SSH
- Manually Execute any SQL statement
- Display MariaDB settings and status counters
- Use as an Interactive Session, or Pipe SQL via Shell

```sh
# mariadb --user=root -h hostname -P 3306 -p
MariaDB [(none)]> SHOW DATABASES;
```

```sh
# mariadb -p -u user_name --execute "SHOW DATABASES"
# mariadb -e "CREATE DATABASE world"
# mariadb world < /path/to/world.sql
```

### Mariadb-admin - command-line tool

- Shell or SSH
- Useful for scripting
- Manage User Accounts, Passwords, Permissions
- Display MariaDB settings and status counters
- View and Kill Active Connections
- Ping or Shutdown mysqld
- Create and Drop Databases

```sh
# mariadb-admin processlist
# mariadb-admin kill 12345678
# mariadb-admin -u root -p create ragnar
# mariadb-admin extended-status
# mariadb-admin variables
# mariadb-admin ping
# mariadb-admin status
# mariadb-admin shutdown
```

### Graphical user interface tools

- Can connect via SSL or SSH tunnel
- Schema and query construction
- Optimization and profiling
- User account management
- Task and backup scheduling
- Data migration and cleaning
- Schema sync and diff tools
- Notifications

**Windows**
- HeidiSQL
- DBeaver
- SQLyog
- Toad

**Mac and Linux**
- DBeaver
- SequelPro
- Querious

### Business intelligence and analysis tools

Can Connect via SSL or SSH tunnel

Schema and Query Construction

Data Visualization and Analysis

Use MariaDB Connectors

- **Microsoft PowerBI** - Connector/ODBC
- **Tableau** - Connector/ODBC
- **Apache Zeppelin** - Connector/J
- **Jupyter Notebook** - Connector/Python

## Basic administration

### Configuration file locations

#### Linux

`/etc/my.cnf`  
`/etc/my.cnf.d/*.cnf`  
`$MYSQL_HOME/my.cnf`  
`[datadir]/my.cnf`  
`~/.my.cnf (clients)`  
`--defaults-file`  
`--defaults-extra-file`

#### Windows

`C:\Windows\my.ini`  
`C:\Windows\my.cnf`  
`C:\my.ini`  
`INSTALLDIR\my.ini`  
`INSTALLDIR\my.cnf`  
`INSTALLDIR\data\my.ini`  
`INSTALLDIR\data\my.cnf`  
`--defaults-extra-file`

### Server defaults

#### Finding Defaults

```sh
# mariadbd --print-defaults

mariadbd would have been started with the following arguments:
--datadir=/data/mysql
--socket=/var/lib/mysql/mysql.sock
--user=mysql
--symbolic_links=0
--local_infile=0
```

#### Configuration File (Server.Cnf)

```sh
[server]

[mysqld]

[galera]

[embedded]

[mariadb]
datadir=/data/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
symbolic_links=0
local_infile=0

[mariadb-10.4]
```

### Starting and stopping the server

`systemctl [start|stop|restart|status] mariadb`

Configure start and stop timeouts by creating timeout.conf file

```sh
# vi /etc/systemd/system/mariadb.service.d/timeout.conf

[Service]
TimeoutStartSec=30min
TimeoutStopSec=30min
```

Reload the systemd daemon
`systemctl daemon-reload`
`systemctl restart mariadb`

### Error log

- Active by default
- Contains startup, shutdown, and error messages
- Unix uses stderr, sent to `host_name.err` in datadir
- Systems that use systemd redirect to the syslog by default
- Windows uses `host_name.err` in datadir, or System Event Log

```sh
[mariadb]
log_error = /path/to/error_log_file_name
log_warnings = 1
```
    
*Excerpt from my.cnf or my.ini configuration file*

*Set `log_warnings` to 2 for verbose mode*

### Sql error log

- Logs SQL errors
  - Records error messages with SQL statement
  - Saves username and host for SQL statement
- Used for detecting SQL injections
- Logs user, host, & time
- Dynamically loadable

```sql
MariaDB [(none)]> INSTALL PLUGIN sql_error_log SONAME 'sql_errlog';
```
Requires INSERT privilege for mysql.plugin table

### Slow query log

- Queries that take more seconds to execute than `long_query_time`
- Contains Queries in Plain Text (security risk!)
- Useful when testing an application or new database
- Use `mariadb-dumpslow` to Read Log

```ini
[mariadb]
slow_query_log = 1
slow_query_log_file = /path/to/mariadb-slow.log
long_query_time = 2.0
# log_queries_not_using_indexes
log_slow_admin_statements=1
log_slow_disabled_statements='admin,call,slave,sp'
```

### Mariadb user accounts

- Authentication is based on user, host and password
- Privileges are based on user and host combined, and the user's current role
- Privileges can be granted globally, at the database level, at the table level or at the column level

### Creating users

Changes made with the account management statements are automatically activated

Use the `CREATE USER` statement to create users in MariaDB

```sql
CREATE USER 'user'@'host' IDENTIFIED BY 'password';
```

Resource limits such as the number of queries, updates or connections per hour can be set by appending the resource_option to the `CREATE USER` statement

```sql
CREATE USER 'user'@'host'
    WITH MAX_QUERIES_PER_HOUR 10
    MAX_UPDATES_PER_HOUR 20
    MAX_CONNECTIONS_PER_HOUR 30
    MAX_USER_CONNECTIONS 40;
```

Use `MAX_USER_CONNECTIONS` to limit the number of simultaneous connections by a user

Append the following tls_option to the `CREATE USER` statement to specify the requirement and details of secure SSL connections per user

```sql
CREATE USER 'foo'@'test'
    REQUIRE ISSUER 'foo_issuer'
    SUBJECT 'foo_subject'
    CIPHER 'text';
```

### Checking users

The `SHOW CREATE USER` statement is a useful way to see the command required to create a user for auditing or the creation of similar accounts

```sql
SHOW CREATE USER 'user'@'host'\G
```

### Managing users

Changes made with the account management statements are automatically activated.

**Change a user’s username and host**

RENAME USER statement

```sql
RENAME USER 'user1'@'host1' TO 'user2'@'host2';
```

**Change a user’s password**

SET PASSWORD statement

```sql
SET PASSWORD FOR 'user'@'host' = PASSWORD ('mariadb2');
```

**The ALTER USER statement allows for easy modification of existing user accounts**

Changing a user's password can be rewritten to use the ALTER USER statement

```sql
ALTER USER CURRENT_USER() IDENTIFIED BY 'mariadb2';
```

### Managing users

`ALTER USER` statement to limit the number of simultaneous connections

```sql
ALTER USER 'user'@'host' WITH MAX_USER_CONNECTIONS 10;
```

The `ALTER USER` statement can also be used to set certain TLS-related restrictions

```sql
ALTER USER 'user2'@'host2'
REQUIRE SUBJECT '/CN=alice/O=My Dom, Inc./C=US/ST=Oregon/L=Portland'
AND ISSUER '/C=FI/ST=Somewhere/L=City/O=Some Company/CN=Peter Parker/emailAddress=p.parker@marvel.com'
AND CIPHER 'TLSv1.2';
```

### Assigning privileges

The `mysql` database contains the grants tables, which are loaded into memory when MariaDB starts up

Users and roles are assigned privileges with the `GRANT` statement

```sql
GRANT
  priv_type [(column_list)]
    [, priv_type [(column_list)]] ...
    ON [object_type] priv_level
    TO user
    [WITH GRANT OPTION
     | MAX_QUERIES_PER_HOUR count
     | MAX_UPDATES_PER_HOUR count
     | MAX_CONNECTIONS_PER_HOUR count
     | MAX_USER_CONNECTIONS count ]
```

### Available privileges (show privileges)

- **Basic Privileges**
  - Usage
  - Select
  - Insert
  - Update
  - Delete
  - Show Databases

- **Customizing Privileges**
  - Create Routine
  - Alter Routine
  - Execute
  - Event
  - Trigger

- **Administrator Privileges**
  - All [Privileges]
  - Super
  - Create User
  - Grant Option
  - Process
  - File
  - Reload
  - Shutdown

- **Developer Privileges**
  - Create
  - Alter
  - Index
  - Drop

- **Special Privileges**
  - `Create Temporary Tables`
  - `Create View`
  - `Show View`
  - `Lock Tables`

- **Replication Privileges**
  - Replication Client
  - Replication Slave

### Checking & revoking privileges

The REVOKE statement removes privileges but does not remove the user account.

To remove a user account, the DROP USER statement must be used.

Query the user table in the mysql database to check the privileges of user accounts

```sql
SELECT user,host from mysql.user;
```

The SHOW GRANTS statement can be used to check a single user account’s privileges

```sql
SHOW GRANTS FOR 'user'@'host';
```

Revoke privileges from a user with the REVOKE statement by either listing the privileges to revoke or using ALL PRIVILEGES

```sql
REVOKE SELECT ON dbname.tablename FROM 'user'@'host';
REVOKE ALL PRIVILEGES ON dbname.tablename FROM 'user'@'host';
```

Confirm that the revoke was successful with SHOW GRANTS

```sql
SHOW GRANTS FOR 'user'@'host';
```

### Restricting users

User accounts can have a limit set on resource use

```sql
GRANT SELECT ON dbname.tablename TO 'user'@'host'
WITH MAX_QUERIES_PER_HOUR 20
     MAX_CONNECTIONS_PER_HOUR 10
     MAX_USER_CONNECTIONS 2
     MAX_UPDATES_PER_HOUR 5;
```

Resource limit usage is kept in the `mysql` database

Resource limit counters reset then the server starts|restarts or when `FLUSH USER_RESOURCES` is executed

```sql
FLUSH USER_RESOURCES;
```

### Removing users

Delete users in MariaDB with the `DROP USER` statement

`DROP USER 'user'@'host';`

Check for leftover user accounts which still allow access from other hosts

`SELECT User,Host from mysql.user WHERE User = 'user';`

## Backing up and restoring data

### Basic types of backup methods

#### Physical or Binary Backup

- MariaDB Backup, manual data directory copy (after stopping MariaDB daemon), or VM/Cloud/LVM/ZFS snapshots
  - A physical backup typically refers to a copy of the entire on disk database
  - Used to mitigate a single or multiple host failure
  - Can build replicas quickly
  - Quick Full Recovery Time
  - Slow to recover single row or table (user error)
  - Recovered Only to the Same Storage Engine with the same tablespaces

#### Logical Backup

- mariadb-dump, mydumper, `SELECT INTO OUTFILE`
  - Produces SQL files containing data that can regenerate a database
  - Easily backup and restore single row, table, or database
  - Can segregate schema and data
  - Can work across versions and forks
  - SQL can be parsed with standard UNIX tools
  - Restore process automatically replicated
  - Long Full Restore Time
  - A SQL dump is independent of Storage Engine, and can be Restored to a different Storage Engine, or Used for Migration
  - Process can be Slow and Requires Locks

### MariaDB backup

MariaDB's own backup package (`mariabackup`) is included since MariaDB 10.1

- Supports MariaDB's features and storage engines
- Supports encryption
- Supports incremental backups
- Also available on Windows
- Command-line compatibility with XtraBackup
- MariaDB Enterprise Backup contains improvements such as less locking

### Backing up data - MariaDB backup

#### Full Backups

To use MariaDB Backup you need to create a user on your MariaDB Server with RELOAD, LOCK TABLES and REPLICATION CLIENT privileges.

```sql
MariaDB [(none)]> CREATE USER 'backupuser'@'localhost' IDENTIFIED BY '<password>';
MariaDB [(none)]> GRANT RELOAD, PROCESS, LOCK TABLES, REPLICATION CLIENT ON *.* TO 'backupuser'@'localhost';
```

To take a full backup at the OS command-line use:

```sh
# mariabackup --backup --target-dir <backupdir> --user <backupuser> --password <password>
# mariabackup --prepare --target-dir <backupdir>
```

### Restoring data - MariaDB backup

#### Full Backups

Working with a full backup taken with MariaDB Backup, you restore the backup into an empty data directory

```sh
# mariabackup --copy-back --target-dir <backupdir> --datadir <datadir>
```

Afterwards it might be necessary to set the ownership of the data directory contents

```sh
# chown -R mysql:mysql <datadir>
```

### Backing up data - mariadb-dump

The standard MariaDB logical dump tool copies schema and data to a SQL text file

To use `mariadb-dump` you need to create a user on your MariaDB Server with SELECT, RELOAD, LOCK TABLES, REPLICATION CLIENT, SHOW VIEW, EVENT, AND TRIGGER privileges.

```sql
MariaDB> CREATE USER 'backupuser'@'localhost' IDENTIFIED BY 'mariadb';

MariaDB> GRANT SELECT, RELOAD, LOCK TABLES, REPLICATION CLIENT, SHOW VIEW, EVENT, TRIGGER ON *.* TO 'backupuser'@'localhost';
```

To take a full backup at the OS command line use:

```sh
# mariadb-dump -u backupuser -p --all-databases --single-transaction --flush-logs -r /path/to/full-backup-YYYYMMDD.sql
```

### Restoring data - mariadb-dump

Restoring from a logical backup

```sh
# mariadb < /path/to/full-backup-YYYYMMDD.sql
```

Or to avoid interpretation by bash:

```sh
# mariadb
MariaDB> source /path/to/full-backup-YYYYMMDD.sql
```

You can also use standard linux tools such as vi or grep to edit or extract the backup.

```sh
# grep city worldbackup.sql
CREATE TABLE `world`.`city` ( ...
```

## Lesson summary

- Discuss MariaDB's client / server architecture
- Gain an understanding of isolation levels and MariaDB's storage engine concept
- Utilize MariaDB connectors and utilities
- Learn basic MariaDB administration tasks relating to server logs, user management, and backing up and restoring data

## Lab exercises

- 1-1 Verifying the MariaDB Installation
- 1-2 Enabling Logging in MariaDB
- 1-3 Creating Users in MariaDB
