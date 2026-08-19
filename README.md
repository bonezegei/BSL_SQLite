# BSL_SQLite

**BSL_SQLite** is a SQLite library and lightweight Object-Relational Mapper (ORM) for the **Bonezegei Scripting Language (BSL)**. Built on top of C SQLite bindings, it provides both raw SQL execution (`exec`/`query`) and a simple ORM interface for model-based CRUD operations, dynamic schema creation, pagination, and sorting.

---

## Table of Contents
- [Features](#features)
- [Installation](#installation)
- [Getting Started](#getting-started)
- [Usage Examples](#usage-examples)
  - [1. Raw SQL CRUD & Schema Inspection](#1-raw-sql-crud--schema-inspection)
  - [2. ORM Model CRUD, Sorting & Pagination](#2-orm-model-crud-sorting--pagination)
- [API Reference](#api-reference)
  - [Database Operations](#database-operations)
  - [ORM Model Operations](#orm-model-operations)
- [License & Author](#license--author)

---

## Features
- **Dual Interface:** Choose between raw SQL queries or object-oriented ORM methods.
- **Auto Primary Keys:** ORM models automatically manage `id INTEGER PRIMARY KEY AUTOINCREMENT`.
- **Memory-Efficient Pagination:** Use `.limit()` and `.count()` to fetch records in small batches without high memory overhead.
- **Schema & Table Inspection:** Interrogate database schema and retrieve table names directly.

---

## Installation

Install `BSL_SQLite` using the BSL Package Manager (`bzg`):

```bash
bzg install sqlite
```

---

## Getting Started

Include the SQLite module in your script:

```javascript
include("lib/sqlite.bzg");
```

---

## Usage Examples

### 1. Raw SQL CRUD & Schema Inspection

This example demonstrates traditional SQL execution using raw queries:

```javascript
/*
    BSL - SQLite Raw CRUD Example
    Author: Jofel Batutay (Bonezegei)
*/

include("lib/sqlite.bzg");

// 1. Open Database
var db = sqliteOpen("app.db");
if (db == null) {
    print("Error: Could not open database.");
}

// 2. Create Table
db.exec("CREATE TABLE IF NOT EXISTS Users (id INTEGER PRIMARY KEY AUTOINCREMENT, name VARCHAR(100), email VARCHAR(100));");

// 3. Insert Records (Create)
db.exec("INSERT INTO Users (name, email) VALUES ('Alice Smith', 'alice@example.com');");
db.exec("INSERT INTO Users (name, email) VALUES ('Bob Jones', 'bob@example.com');");

// 4. Schema Inspection
var schema = db.schema("Users");
for (var i = 0; i < sizeof(schema); i = i + 1) {
    var col = schema[i];
    print("Column: " + col.name + " | Type: " + col.type + " | PrimaryKey: " + col.pk);
}

// 5. Query Records (Read)
var users = db.query("SELECT * FROM Users;");
for (var i = 0; i < sizeof(users); i = i + 1) {
    var user = users[i];
    print("ID: " + user.id + " | Name: " + user.name + " | Email: " + user.email);
}

// 6. Update Record
db.exec("UPDATE Users SET email = 'alice.smith@newdomain.com' WHERE name = 'Alice Smith';");

// 7. Delete Record
db.exec("DELETE FROM Users WHERE name = 'Bob Jones';");
```

---

### 2. ORM Model CRUD, Sorting & Pagination

This example uses the built-in ORM to define schemas, execute CRUD operations, filter records, and paginate results:

```javascript
/*
    BSL - SQLite ORM CRUD, Sorting & Pagination
    Author: Jofel Batutay (Bonezegei)
*/

include("lib/sqlite.bzg");

// 1. Open Database & Initialize ORM Model
var db = sqliteOpen("app.db");
var Users = db.model("Users");

// 2. Dynamic Schema Creation (Auto-includes 'id' primary key)
Users.createTable({
    name: "VARCHAR(100)",
    email: "VARCHAR(100)"
});

// 3. Insert Records via ORM
var aliceId = Users.insert({
    name: "Alice Smith",
    email: "alice@example.com"
});

var bobId = Users.insert({
    name: "Bob Jones",
    email: "bob@example.com"
});

var charlieId = Users.insert({
    name: "Charlie Brown",
    email: "charlie@example.com"
});

// 4. Read Single Record (.find())
var alice = Users.find(aliceId);
if (alice) {
    print("Found: " + alice.name + " (" + alice.email + ")");
}

// 5. Memory-Efficient Pagination (.count() & .limit())
var totalRows = Users.count();
var pageSize = 2;

for (var offset = 0; offset < totalRows; offset = offset + pageSize) {
    var page = Users.limit(pageSize, offset);
    for (var j = 0; j < sizeof(page); j = j + 1) {
        print(" -> [ID " + page[j].id + "] " + page[j].name);
    }
}

// 6. Sorting Results (.orderBy())
var ascUsers = Users.orderBy("name", "ASC");
var descUsers = Users.orderBy("id", "DESC", 2, 0); // With Limit and Offset

// 7. Filter Condition (.where())
var filtered = Users.where("name = 'Alice Smith'");

// 8. Update Record via ORM (.update())
Users.update(aliceId, {
    email: "alice.smith@newdomain.com"
});

// 9. Delete Record via ORM (.delete())
Users.delete(bobId);

// 10. Fetch All Remaining Active Users
var finalUsers = Users.all();
```

---

## API Reference

### Database Operations

| Function / Method | Return Value | Description |
| :--- | :--- | :--- |
| `sqliteOpen(filename)` | `Object` / `null` | Opens or creates a SQLite database file and returns a handle. |
| `db.exec(sql_string)` | `int` | Executes non-query SQL statements (`CREATE`, `INSERT`, `UPDATE`, `DELETE`). |
| `db.query(sql_string)` | `Array[Object]` | Executes a `SELECT` statement and returns an array of row objects. |
| `db.schema(table_name)` | `Array[Object]` | Returns metadata for table columns (`name`, `type`, `pk`). |
| `db.tables()` | `Array[Object]` | Returns an array containing all user table names in the database. |
| `db.model(table_name)` | `Model` | Initializes and returns an ORM Model bound to the specified table. |

---

### ORM Model Operations

| Method | Return Value | Description |
| :--- | :--- | :--- |
| `Model.createTable(schema_object)` | `void` | Creates table if not exists with given column definitions and auto `id`. |
| `Model.insert(data_object)` | `int` | Inserts a record and returns the auto-generated primary key ID. |
| `Model.find(id)` | `Object` / `null` | Fetches a single record by its primary key ID. |
| `Model.all()` | `Array[Object]` | Fetches all records from the table. |
| `Model.count()` | `int` | Returns total number of rows in the table. |
| `Model.limit(limit, offset)` | `Array[Object]` | Fetches a paginated subset of records using `LIMIT` and `OFFSET`. |
| `Model.orderBy(col, dir, [limit], [offset])` | `Array[Object]` | Returns records sorted by `col` (`"ASC"` or `"DESC"`). |
| `Model.where(sql_condition)` | `Array[Object]` | Returns records matching custom SQL condition string. |
| `Model.update(id, data_object)` | `void` | Updates specified fields for the record matching `id`. |
| `Model.delete(id)` | `void` | Deletes record matching primary key `id`. |

---

## License & Author

* **Author:** Jofel Batutay ([Bonezegei](https://github.com/bonezegei))
* **Website:** [bonezegei.com](https://bonezegei.com)
