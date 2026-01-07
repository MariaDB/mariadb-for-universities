---
title: "Lesson 1: Getting Started"
url: "/dba/getting-started/"
date: 2025-04-07
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

## Learning Objectives

- List the steps to deploy MariaDB Enterprise Server using distribution packages
- Explain the benefits and challenges of upgrading MariaDB Enterprise Server, and list the steps you need to take before and after the upgrade
- Know which tools and client utilities to use to perform common tasks such as managing users accounts and permissions, benchmarking, and backing up and restoring data
- Change the default settings of MariaDB Enterprise Server while it's running to make the server work better for your needs
- Turn on and off server logs and use them to check the status and performance of MariaDB Enterprise Server

## Deployment

### Supported Operating Systems

| Operating System     | ES10.6 | ES10.5 | ES10.4 | ES10.3 |
|----------------------|--------|--------|--------|--------|
| RHEL8 (x86_64 / ARM64) | Yes    | Yes    | Yes    | Yes    |
| RHEL7 (x86_64)        | Yes    | Yes    | Yes    | Yes    |
| CentOS 7 (x86_64)     | Yes    | Yes    | Yes    | Yes    |
| Ubuntu 20.04 (x86_64 / ARM64) | Yes    | Yes    | Yes    | Yes    |
| Ubuntu 18.04 (x86_64 / ARM64) | Yes    | Yes    | Yes    | Yes    |
| Debian 11 (x86_64 / ARM64)    | Yes    | Yes    | No     | No     |
| Debian 10 (x86_64 / ARM64)    | Yes    | Yes    | Yes    | Yes    |
| Debian 9 (x86_64 / ARM64)     | Yes    | Yes    | Yes    | Yes    |
| SLES15 (x86_64 / ARM64)       | Yes    | Yes    | Yes    | Yes    |
| SLES12 (x86_64)               | Yes    | Yes    | Yes    | Yes    |
| Windows (x86_64)              | Yes    | Yes    | Yes    | Yes    |

### Installation Methods and Packages

#### Installation Tools
- Built and tested by experts
- Installer (e.g., `yum`)
- Use up-to-date versions

#### Distribution Packages
- `(rpm, deb, pkg, dmg)`
- Easy and quick to install
- Need to configure manually
- Possibly old versions

#### Docker
- Enterprise Docker registry
- Easy Docker installs
- Maintained by MariaDB

The recommended way to install MariaDB is via MariaDB's `repository` in conjunction with your distribution's package manager.

### Installing with Packages

**Package Managers**  
Automatically places binaries, configuration and other files in correct locations

**Configuration File**  
Add any system configuration items you require to the `.cnf` file

**Maintained by MariaDB**  
Official package repositories are available on our website

### Starting and Stopping the Server

```shell
$ systemctl [start|stop|restart|status] mariadb
```

Configure start and stop timeouts by creating a `timeout.conf` file

```shell
$ vi /etc/systemd/system/mariadb.service.d/timeout.conf

[Service]
TimeoutStartSec=30min
TimeoutStopSec=30min
```

Reload the systemd daemon

```shell
$ systemctl daemon-reload
$ systemctl restart mariadb
```

### Verify Your Installation
Run the following command to check installed version:
```bash
mariadb --version
```

When database installed correctly, you should to see output like below 
```shell
mariadb from 11.8.5-MariaDB, client 15.2 for debian-linux-gnu (x86_64) using  EditLine wrapper
```

### Upgrading MariaDB

#### What to Consider

- Review release notes
  (major and minor version upgrades)

- Upgrade system tables (`mariadb-upgrade`)

- Requires downtime on standalone servers

- Rolling restart available with MariaDB Cluster

#### What to Prepare

- Backup Data

- Update applications and libraries

- Use a Test Server
  - MariaDB SkySQL is an easy way to spin up what you need, test and then delete it

- Use Replication
  - Upgrade the Replica, Promote the Replica and then Upgrade the Primary
    - Made even easier with MariaDB MaxScale

## Connectors and Utilities

### Connectors

Various Libraries for connecting to MariaDB

* MariaDB Connector/C
* MariaDB Connector/C++ * 
* MariaDB Connector/J (JDBC) 
* MariaDB Connector/R2DBC
* MariaDB Connector/Node.js 
* MariaDB Connector/ODBC *
* MariaDB Connector/Python *
* .NET, Perl DBI, PHP, Ruby ** 

*Wrapper for MariaDB C API, which uses the MariaDB Network Protocol  
** Non-native (unsupported) Connectors to use with MariaDB  


### Client Connections

TCP/IP Connections available on All Platforms (`skip_networking` disables this)

Socket Files available on Unix Platforms (fastest choice)

Named Pipe (`named_pipe`) and Shared Memory (`shared_memory`) available on Windows

MariaDB Connections require few resources and are easy to open

Most use External Connection Pools (not needed usually)

Set Global Client Connection Limit (`max_connections=n`)

### MariaDB Client - Command-line Tool

- Shell or SSH
- Manually Execute any SQL statement
- Display MariaDB settings and status counters
- Use as an Interactive Session, or Pipe SQL via Shell

```shell
$ mariadb --user root -p
MariaDB [(none)]> SHOW DATABASES;
```

```sh
$ mariadb -p -u user_name --execute "SHOW DATABASES"
$ mariadb -e "CREATE DATABASE world"
$ mariadb world < /path/to/world.sql
```

### MariaDB-admin - Command-line Tool

- Shell or SSH
- Manage User Accounts, Passwords, Permissions
- Display MariaDB settings and status counters
- View and Kill Active Connections
- Ping or Shutdown mysqld
- Create and Drop Databases

```sh
$ mariadb-admin processlist
$ mariadb-admin kill 12345678
$ mariadb-admin -u root -p create ragnar
$ mariadb-admin extended-status
$ mariadb-admin variables
$ mariadb-admin ping
$ mariadb-admin status
$ mariadb-admin shutdown
```

### Other Client Utilities

| Name                           | Description                                                                                                                                                                                                                                                 |
|-------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `mariadb-backup`              | Open source tool provided by MariaDB for performing physical online backups of InnoDB and Aria tables. For InnoDB "hot online" backups are possible.                                                                                                                                               |
| `mariadb-binlog`              | Processes binary logs and relay logs. Used to read the binary log file, and is often used in restoring data to a particular point in time.                                                                                                                                                        |
| `mariadb-check`               | Checks, repairs, analyzes and optimizes tables.                                                                                                                                                                                                                                                   |
| `mariadb-dump`                | Backup tool that allows a logical dump of SQL data or schema from your database.                                                                                                                                                                                                                   |
| `mariadb-secure-installation` | Tool that enables you to improve the security of your MariaDB installation in various ways. You should run this after every installation of MariaDB server.                                                                                                                                       |
| `mariadb-upgrade`             | Tool that checks and updates your tables to the latest version.                                                                                                                                                                                                                                    |
| `mariadb-slap`                | Tool for load-testing MariaDB.                                                                                                                                                                                                                                                                     |
| `sysbench`                    | Another benchmarking tool that provides benchmarking capabilities for Linux. It supports testing CPU, memory, file I/O, mutex performance.                                                                                                                                                          |
| `my_print_defaults`           | Useful in displaying the default settings that are given to the MariaDB daemon at startup, either manually or from the configuration file. Execute this from the command line followed by the --mysqld option.                                                                                    |

### Some Command Line Tools

| Name                      | Description                                                                                                  |
|---------------------------|--------------------------------------------------------------------------------------------------------------|
| `mariadb-tzinfo-to-sql`   | Use to load time zones on systems that have a zoneinfo database to load the time zone tables into the mysql database. |
| `mariadb-install-db`      | Initializes the MariaDB data directory and creates the system tables in the mysql database.                |
| `mariadb-show`            | Shows the structure of a MariaDB database (databases, tables, columns and indexes).                          |
| `mariadb-dump`            | Tool for examining the slow query log.                                                                       |
| `mariadb-dumpslow`        | Tool that enables you to improve the security of your MariaDB installation in various ways. This should be run after every installation of MariaDB server. |
| `innochecksum`            | printing checksums for InnoDB files.                                                                         |
| `aria_chk`                | checks, repairs, optimizes, sorts and gets information about Aria tables.                                    |
| `aria_pack`               | Tool for compressing Aria tables.                                                                            |
| `aria_read_log`           | Tool for displaying and applying log records from an Aria transaction log.                                   |

### Graphical User Interface Tools

- Can Connect via SSL or SSH tunnel
- Schema and Query Construction
- Optimization and Profiling
- User Account Management
- Task and Backup Scheduling
- Data Migration and Cleaning
- Schema Sync and Diff Tools
- Notifications

**Windows**
- HeidiSQL
- DBeaver
- SQLyog
- Toad

**Mac and Linux**
- DBeaver
- Sequel Ace
- Querious

## Configuration and Variables

### Configuration File Locations

#### Linux

- `/etc/my.cnf`
- `/etc/my.cnf.d/mariadb-enterprise.cnf`
- `/etc/my.cnf.d/server.cnf`
- `$MYSQL_HOME/my.cnf`
- `[datadir]/my.cnf`
- `~/.my.cnf (clients)`
- `/etc/my.cnf.d/*.cnf`
- `--defaults-extra-file`

#### Windows

- `C:\Windows\my.ini`
- `C:\Windows\my.cnf`
- `C:\my.ini`
- `INSTALLDIR\my.ini`
- `INSTALLDIR\my.cnf`
- `INSTALLDIR\data\my.ini`
- `INSTALLDIR\data\my.cnf`
- `--defaults-extra-file`

### Server Defaults

#### Finding Defaults

```shell
$ mariadbd --print-defaults
```

mariadbd would have been started with the following arguments:
- `--datadir=/data/mariadb`
- `--socket=/var/lib/mysql/mysql.sock`
- `--user=mysql`
- `--symbolic_links=0`
- `--local_infile=0`

#### Configuration File (server.cnf)

```ini
[server]

[mysqld]

[galera]

[embedded]

[mariadb]
datadir=/data/mariadb
socket=/var/lib/mysql/mysql.sock
user=mysql
symbolic_links=0
local_infile=0

[mariadb-10.6]
```

### Setting the InnoDB Buffer Pool Size

The size of the buffer pool is the most important tunable for the InnoDB storage engine

The buffer pool size is determined by the innodb_buffer_pool_size

```ini
innodb_buffer_pool_size = 134217728

innodb_buffer_pool_chunk_size = 134217728

innodb_buffer_pool_dump_at_shutdown = ON

innodb_buffer_pool_load_at_startup = ON

innodb_buffer_pool_filename = ib_buffer_pool
```

### Global and Status Variables

#### Global Variables

- Global Variables are System Wide
- Set in configuration file at start
- Can be fetched with SHOW statement

  ```sql
  -- list of all global variables
  SHOW GLOBAL VARIABLES;

  -- get variable by name
  SHOW GLOBAL VARIABLES WHERE Variable_name = '<valiabe_name>';

  -- get variables by mask
  SHOW GLOBAL VARIABLES LIKE 'innodb%';
  ```

- Can be changed with SET statement

  ```sql
  SET GLOBAL tmp_table_size = 32*1024*1024;
  ```

  Only user accounts with the SUPER privilege can dynamically change global system variables.

#### Global Status Variables

- SHOW GLOBAL STATUS provides Counters and Thresholds of Internal Statistics

  ```sql
  -- get all status variables with values
  SHOW GLOBAL STATUS;

  -- get status variable by name 
  SHOW GLOBAL STATUS WHERE Variable_name = 'Opened_tables';

  -- get status variable by maask 
  SHOW GLOBAL STATUS LIKE '%innodb_buffer_pool%';
  ```

- Check Status Variables to Decide on Adjustments to Global Variables

### Session Variables

Global Variable Values are Defaults for New Sessions

```sql
SHOW GLOBAL VARIABLES LIKE '%character_set_client%';
```
```shell
+----------------------+--------+
| Variable_name        | Value  |
+----------------------+--------+
| character_set_client | latin1 |
+----------------------+--------+
```

Session Variables are Values for Current Session

```sql
SHOW SESSION VARIABLES LIKE '%character_set_client%';
```
```shell
+----------------------+-------+
| Variable_name        | Value |
+----------------------+-------+
| character_set_client | greek |
+----------------------+-------+
```

## Log Files

### MariaDB Logs

- ERROR LOG
- GENERAL QUERY LOG
- SQL ERROR LOG
- SLOW QUERY LOG
- BINARY LOG
- AUDIT LOG

### Error Log

- Required for the server to operate
- Contains startup, shutdown, and error messages
- Unix uses `stderr` and sends to `host_name.err` in datadir
- Systems that use systemd redirect to the syslog by default
- Windows uses `host_name.err` in the datadir, or System Event Log

```ini
[mariadb]
log_error = /path/to/error_log_file_name
log_warnings = 1
```

_Excerpt from server.cnf, my.cnf or my.ini configuration file_

_Set `log_warnings` to 2 for verbose mode_

### General Query Log

- Logs all queries received from all clients, even queries with syntax errors
- Order Received - Not Executed
- Potential Problems with Log
  - Possible disk I/O bottleneck on high-traffic servers
  - Log can quickly become huge
  - Contains queries in plain text so make sure to secure the logs as you would the data in the database
  - Reduces performance by up to 20%

```ini
[mariadb]
general_log
general_log_file='/path/to/host.log'
```

Excerpt from server.cnf, my.cnf or my.ini configuration file

### SQL Error Log

- Logs SQL errors
  - Records error messages with SQL statement
  - Saves username and host for SQL statement
- Used for Detecting SQL Injections
- Logs User, Host, and Time
- Dynamically loadable

```sql
INSTALL PLUGIN sql_error_log SONAME 'sql_errlog';
```

Requires `INSERT` privilege for `mysql.plugin` table

### Slow Query Log

- Queries that take more seconds to execute than `long_query_time`
- Contains queries in plain text (security risk!)
- Useful when testing an application or new database

```ini
[mariadb]
slow_query_log = 1
slow_query_log_file = /path/to/mariadb-slow.log
long_query_time = 2.0
# log_queries_not_using_indexes
log_slow_admin_statements=1
log_slow_disabled_statements='admin,call,slave,sp'
```

### Managing Log Files

#### Regularly Rotate Logs File
- Manually move Existing Log Files and then Flush Logs

```sql
FLUSH LOGS;
```

```shell
$ mariadb-admin flush-logs
```

An alternative method from the command-line

#### Backups
- Include Logs in Backups
- Synchronise Binary Logs with Backups

#### Error Log
- Move the log file, then `FLUSH LOGS` to recreate
- Deleted file may still be opened by server

#### Linux logrotate Utility
- Automatic log rotation
- Various conditions can be set for log files (size, time frame)

## Lesson Summary

- List the steps to deploy MariaDB Enterprise Server using distribution packages
- Explain the benefits and challenges of upgrading MariaDB Enterprise Server, and list the steps you need to take before and after the upgrade
- Know which tools and client utilities to use to perform common tasks such as managing users accounts and permissions, benchmarking, and backing up and restoring data
- Change the default settings of MariaDB Enterprise Server while it's running to make the server work better for your needs
- Turn on and off server logs and use them to check the status and performance of MariaDB Enterprise Server

