# SQL Basics

SQL is a query-language devised around elegant searching for the records you want, and joining them into tabular (rows & columns) data. Think of it as a search language for a fancy dynamic spreadsheet.

SQL data is made up of these things:

*  databases (collections of tables)
*  tables (column & rows)
*  columns (typed fields)
*  rows (the actual data)

Within a database, you can refer to tables by name, join multiple tables by different criteria.

## sqlite

SQL is a generic query language that has many variants. [SQLite](http://www.sqlite.org/) is the variant of SQL we are going to use. It's simple, small, fast, meant for local clients, & even included in [some browsers](http://caniuse.com/#feat=sql-storage).

If you use Chrome, you can goto the "Resorces" tab of "Javascript Tools" and see "Web SQL" which will show you what data is stored in SQLite for the current web-domiain.  This can be handy if you just want to quickly see what's going on without running an SQL query.

I made a cute lil console for it  [here](http://konsumer.github.io/learnsql/) that will help you get started.  You can run SQL and see results.

If you prefer to follow along with non-web client, check out [SQLite Browser](http://sqlitebrowser.org/) for a GUI, or [sqlite3](http://www.sqlite.org/cli.html) for a command-line client.


## tables

SQL is type-enforced & has some error-checking. It prevents you from adding records to a table that don't conform to the rules for that table.

You can define a table with a `CREATE` statement.  Here is an example:

```sql
CREATE TABLE IF NOT EXISTS Users(
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  firstName VARCHAR(255) NOT NULL,
  lastName VARCHAR(255) NOT NULL,
  email VARCHAR(255),
  created TIMESTAMP DEFAULT (DATETIME('now','localtime'))
);
```

I will break it down:

*  create a table called `Users` if there isn't one with the same name, already
*  `id` is an auto-increment integer (every time you insert a record, it will give you the next available number, if it's not set.) It's a `PRIMARY KEY` meaning that it's indexed to be the fastest way to get a record, and it's unique (no 2 records can have the same `id`.)
*  `firstName`, `lastName`, & `email` are strings. [SQLite types](https://www.sqlite.org/datatype3.html) are `NULL`, `INTEGER`, `REAL`, `TEXT`, & `BLOB`. `VARCHAR` resolves to `TEXT` and `(255)` is the maximum length. I use `VARCHAR` because some other SQL variants can optimize based on the concept that it's "variable length", and since SQLite resolves to this, I will get better performance on other database-engines, with the same syntax. `NOT NULL` means that you must have a value for this field.
*  `created` is a `TIMESTAMP`, which resolves to a number at the lowest level, but will  output nice dat-stamp formatted text. I set it's default to `DATETIME('now','localtime')` which is whatever time you insert a record. This means that if you leave it blank, it will insert the created time of the record. In many other SQL variants the shorthand for this is just `NOW()`

These are the rules that will validate new records.


### DROP

You can get rid of a table and all of it's rows with the `DROP TABLE` command, like this:

```sql
DROP TABLE Users;
```

You can also do this, to prevent an error if the table doesn't exist:

```sql
DROP TABLE IF EXISTS Users;
```


## CRUD

Create, Read, Update, Delete - this is the basis of all things you might want to do to objects that can be described with SQL. You can do them all by selecting rows using [expressions](https://www.sqlite.org/lang_expr.html).


### Create

Here is how to create a new record:

```sql
INSERT INTO Users (firstName, lastName, email) VALUES ("David", "Konsumer", "konsumer@jetboystudio.com");
```

You'll notice I leave out `id` & `created` fields. This will make them whatever is the next ID & whatever the current time/date is (because the table definition says it should.)

If I try to insert a record that doesn't follow our table field rules above, like  this:

```sql
INSERT INTO Users (email) VALUES ("konsumer@jetboystudio.com");
```

I will get this error:

```
could not execute statement due to a constaint failure (19 constraint failed)
```

This issue is with `firstName` & `lastName` which have `NOT NULL` in the definition, so they must be set to create a record.

### Read

Once you have a record in your table, you'll want to read it. You can do that with `SELECT`, which is a way to choose 1 or more records, based on some criteria. Here is an example:

```sql
SELECT * FROM Users WHERE firstName="David";
```

`*` means "all columns", and `firstName="David"` is the criteria for grabbing records. Instead of `*`, you could say `firstName, lastName`, if you only wanted those fields.

| id | firstName | lastName | email                     | created             |
|----|-----------|----------|---------------------------|---------------------|
| 1  | David     | Konsumer | konsumer@jetboystudio.com | 2014-11-24 00:38:15 |

You can have multiple `WHERE` terms, joined by `OR` or `AND`. Here is an example:

```sql
SELECT email FROM Users WHERE lastName="Cool" OR lastName="Konsumer";
```

You can also search things with wildcards, using the [LIKE clause](http://www.tutorialspoint.com/sqlite/sqlite_like_clause.htm).

```sql
SELECT email FROM Users WHERE lastName LIKE "%ons%";
```

`SELECT` has [other things](https://www.sqlite.org/lang_select.html) you can use. Be sure to look at [SQL statement structure](https://www.sqlite.org/lang_expr.html) to get hints about how to construct expressions.

You can also leave off the `WHERE` clause:

```sql
SELECT * FROM Users;
```

to get all the rows in the table.


### Update

If you already have a record, and  you want to update it, here is how you do it:

```sql
UPDATE Users SET email="david.konsumer@gmail.com" WHERE lastName="Konsumer";
```

The `WHERE` clause can do the same things as `SELECT`'s. It can affect 1 or many records, depending on your `WHERE` clause, just like `SELECT`s.


### Delete

`DELETE` works the same as `SELECT`, but to delete rows. Here is an example:

```sql
DELETE FROM Users WHERE lastName="Konsumer";
```

Be careful. Just like `UPDATE`, this can affect many rows if you make your `WHERE` clause funny.


## JOINS

`JOIN`s are a way to join multiple tables together in a `SELECT`, as if they were a single table. There are [a few types of `JOIN`s in SQLite](https://www.sqlite.org/syntax/join-operator.html). I am going to talk about `INNER JOIN` & `LEFT OUTER JOIN`, as these are what I generally use for things.


### the setup

Let's imagine you have 2 tables, like this for the upcoming examples:

```sql
DROP TABLE IF EXISTS Employees;
DROP TABLE IF EXISTS Departments;

CREATE TABLE Employees(
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name VARCHAR(255) NOT NULL
);

CREATE TABLE Departments(
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  EmployeeID INTEGER NOT NULL,
  dept VARCHAR(255) NOT NULL
);

INSERT INTO Employees (id,name) VALUES (1, "David Konsumer");
INSERT INTO Employees (id,name) VALUES (2, "Cheryl Cool");
INSERT INTO Employees (id,name) VALUES (3, "Christy Cool");
INSERT INTO Employees (id,name) VALUES (4, "Carola Priebke");
INSERT INTO Employees (id,name) VALUES (5, "Cong Tang");
INSERT INTO Employees (id,name) VALUES (6, "Ryoichi Horigoshi");
INSERT INTO Employees (id,name) VALUES (7, "Emma Wallishauser");
INSERT INTO Employees (id,name) VALUES (8, "Rosalie Paternoster");
INSERT INTO Employees (id,name) VALUES (9, "Eugene Brayer");

INSERT INTO Departments (dept,EmployeeID) VALUES ('Engineering', 1);
INSERT INTO Departments (dept,EmployeeID) VALUES ('Customer Relations', 2);
INSERT INTO Departments (dept,EmployeeID) VALUES ('Customer Relations', 3);
INSERT INTO Departments (dept,EmployeeID) VALUES ('Sales', 4);
INSERT INTO Departments (dept,EmployeeID) VALUES ('Sales', 5);
INSERT INTO Departments (dept,EmployeeID) VALUES ('Accounting', 6);
INSERT INTO Departments (dept,EmployeeID) VALUES ('Human Resources', 8);
```


This describes some `Employees` who work in various departments. The department `INSERT`s are adding  employees, by id, to the department & not all employees have a department. You records looks like this:

```sql
SELECT * FROM Employees;
```

| id | name                |
|----|---------------------| 
| 1  | David Konsumer      |
| 2  | Cheryl Cool         |
| 3  | Christy Cool        |
| 4  | Carola Priebke      |
| 5  | Cong Tang           |
| 6  | Ryoichi Horigoshi   |
| 7  | Emma Wallishauser   |
| 8  | Rosalie Paternoster |
| 9  | Eugene Brayer       |

```sql
SELECT * FROM Departments;
```

| id | EmployeeID | dept               |
|----|------------|--------------------|
| 1  | 1          | Engineering        |
| 2  | 2          | Customer Relations |
| 3  | 3          | Customer Relations |
| 4  | 4          | Sales              |
| 5  | 5          | Sales              |
| 6  | 6          | Accounting         |
| 7  | 8          | Human Resources    |

### INNER JOIN

The `INNER JOIN` selects only those records from database tables that have matching values. `INNER` is also the default for `JOIN`.

```sql
SELECT name, dept FROM Employees JOIN Departments ON Employees.id = Departments.EmployeeID;
```

| name                | dept               |
|---------------------|--------------------|
| David Konsumer      | Engineering        |
| Cheryl Cool         | Customer Relations |
| Christy Cool        | Customer Relations |
| Carola Priebke      | Sales              |
| Cong Tang           | Sales              |
| Ryoichi Horigoshi   | Accounting         |
| Rosalie Paternoster | Human Resources    |

This can also be accomplished in many SQL variants with the simpler (in my opinion) syntax:

```sql
SELECT name, dept FROM Employees, Departments WHERE Employees.id = Departments.EmployeeID;
```

### OUTER JOIN

An `OUTER JOIN` does not require each record in the two joined tables to have a matching record. There are three types of outer joins: `LEFT`, `RIGHT`, & `FULL`. SQLite only supports `LEFT`.

```sql
SELECT name, dept FROM Employees LEFT JOIN Departments ON Employees.id = Departments.EmployeeID;
```

| name                | dept               |
|---------------------|--------------------|
| David Konsumer      | Engineering        |
| Cheryl Cool         | Customer Relations |
| Christy Cool        | Customer Relations |
| Carola Priebke      | Sales              |
| Cong Tang           | Sales              |
| Ryoichi Horigoshi   | Accounting         |
| Emma Wallishauser   |                    |
| Rosalie Paternoster | Human Resources    |
| Eugene Brayer       |                    |

### NATURAL

If you have matching fieldnames in 2 tables, you can use `NATURAL` to save some keystrokes (and keep it neater) in both `INNER` & `OUTER`. Like, if `Employees` has `EmployeeID` instead of `id`, you can do this:

```sql
SELECT name, dept FROM Employees NATURAL JOIN Departments;
```


This actually works fine with our structure, as SQLite figures that `Employees.id` connects to `Departments.EmployeeID`, based on the name.

## normalization

This is the process of organizing the fields and tables of a database to minimize redundancy. Normalization usually involves dividing large tables into smaller (and less redundant) tables and defining relationships between them. It's not always the best way to structure your data, it's just a very common pattern. Read more about why you might want denormalized data, [here](http://www.ovaistariq.net/199/databases-normalization-or-denormalization-which-is-the-better-technique/). In the above example, you might notice there are several records with "Sales" & "Customer Relations". Let's re-architect our data-model so that it is normalized and doesn't have lot of wasteful redundant strings. Let's also use `NATURAL` to make our queries simpler by using same-named fields for `JOIN`ing.

```sql
DROP TABLE IF EXISTS Employees;
DROP TABLE IF EXISTS Departments;

CREATE TABLE Employees(
  EmployeeID INTEGER PRIMARY KEY AUTOINCREMENT,
  name VARCHAR(255) NOT NULL
);

CREATE TABLE Departments(
  DepartmentID INTEGER PRIMARY KEY AUTOINCREMENT,
  department VARCHAR(255) NOT NULL
);

CREATE TABLE EmployeeToDepartment(
  EmployeeID INTEGER NOT NULL,
  DepartmentID INTEGER NOT NULL
);

INSERT INTO Employees (EmployeeID,name) VALUES (1, "David Konsumer");
INSERT INTO Employees (EmployeeID,name) VALUES (2, "Cheryl Cool");
INSERT INTO Employees (EmployeeID,name) VALUES (3, "Christy Cool");
INSERT INTO Employees (EmployeeID,name) VALUES (4, "Carola Priebke");
INSERT INTO Employees (EmployeeID,name) VALUES (5, "Cong Tang");
INSERT INTO Employees (EmployeeID,name) VALUES (6, "Ryoichi Horigoshi");
INSERT INTO Employees (EmployeeID,name) VALUES (7, "Emma Wallishauser");
INSERT INTO Employees (EmployeeID,name) VALUES (8, "Rosalie Paternoster");
INSERT INTO Employees (EmployeeID,name) VALUES (9, "Eugene Brayer");

INSERT INTO Departments (DepartmentID, department) VALUES (1, "Engineering");
INSERT INTO Departments (DepartmentID, department) VALUES (2, "Customer Relations");
INSERT INTO Departments (DepartmentID, department) VALUES (3, "Sales");
INSERT INTO Departments (DepartmentID, department) VALUES (4, "Accounting");
INSERT INTO Departments (DepartmentID, department) VALUES (5, "Human Resources");

INSERT INTO EmployeeToDepartment (EmployeeID,DepartmentID) VALUES (1,1);
INSERT INTO EmployeeToDepartment (EmployeeID,DepartmentID) VALUES (2,2);
INSERT INTO EmployeeToDepartment (EmployeeID,DepartmentID) VALUES (3,2);
INSERT INTO EmployeeToDepartment (EmployeeID,DepartmentID) VALUES (4,3);
INSERT INTO EmployeeToDepartment (EmployeeID,DepartmentID) VALUES (5,3);
INSERT INTO EmployeeToDepartment (EmployeeID,DepartmentID) VALUES (6,4);
INSERT INTO EmployeeToDepartment (EmployeeID,DepartmentID) VALUES (8,5);

```

This is the same data-structure as before, but described with 3 tables. Some employees have departments, some are the same department, and some don't have a department. If we want nice text of all the employees with their departments, we would do this:

```sql
SELECT EmployeeID,name,department
FROM Employees
NATURAL JOIN EmployeeToDepartment
NATURAL JOIN Departments
GROUP BY EmployeeID;
```

As you can see, with one more `JOIN`, we get the same rows as `INNER JOIN` above:

| EmployeeID | name                | dept               |
|------------|---------------------|--------------------|
| 1          | David Konsumer      | Engineering        |
| 2          | Cheryl Cool         | Customer Relations |
| 3          | Christy Cool        | Customer Relations |
| 4          | Carola Priebke      | Sales              |
| 5          | Cong Tang           | Sales              |
| 6          | Ryoichi Horigoshi   | Accounting         |
| 8          | Rosalie Paternoster | Human Resources    |

I am using `GROUP BY` to prune out duplicate records caused by the `JOIN`. `INNER JOIN` leaves out any record that doesn't have all the fields.

For getting all the Employeess, regardless of whether or not they have a department, we would use `LEFT JOIN`, which looks like this:

```sql
SELECT EmployeeID,name,department
FROM Employees
NATURAL LEFT JOIN EmployeeToDepartment
NATURAL LEFT JOIN Departments
GROUP BY EmployeeID;
```

| EmployeeID | name                | dept               |
|------------|---------------------|--------------------|
| 1          | David Konsumer      | Engineering        |
| 2          | Cheryl Cool         | Customer Relations |
| 3          | Christy Cool        | Customer Relations |
| 4          | Carola Priebke      | Sales              |
| 5          | Cong Tang           | Sales              |
| 6          | Ryoichi Horigoshi   | Accounting         |
| 7          | Emma Wallishauser   |                    |
| 8          | Rosalie Paternoster | Human Resources    |
| 9          | Eugene Brayer       |                    |

## more reading

Once you have played around with all this stuff, and feel like you understand it, have a look at [this article](http://zetcode.com/db/sqlite/viewstriggerstransactions/) about views, triggers, & transactions and [this article](http://zetcode.com/db/sqlite/sqlitefunctions/) about functions. These are more advanced SQLite tricks that translate well to other SQL variants. This really is just the basics. If you are  excited about doing even more with SQL, check out [the docs](https://www.sqlite.org/lang.html) to find out about all the other stuff you can do.

[NOSQL](http://en.wikipedia.org/wiki/NoSQL) is a newer way of storing data that often is much simpler and faster. For most things I do, I use [mongdb](http://www.mongodb.org/) & [CouchDB](http://couchdb.apache.org/) which both use a [Document storage model](http://en.wikipedia.org/wiki/Document-oriented_database), and are much faster for me to prototype an application with, and generally get the data I need, immediately. Both perform a lot better than their SQL counterparts & scale to many systems easier and more robustly. [CouchDB](http://couchdb.apache.org/) tends to be better at dealing with trees & maps, and [mongdb](http://www.mongodb.org/) is much simpler to query. Both are awesome.

[d3.js](http://d3js.org/) is an amazing library that helps you bring data to life. It's an awesome way to visualize data whether it comes from SQL or NOSQL or somewhere else. If you need a graph of any kind, this library can probably make a really pretty one.
