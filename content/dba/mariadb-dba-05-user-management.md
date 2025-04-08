---
title: "Lesson 5: User Management"
url: "/dba/user-management/"
date: 2025-04-07
description: >
categories: []
tags: []
weight: 5
toc: true
---

# User Management

## Learning Objectives

- Demonstrate how to create and manage users in MariaDB
- Explain how to limit resource usage per user in MariaDB
- Show how to assign and revoke privileges to users in MariaDB

## Authentication Methods and User Accounts

### Authentication Methods

- `DEFAULT`
- `ed25519`
- `NAMED PIPE`
- `UNIX SOCKET`
- `GSSAPI`
- `PAM`

### Default Authentication Method

- User ID / Passwords stored within database (`mysql.global_priv`)
- Passwords are encrypted (`PASSWORD()` function)
  - Use `SET PASSWORD=PASSWORD('new_password')` statement for password changes
- Easy for DBAs to control and manage users, roles and policies
- Password strength checking set by DBA
- Not tied to LDAP/AD/Operating System users

### ed25519

- Uses Elliptic Curve Digital Signature Algorithm (ECDSA)
- More secure than default SHA-1 password hashing
- Installed via a plugin - `INSTALL SONAME 'auth_ed25519'`
- `CREATE USER username@hostname IDENTIFIED VIA ed25519 USING PASSWORD('secret');`

### Named Pipe

- Used with Windows named pipe impersonation
- Retrieves username from Windows with `GetUserName()` function
- Password is not required
- Installed via plugin
- `CREATE USER username@hostname IDENTIFIED VIA named_pipe;`

### Unix Socket

- Uses operating system credentials to authenticate
- Password is not required
- Plugin installed by default
- `CREATE USER username@hostname IDENTIFIED VIA unix_socket;`

### MariaDB User Accounts

- Authentication is based on user, host and password except with specific authentication plugins for example Unix socket, GSSAPI and PAM

- Privileges are based on user and host combined, and the user's current role

- Privileges can be granted globally, at the database level, at the table level or at the column level

### Authentication Tables and Views

Internal credentials and global privileges are stored as JSON `mysql.global_priv` table

This allows for further flexibility and avoids frequent changes to the table structure

The old `mysql.user` table is a view on `mysql.global_priv`

```sql
MariaDB [(none)]> DESCRIBE mysql.global_priv;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| Host  | char(60)    | NO   | PRI |         |       |
| User  | char(80)    | NO   | PRI |         |       |
| Priv  | longtext    | NO   |     | '{}'    |       |
+-------+-------------+------+-----+---------+-------+
3 rows in set (0.001 sec)
```

### Creating Users

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

Append the following `tls_option` to the `CREATE USER` statement to specify the requirement and details of secure SSL connections per user

```sql
CREATE USER 'foo'@'test'
REQUIRE ISSUER 'foo_issuer'
SUBJECT 'foo_subject'
CIPHER 'text';
```

### Checking Users

The `SHOW CREATE USER` statement is a useful way to see the command required to create a user for auditing or the creation of similar accounts

```sql
SHOW CREATE USER 'user'@'host'\G
```

### Managing Users

Changes made with the account management statements are automatically activated

#### Change a user's username and host

`RENAME USER` statement

```sql
RENAME USER 'user1'@'host1' TO 'user2'@'host2';
```

#### Change a user's password

`SET PASSWORD` statement

```sql
SET PASSWORD FOR 'user'@'host' = PASSWORD('mariadb2');
```

#### The `ALTER USER` statement allows for easy modification of existing user accounts

Changing a user's password can be rewritten to use the `ALTER USER` statement
sql
ALTER USER CURRENT_USER() IDENTIFIED BY 'mariadb2';
```

### Managing Users

`ALTER USER` statement to limit the number of simultaneous connections

```sql
ALTER USER 'user'@'host' WITH MAX_USER_CONNECTIONS 10;
```

The `ALTER USER` statement can also be used to set certain TLS-related restrictions

```sql
ALTER USER 'user2'@'host2'
REQUIRE SUBJECT '/CN=alice/O=My Dom, Inc./C=US/ST=Oregon/L=Portland'
AND ISSUER '/C=IT/ST=Somewhere/L=City/O=Some Company/CN=Peter Parker/emailAddress=p.parker@marvel.com'
AND CIPHER 'TLSv1.2';
```

### Assigning Privileges

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

### Available Privileges (Show Privileges)

#### Basic Privileges
- Usage
- Select
- Insert
- Update
- Delete
- Show Databases

#### Customizing Privileges
- Create Routine
- Alter Routine
- Execute
- Event
- Trigger

#### Administrator Privileges
- All [Privileges]
- Super
- Create User
- Grant Option
- Process
- File
- Reload
- Shutdown
- Connection Admin
- Federated Admin
- Read only Admin
- Set User

#### Developer Privileges
- Create
- Alter
- Index
- Drop

#### Special Privileges
- Create Temporary Tables
- Create View
- Show View
- Lock Tables

#### Replication Privileges
- Binary log Monitor
- Replication Replica
- Binary log Admin
- Binary log Replay
- Replication master admin
- Replication slave admin

### Checking and Revoking Privileges

The `REVOKE` statement removes privileges but does not remove the user account

To remove a user account, the `DROP USER` statement must be used

Query the user table in the mysql database to check the privileges of user accounts

```sql
SELECT user,host from mysql.user;
```

The `SHOW GRANTS` statement can be used to check a single user account's privileges

```sql
SHOW GRANTS FOR 'user'@'host';
```

Revoke privileges from a user with the `REVOKE` statement by either listing the privileges to revoke or using `ALL PRIVILEGES`

```sql
REVOKE SELECT ON dbname.tablename FROM 'user'@'host';
REVOKE ALL PRIVILEGES ON dbname.tablename FROM 'user'@'host';
```

Confirm that the revoke was successful with `SHOW GRANTS`

```sql
SHOW GRANTS FOR 'user'@'host';
```

## Restricting Users

User accounts can have a limit set on resource use

```sql
GRANT SELECT ON dbname.tablename TO 'user'@'host' 
WITH MAX_QUERIES_PER_HOUR 20 
     MAX_CONNECTIONS_PER_HOUR 10 
     MAX_USER_CONNECTIONS 2 
     MAX_UPDATES_PER_HOUR 5;
```

Resource limit usage is kept in the `mysql` database

Resource limit counters reset then the server starts/restarts or when `FLUSH USER_RESOURCES` is executed

```sql
FLUSH USER_RESOURCES;
```

### Removing Users

Delete users in MariaDB with the `DROP USER` statement

```sql
DROP USER 'user'@'host';
```

Check for leftover user accounts which still allow access from other hosts

```sql
SELECT User,Host from mysql.user WHERE User = 'user';
```

## Lesson Summary

- Demonstrate how to create and manage users in MariaDB
- Explain how to limit resource usage per user in MariaDB
- Show how to assign and revoke privileges to users in MariaDB

