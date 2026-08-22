# Table of Contents <!-- omit in toc -->

- [1. Chapter 1: Getting Started](#1-chapter-1-getting-started)
  - [1.1. What Is PostgreSQL?](#11-what-is-postgresql)
  - [1.2. Installation](#12-installation)
  - [1.3. Architectural Fundamentals](#13-architectural-fundamentals)
  - [1.4. Creating a Database](#14-creating-a-database)
  - [1.5. Accessing a Database](#15-accessing-a-database)
- [2. Chapter 2: The SQL Language](#2-chapter-2-the-sql-language)
  - [2.1. Introduction](#21-introduction)
  - [2.2. Concepts](#22-concepts)
  - [2.3. Creating a New Table](#23-creating-a-new-table)
  - [2.4. Populating a Table with Rows](#24-populating-a-table-with-rows)
  - [2.5. Querying a Table](#25-querying-a-table)
  - [2.6. Joins Between Tables](#26-joins-between-tables)
    - [2.6.1. Outer join](#261-outer-join)
    - [2.6.2. Self join and abbreviation](#262-self-join-and-abbreviation)
  - [2.7. Aggregate Functions](#27-aggregate-functions)
  - [2.8. Updates](#28-updates)
  - [2.9. Deletions](#29-deletions)

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

## 2.1. Introduction

This chapter provides an overview of how to use `SQL` to perform simple operations.

Previously, you have created a database named `mydb`, and have been able to start `psql`.

To start the tutorial, do the following:

```psql
$ psql -s mydb
...
mydb=>
```

psql's `-s` option puts you in single step mode which pauses before sending each statement to the server.

## 2.2. Concepts

PostgreSQL is a _relational database management system (RDBMS)_. That means it is a system for managing data stored in relations. `Relation` is essentially a mathematical term for table.

Each `table` is a named collection of rows. Each `row` of a given table has the same set of named columns, and each `column` is of a specific `data type`. Whereas columns have a fixed order in each row, it is important to remember that `SQL` does not guarantee the order of the rows within the table in any way (although they can be explicitly sorted for display).

Tables are grouped into databases, and a collection of databases managed by a single PostgreSQL server instance constitutes a `database cluster`.

### Summary <!-- omit in toc -->

- database cluster comprises of databases
- database comprises of tables
- table comprises of rows
- row comprises of named columns
- column has specific data type and fixed order in a row
- rows have no guaranteed order unless explicitly sorted

[⬆️ Return to Table of contents](#table-of-contents)

## 2.3. Creating a New Table

You can create a new table by specifying the table name, along with all column names and their types:

`psql shell`

```psql
CREATE TABLE weather (
city            varchar(80),
temp_lo                 int,       -- low temperature
temp_hi                 int,       -- high temperature
prcp                   real,       -- precipitation
date                   date
);
```

You can enter this into `psql` shell using line breaks. The command won't be terminated until the semicolon.

White space (i.e., spaces, tabs, and newlines) can be used freely in `SQL` commands. That means you can type the command aligned differently than above, or even all on one line. Two dashes (`--`) introduce comments. Whatever follows them is ignored up to the end of the line. `SQL` is case-insensitive about key words and identifiers, except when identifiers are double-quoted to preserve the case (not done above).

- `varchar(80)` specifies a data type that can store arbitrary character strings up to 80 characters in length.
- `int` is the normal integer type.
- `real` is a type for storing single precision floating-point numbers.
- `date` should be self-explanatory.

PostgreSQL supports the standard SQL types `int`, `smallint`, `real`, `double precision`, `char(N)`, `varchar(N)`, `date`, `time`, `timestamp`, and `interval`, etc. PostgreSQL can be customized with an arbitrary number of user-defined data types.

The second example will store `cities` and their associated geographical location:

`psql shell`

```psql
CREATE TABLE cities (
    name            varchar(80),
    location        point
);
```

The `point` type is an example of a PostgreSQL-specific data type.

### List tables <!-- omit in toc -->

You can view all tables in `mydb` database:

`psql shell`

```psql
\dt
```

### View table columns <!-- omit in toc -->

You can view the columns of a table:

`psql shell`

```psql
\d <table_name>
```

### Delete a table <!-- omit in toc -->

You can remove a table using the following command:

`psql shell`

```psql
DROP TABLE <table_name>;
```

[⬆️ Return to Table of contents](#table-of-contents)

## 2.4. Populating a Table with Rows

The `INSERT` statement is used to populate a table with rows:

`psql`

```psql
mydb=> INSERT INTO weather VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27');
```

Note that constants that are not simple numeric values usually must be surrounded by single quotes (`'`).

The `point` type requires a coordinate pair as input:

`psql`

```psql
mydb=> INSERT INTO cities VALUES ('San Francisco', '(-194.0, 53.0)');
```

The syntax requires you to remember the order of the columns. An alternative syntax allows you to list the columns explicitly. You can list the columns in a different order if you wish or even omit some columns, e.g., if the 'precipitation' is unknown:

`psql`

```psql
mydb=> INSERT INTO weather (date, city, temp_hi, temp_lo)
mydb=> VALUES ('1994-11-29', 'Hayward', 54, 37);
```

Please enter all the commands shown above so you have some data to work with in the following sections.

You could also have used `COPY` to load large amounts of data from flat-text files. This is usually faster. An example would be:

`psql`

```psql
mydb=> COPY weather FROM '/home/user/weather.txt';
```

> [!NOTE]
> With `COPY`, the source file must be on the machine running the PostgreSQL server because the server reads the file directly. To read a file from the client machine, use `\copy` instead.

The data inserted above into the `weather` table could be inserted from a file containing (values are separated by a `tab` character):

```txt
San Francisco 46 50 0.25 1994-11-27
San Francisco 43 57 0.0 1994-11-29
Hayward 37 54 \N 1994-11-29
```

[⬆️ Return to Table of contents](#table-of-contents)

## 2.5. Querying a Table

An SQL `SELECT` statement is used to _query_ a table (retrieve data from a table). For example, to retrieve all the rows of table `weather`, type:

`psql`

```psql
mydb=> SELECT * FROM weather;
```

Here `*` is a shorthand for “all columns”. This is the shortcut for:

```psql
mydb=> SELECT city, temp_lo, temp_hi, prcp, date FROM weather;
```

The output should be like:

```
     city      | temp_lo | temp_hi | prcp |    date
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
 Hayward       |      37 |      54 |      | 1994-11-29
 Dhaka         |      79 |      92 |  0.1 | 2026-08-18
 Chattogram    |      78 |      89 | 0.35 | 2026-08-18
(4 rows)
```

### Using expressions <!-- omit in toc -->

You can write expressions, not just simple column references. For example, you can do:

```psql
mydb=> SELECT city, (temp_hi+temp_lo)/2 AS temp_avg, date FROM weather;
```

This should give:

```psql
     city      | temp_avg |    date
---------------+----------+------------
 San Francisco |       48 | 1994-11-27
 Hayward       |       45 | 1994-11-29
 Dhaka         |       85 | 2026-08-18
 Chattogram    |       83 | 2026-08-18
(4 rows)
```

Notice how the `AS` clause (optional) is used to relabel the output column.

### Query specific rows <!-- omit in toc -->

A query can be “qualified” by adding a `WHERE` clause that specifies which rows are wanted. The usual Boolean operators (`AND`, `OR`, and `NOT`) are allowed in the qualification. For example, the following retrieves the weather of San Francisco on rainy days:

`psql`

```psql
SELECT * FROM weather
  WHERE city = 'San Francisco' AND prcp > 0.0;
```

Result:

```psql
     city      | temp_lo | temp_hi | prcp |    date
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
(1 row)
```

### Sorting <!-- omit in toc -->

The results of a query can be returned in sorted order:

`psql`

```psql
SELECT * FROM weather
  ORDER BY city, temp_lo;
```

`psql`

```psql
city | temp_lo | temp_hi | prcp | date
---------------+---------+---------+------+------------
Hayward | 37 | 54 | | 1994-11-29
San Francisco | 43 | 57 | 0 | 1994-11-29
San Francisco | 46 | 50 | 0.25 | 1994-11-27
```

### Remove duplicate rows <!-- omit in toc -->

You can request that duplicate rows be removed from the result of a query:

`psql`

```psql
SELECT DISTINCT city
    FROM weather;
```

Output:

```psql
     city
---------------
 Chattogram
 Dhaka
 Hayward
 San Francisco
(4 rows)
```

Here, the result row ordering might vary. You can ensure consistent results by using `DISTINCT` and `ORDER BY` together:

`psql`

```psql
SELECT DISTINCT city
  FROM weather
  ORDER BY city;

```

[⬆️ Return to Table of contents](#table-of-contents)

## 2.6. Joins Between Tables

Queries can access multiple tables at once, or multiple rows (instances) of the same table at one time. This type of queries are called _`join queries`_.

For example, to return all the weather records together with the location of the associated city, the database needs to compare the `city` column of each row of the `weather` table with the `name` column of all rows in the `cities` table. This would be accomplished by the following query:

`psql`

```psql
mydb=> SELECT * FROM weather JOIN cities ON city = name;
```

Output:

```psql
     city      | temp_lo | temp_hi | prcp |    date    |     name      |     location
---------------+---------+---------+------+------------+---------------+-------------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27 | San Francisco | (-194,53)
 Dhaka         |      79 |      92 |  0.1 | 2026-08-18 | Dhaka         | (90.4125,23.8103)
 Dhaka         |      75 |      95 |  0.2 | 2026-08-17 | Dhaka         | (90.4125,23.8103)
(3 rows)
```

Observe two things here:

- There is no result row for the city of Hayward and Chattogram. This is because there is no matching entry in the `cities` table for them. We will see shortly how this can be fixed.

- There are two columns containing the city name. In practice this is undesirable, though, so you will probably want to list the output columns explicitly rather than using `*`:

  ```psql
  SELECT city, temp_lo, temp_hi, prcp, date, location
      FROM weather JOIN cities ON city = name;
  ```

Since the columns all had different names, the parser automatically found which table they belong to. If there were duplicate column names in the two tables you'd need to _qualify_ the column names to show which one you meant, as in:

`psql`

```psql
SELECT weather.city, weather.temp_lo, weather.temp_hi,
       weather.prcp, weather.date, cities.location
    FROM weather JOIN cities ON weather.city = cities.name;
```

It is widely considered good style for this type of queries.

> [!NOTE]
> Join queries we wrote so far can also be written in this form:
>
> ```psql
> SELECT *
>    FROM weather, cities
>    WHERE city = name;
> ```
>
> But this is old style and not recommended.

### 2.6.1. Outer join

Now we will figure out how we can get the Hayward and Chattogram records back in. What we want the query to do is to scan the `weather` table and for each row to find the matching `cities` row(s). If no matching row is found we want some “empty/null values” to be substituted for the `cities` table's columns. This kind of query is called an _`outer join`_. (The joins we have seen so far are _`inner joins`_.) The command looks like this:

`psql`

```psql
SELECT *
    FROM weather LEFT OUTER JOIN cities ON weather.city = cities.name;
```

Output:

```psql
     city      | temp_lo | temp_hi | prcp |    date    |     name      |     location
---------------+---------+---------+------+------------+---------------+-------------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27 | San Francisco | (-194,53)
 Hayward       |      37 |      54 |      | 1994-11-29 |               |
 Dhaka         |      79 |      92 |  0.1 | 2026-08-18 | Dhaka         | (90.4125,23.8103)
 Chattogram    |      78 |      89 | 0.35 | 2026-08-18 |               |
 Dhaka         |      75 |      95 |  0.2 | 2026-08-17 | Dhaka         | (90.4125,23.8103)
(5 rows)
```

This query is actually called a _`left outer join`_ because the table mentioned on the left of the join operator will have each of its rows in the output, whereas the table on the right will only have those rows output that match some row of the left table. When outputting a left-table row for which there is no right-table match, empty (null) values are substituted for the right-table columns.

**Exercise:** There are also _`right outer joins`_ and _`full outer joins`_. Try to find out what those do.

### 2.6.2. Self join and abbreviation

We can also join a table against itself. This is called a _`self join`_. As an example, suppose we wish to find all the weather records that are in the temperature range of other weather records. So we need to compare the `temp_lo` and `temp_hi` columns of each weather row to the `temp_lo` and `temp_hi` columns of all other weather rows. We can do this with the following query:

`psql`

```psql
SELECT w1.city, w1.temp_lo AS low, w1.temp_hi AS high,
       w2.city, w2.temp_lo AS low, w2.temp_hi AS high
    FROM weather w1 JOIN weather w2
        ON w1.temp_lo < w2.temp_lo AND w1.temp_hi > w2.temp_hi;
```

Output:

```psql
  city   | low | high |     city      | low | high
---------+-----+------+---------------+-----+------
 Hayward |  37 |   54 | San Francisco |  46 |   50
 Dhaka   |  75 |   95 | Dhaka         |  79 |   92
 Dhaka   |  75 |   95 | Chattogram    |  78 |   89
(3 rows)
```

Here we have relabeled the `weather` table as `w1` and `w2` to be able to distinguish the left and right side of the join. You can also use these kinds of aliases in other queries. e.g.:

`psql`

```psql
SELECT *
  FROM weather w JOIN cities c ON w.city = c.name;
```

You will encounter this style of abbreviating quite frequently.

[⬆️ Return to Table of contents](#table-of-contents)

## 2.7. Aggregate Functions

An _`aggregate function`_ computes a single result from multiple input rows. For example, there are aggregates to compute the `count`, `sum`, `avg` (average), `max` (maximum) and `min` (minimum) over a set of rows.

As an example, we can find the highest low-temperature reading within `weather` table with:

`psql`

```psql
SELECT max(temp_lo) FROM weather;
```

Output:

```psql
 max
-----
  79
(1 row)
```

If we want to know what city (or cities) that reading occurred in, we can accomplish this by using a _subquery_ with `WHERE` clause:

`psql`

```psql
SELECT city FROM weather
    WHERE temp_lo = (SELECT max(temp_lo) FROM weather);
```

Output:

```psql
 city
-------
 Dhaka
(1 row)
```

Aggregates are also very useful in combination with `GROUP BY` clauses. For example, we can get the number of readings (cities) and the maximum low temperature observed in each city with:

`psql`

```psql
SELECT city, count(*), max(temp_lo)
    FROM weather
    GROUP BY city;
```

Output:

```psql
     city      | count | max
---------------+-------+-----
 Chattogram    |     1 |  78
 Dhaka         |     2 |  79
 Hayward       |     1 |  37
 San Francisco |     1 |  46
(4 rows)
```

`GROUP BY city` gives us one output row per city.

We can filter these grouped rows using `HAVING`:

`psql`

```psql
SELECT city, count(*), max(temp_lo)
    FROM weather
    GROUP BY city
    HAVING max(temp_lo) < 40;
```

Output:

```psql
  city   | count | max
---------+-------+-----
 Hayward |     1 |  37
(1 row)
```

which gives us the same results for only the cities that have max `temp_lo` values below 40.

Finally, if we only care about cities whose names begin with “D”, we might do:

`psql`

```psql
SELECT city, count(*), max(temp_lo)
    FROM weather
    WHERE city LIKE 'D%'
    GROUP BY city;
```

Output:

```psql
 city  | count | max
-------+-------+-----
 Dhaka |     2 |  79
(1 row)
```

The `LIKE` operator does pattern matching and is explained [here](https://tinyurl.com/4suwnnj6).

It is important to understand the interaction between aggregates and SQL's `WHERE` and `HAVING` clauses (both used for filtering).

- `WHERE` selects input rows before groups and aggregates are computed. Thus, the `WHERE` clause must not contain aggregate functions;
- `HAVING` selects group rows after groups and aggregates are computed and the `HAVING` clause always contains aggregate functions.

Another way to select the rows that go into an aggregate computation is to use `FILTER`, which is a per-aggregate option:

`psql`

```psql
SELECT city, count(*) FILTER (WHERE temp_lo < 45), max(temp_lo)
    FROM weather
    GROUP BY city;
```

Output:

```psql
     city      | count | max
---------------+-------+-----
 Chattogram    |     0 |  78
 Dhaka         |     0 |  79
 Hayward       |     1 |  37
 San Francisco |     0 |  46
(4 rows)
```

`FILTER` is much like `WHERE`, except that it removes rows only from the input of the particular aggregate function that it is attached to. Here, the `count` aggregate counts only rows with `temp_lo` below 45; but the `max` aggregate is still applied to all rows, so it still finds the reading of 46.

[⬆️ Return to Table of contents](#table-of-contents)

## 2.8. Updates

You can update existing rows using the `UPDATE` command. Suppose you discover the temperature readings are all off by 2 degrees after November 28. You can correct the data as follows:

`psql`

```psql
UPDATE weather
    SET temp_hi = temp_hi - 2,  temp_lo = temp_lo - 2
    WHERE date > '1994-11-28';
```

Look at the new state of the data:

`psql`

```psql
SELECT * FROM weather;
```

Output:

```
     city      | temp_lo | temp_hi | prcp |    date
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
 Hayward       |      35 |      52 |      | 1994-11-29
 Dhaka         |      77 |      90 |  0.1 | 2026-08-18
 Chattogram    |      76 |      87 | 0.35 | 2026-08-18
 Dhaka         |      73 |      93 |  0.2 | 2026-08-17
(5 rows)
```

[⬆️ Return to Table of contents](#table-of-contents)

## 2.9. Deletions

Rows can be removed from a table using the `DELETE` command. Suppose you are no longer interested in the weather of Hayward. Then you can do the following:

`psql`

```psql
mydb=> DELETE FROM weather WHERE city = 'Hayward';
```

All weather records belonging to Hayward are removed.

`psql`

```psql
SELECT * FROM weather;
```

Output:

```psql
     city      | temp_lo | temp_hi | prcp |    date
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
 Dhaka         |      77 |      90 |  0.1 | 2026-08-18
 Chattogram    |      76 |      87 | 0.35 | 2026-08-18
 Dhaka         |      73 |      93 |  0.2 | 2026-08-17
(4 rows)
```

> [!CAUTION]
> One should be wary of statements of the form
>
> `psql`
>
> ```psql
> DELETE FROM tablename;
> ```
>
> Without a qualification, `DELETE` will remove all rows from the given table, leaving it empty. The system will not request confirmation before doing this!

[⬆️ Return to Table of contents](#table-of-contents)
