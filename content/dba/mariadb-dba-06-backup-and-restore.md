---
title: "Lesson 6: Backup and Restore"
url: "/dba/backup-and-restore/"
date: 2025-04-07
description: >
categories: []
tags: []
weight: 6
toc: true
license_note: "For academic and non-commercial usage, licensed under CC BY-NC-SA 4.0 by MariaDB plc."
license: "CC BY-NC-SA 4.0"
license_url: "https://creativecommons.org/licenses/by-nc-sa/4.0/"
---

# Backup and Restore

## Learning Objectives

- Explain key backup principles and their importance, including the importance of verifying and testing backups
- Demonstrate and compare how to create full backups using different backup methods such as `MariaDB Backup` and `mariadb-dump`
- Apply and assess recovery methods to restore data in MariaDB
- Design and implement a backup and recovery plan and schedule

## Backing Up Data

### Backup Principles

- Helps ensure high availability
- Cannot prevent user errors
- Backup data, logs, and configuration files
- Store backups in multiple locations
  - Onsite for fast access
  - Offsite for security
- Create consistent backups

- Backup methods
  - Physical Backup
  - Logical Backup
- Binary logs for Point-in-Time recovery
- **Always test the backup and recovery regularly**

### Basic Types of Backup Methods

#### Physical or Binary Backup

MariaDB Backup, manual data directory copy (after stopping the MariaDB daemon), or VM/Cloud/LVM/ZFS snapshots

- A physical backup typically refers to a copy of the entire on disk database
- Used to mitigate a single or multiple host failure
- Can build replicas quickly
- Quick full recovery time
- Slow to recover single row or table (user error)
- Recovered only to the same storage engine with the same tablespaces

#### Logical Backup

`mariadb-dump`, `mydumper`, `SELECT INTO OUTFILE`

- Produces SQL files containing data that can regenerate a database
- Easily backup and restore single row, table, or database
- Can separate schema and data
- Can work across versions and forks
- SQL can be parsed with standard UNIX tools
- Restore process automatically replicated
- Long full restore time
- A SQL dump is independent of storage engine, and can be restored to a different storage engine, or used for migration
- Process can be slow and requires locks

### MariaDB Backup

_MariaDB's own backup package_ (`mariabackup`) _is available from MariaDB repositories_

- Supports MariaDB's features and storage engines
- Supports encryption
- Supports incremental backups
- Also available on Windows
- MariaDB Enterprise Backup contains improvements such as reduced locking

### Backing Up Data - MariaDB Backup

#### Full Backups

To use MariaDB Backup you need to create a user on your MariaDB Server with `RELOAD`, `LOCK TABLES` and `REPLICATION CLIENT` privileges.

```sql
MariaDB [(none)]> CREATE USER '<backupuser>'@'localhost' IDENTIFIED BY '<password>';
MariaDB [(none)]> GRANT RELOAD, PROCESS, LOCK TABLES, REPLICATION CLIENT ON *.* TO '<backupuser>'@'localhost';
```

To take a full backup at the OS command-line use:

```
# mariabackup --backup --target-dir <backupdir> --user <backupuser> --password <password>
# mariabackup --prepare --target-dir <backupdir>
```

### Backing Up Data - MariaDB-dump

The standard MariaDB logical dump tool copies schema and data to a SQL text file

To use `mariadb-dump` you need to create a user on your MariaDB Server with `SELECT`, `RELOAD`, `LOCK TABLES`, `REPLICATION CLIENT`, `SHOW VIEW`, `EVENT`, and `TRIGGER` privileges

```sql
CREATE USER 'backupuser'@'localhost' IDENTIFIED BY 'mariadb';

GRANT SELECT, RELOAD, LOCK TABLES, REPLICATION CLIENT, SHOW VIEW, EVENT, TRIGGER ON *.* TO 'backupuser'@'localhost';
```

To take a full backup at the OS command line use

```sh
# mariadb-dump -u backupuser -p --all-databases --single-transaction --flush-logs -r /path/to/full-backup-YYYYMMDD.sql
```

### Planning and Scheduling Backups

Take inventory of databases

- List of databases & tables
- Number of rows
- Frequency of changes
- How active
- Sensitivity of data

Write a backup schedule

Write a verification schedule

Verify and practice restoring from backups

- Backups are useless unless they have been tested
- Recovery procedures should be tested at least once per quarter
- Keep track of restore times during tests

Naming standard for backup files

Frequency — days and times

Location — security and off-site

## Restoring Data

### Restoring Data - MariaDB Backup

#### Full Backups

Working with a full backup taken with MariaDB Backup, you restore the backup into an *empty* data directory

```sh
# mariabackup --copy-back --target-dir <backupdir> --datadir <datadir>
```

Afterwards it might be necessary to set the ownership of the data directory contents

```sh
# chown -R mysql:mysql <datadir>
```

### Restoring Data - MariaDB-dump

Restoring from a logical backup

```sh
# mariadb < /path/to/full-backup-YYYYMMDD.sql
```

Or to avoid interpretation by the shell

```sh
# mariadb
SOURCE /path/to/full-backup-YYYYMMDD.sql
```

You can also use standard linux tools such as `vi` or `grep` to edit or extract the backup

```sh
# grep city worldbackup.sql
CREATE TABLE `world`.`city` ( ...
```

### Verifying Backups

- Backups do not exist until you are certain they can be recovered so prepare a backup immediately
- Some of the reasons for preparing backups are to:
  - Enable recovery during a disaster
  - Migrate to a new server
  - Build a new replica
  - Build a test environment that is cloned from production
- Recovery procedures should be tested on a regular basis, at least once a quarter
- Validate the correctness of your backups
- Provide metrics on recovery time and recovery point
- Test by refreshing data to stage/test or development
- Test different kinds of restores as well
  - Do I know how to restore all of the data?
  - Do I know how to restore just a single table?
  - Do I know how to perform Point-in-Time recovery?
  - Do I know how to restore a replica?
- Restore and check for:
  - Errors
  - Data Size
  - Checksums
  - Resultsets

## Lesson Summary

- Explain key backup principles and their importance, including the importance of verifying and testing backups
- Demonstrate and compare how to create full backups using different backup methods such as `MariaDB Backup` and `mariadb-dump`
- Apply and assess recovery methods to restore data in MariaDB
- Design and implement a backup and recovery plan and schedule

