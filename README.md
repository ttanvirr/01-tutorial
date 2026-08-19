# Table of Contents <!-- omit in toc -->

- [1. Chapter 1: Getting Started](#1-chapter-1-getting-started)
  - [1.1. What Is PostgreSQL?](#11-what-is-postgresql)
  - [1.2. Installation](#12-installation)
  - [1.3. Architectural Fundamentals](#13-architectural-fundamentals)
  - [1.4. Creating a Database](#14-creating-a-database)
  - [1.5. Accessing a Database](#15-accessing-a-database)
- [2. Chapter 2: The SQL Language](#2-chapter-2-the-sql-language)

# 1. Chapter 1: Getting Started

## 1.1. What Is PostgreSQL?

`PostgreSQL` is an object-relational database management system (ORDBMS).

It supports a large part of the `SQL` standard and offers many modern features.

## 1.2. Installation

Before you can use PostgreSQL you need to install it.

[Follow this link](https://www.postgresql.org/download/) to install it in your machine.

For Ubuntu run these commands to install it:

```bash
sudo apt update
sudo apt install postgresql
```

Verify:

```bash
psql --version
```

## 1.3. Architectural Fundamentals

PostgreSQL uses a `client/server` model. A PostgreSQL session consists of the following two main cooperating processes (programs):

1. A database server process (called `postgres`)
   - manages the database files,
   - accepts connections to the database from client applications
   - performs database actions such as reading and changing data, on behalf of the clients.

2. The user's client (frontend) application that wants to perform database operations. Examples include -
   - a client could be a text-oriented tool (`psql`),
   - a graphical application,
   - a web server that accesses the database to display web pages,
   - a specialized database maintenance tool.

   Some client applications are supplied with the PostgreSQL distribution; most are developed by users.

### 2.2.1. How client-server communicate <!-- omit in toc -->

The client and the server can be on different hosts. In that case they communicate over a `TCP/IP` network connection. So, the files that can be accessed on a client machine might not be accessible on the database server machine.

The PostgreSQL server can handle multiple concurrent connections from clients. To achieve this it starts (“forks”) a new process for each connection. The supervisor server process is always running, waiting for client connections, whereas client and associated server processes come and go. (All of this is invisible to the user)

## 1.4. Creating a Database

- Create a database with `createdb`:

  `terminal`

  ```bash
  $ createdb mydb
  ```

  Here, `createdb` client program sends a request to your PostgreSQL server.

  If the command finishes with no message, the database was created successfully.

### Errors <!-- omit in toc -->

- If you see a message similar to:

  ```
  createdb: command not found
  ```

  Then either PostgreSQL was not installed at all or your shell's search path was not set to include it. Try calling the command with an absolute path instead:

  ```
  $ /usr/local/pgsql/bin/createdb mydb
  ```

  The path at your site might be different.

- Another response could be this:

  ```
  createdb: error: connection to server on socket "/tmp/.s.PGSQL.5432" failed: FATAL:  role "tanvir" does not exist
  ```

  This will happen if your PostgreSQL user names are different from your operating system user name (run `whoami`); in that case you need to use the `-U` switch to specify your PostgreSQL user name.

- You need a PostgreSQL user with permission to create databases.

  If you don't have permission, you'll get:

  ```
  createdb: error: database creation failed: ERROR:  permission denied to create database
  ```

### Specify a user <!-- omit in toc -->

- You can explicitly specify the PostgreSQL user:

  ```bash
  $ createdb -U <username> mydb
  ```

### Delete database <!-- omit in toc -->

- You can delete a database with:

  ```bash
  $ dropdb mydb
  ```

  > [!CAUTION]
  > **Be careful:**
  > This permanently deletes the database and its data

## 1.5. Accessing a Database

Once you have created a database, you can access it by:

- Running the PostgreSQL interactive terminal program, called `psql`, which allows you to execute `SQL` commands.
- Using a graphical frontend tool like `pgAdmin`.
- Writing a custom application.

Start up `psql`. It can be activated for the `mydb` database by typing the command:

```
$ psql mydb
```

In psql, you will see output:

```
psql (18.6)
Type "help" for help.

mydb=>
```

If you encounter problems starting `psql` then go back to the previous section. The diagnostics of `createdb` and `psql` are similar, and if the former worked the latter should work as well.

The last line printed out by `psql` indicates that it is listening to you and that you can type `SQL` queries. Try out these commands:

> DON'T FORGET SEMICOLON AT THE END OF EACH SQL COMMAND

```
mydb=> SELECT version();
                             version
-------------------------------------------------------------------​-
 PostgreSQL 18.6 on x86_64-pc-linux-gnu, compiled by gcc (Debian 4.9.2-10) 4.9.2, 64-bit
(1 row)

mydb=> SELECT current_date;
  current_date
---------------
 2016-01-07
(1 row)

mydb=> SELECT 2 + 2;
 ?column?
----------
        4
(1 row)
```

### `psql` internal commands <!-- omit in toc -->

The `psql` program has a number of internal commands that are not `SQL` commands. They begin with the backslash character, `\`.

For example, you can get help on the syntax of various PostgreSQL `SQL` commands by typing:

```
mydb=> \h
```

For more internal commands, type `\?` at the `psql` prompt.

> If you are seeing a long output in a pager, press `q` to quit and go back to `psql` prompt.

### Quit `psql` <!-- omit in toc -->

To quit `psql`, type:

```
mydb=> \q
```

[⬆️ Return to Table of contents](#table-of-contents)

# 2. Chapter 2: The SQL Language
