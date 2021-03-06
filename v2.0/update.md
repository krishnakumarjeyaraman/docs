---
title: UPDATE
summary: The UPDATE statement updates rows in a table.
toc: false
---

The `UPDATE` [statement](sql-statements.html) updates rows in a table.

{{site.data.alerts.callout_danger}}If you update a row that contains a column referenced by a <a href="foreign-key.html">foreign key constraint</a> and has an <a href="foreign-key.html#foreign-key-actions-new-in-v2-0"><code>ON UPDATE</code> action</a>, all of the dependent rows will also be updated.{{site.data.alerts.end}}

<div id="toc"></div>

## Required Privileges

The user must have the `SELECT` and `UPDATE` [privileges](privileges.html) on the table.

## Synopsis

{% include sql/{{ page.version.version }}/diagrams/update.html %}

<div markdown="1"></div>

## Parameters

Parameter | Description
----------|------------
`common_table_expr` | See [Common Table Expressions](common-table-expressions.html).
`table_name` | The name of the table that contains the rows you want to update.
`AS table_alias_name` | An alias for the table name. When an alias is provided, it completely hides the actual table name.
`column_name` | The name of the column whose values you want to update.
`a_expr` | The new value you want to use, the [aggregate function](functions-and-operators.html#aggregate-functions) you want to perform, or the [scalar expression](scalar-expressions.html) you want to use.
`DEFAULT` | To fill columns with their [default values](default-value.html), use `DEFAULT VALUES` in place of `a_expr`. To fill a specific column with its default value, leave the value out of the `a_expr` or use `DEFAULT` at the appropriate position.
`column_name` | The name of a column to update.
`select_stmt` | A [selection query](selection-queries.html). Each value must match the [data type](data-types.html) of its column on the left side of `=`.
`WHERE a_expr`| `a_expr` must be a [scalar expression](scalar-expressions.html) that returns Boolean values using columns (e.g. `<column> = <value>`). Update rows that return `TRUE`.<br><br/>**Without a `WHERE` clause in your statement, `UPDATE` updates all rows in the table.**
`sort_clause` | An `ORDER BY` clause. See [Ordering Query Results](query-order.html) for more details.
`limit_clause` | A `LIMIT` clause. See [Limiting Query Results](limit-offset.html) for more details.
`RETURNING target_list` | Return values based on rows updated, where `target_list` can be specific column names from the table, `*` for all columns, or computations using [scalar expressions](scalar-expressions.html). <br><br>To return nothing in the response, not even the number of rows updated, use `RETURNING NOTHING`.

## Examples

### Update a Single Column in a Single Row

~~~ sql
> SELECT * FROM accounts;
~~~
~~~
+----+----------+----------+
| id | balance  | customer |
+----+----------+----------+
|  1 | 10000.50 | Ilya     |
|  2 |   4000.0 | Julian   |
|  3 |   8700.0 | Dario    |
|  4 |   3400.0 | Nitin    |
+----+----------+----------+
(4 rows)
~~~

~~~ sql
> UPDATE accounts SET balance = 5000.0 WHERE id = 2;

> SELECT * FROM accounts;
~~~
~~~
+----+----------+----------+
| id | balance  | customer |
+----+----------+----------+
|  1 | 10000.50 | Ilya     |
|  2 |   5000.0 | Julian   |
|  3 |   8700.0 | Dario    |
|  4 |   3400.0 | Nitin    |
+----+----------+----------+
(4 rows)
~~~

### Update Multiple Columns in a Single Row

~~~ sql
> UPDATE accounts SET (balance, customer) = (9000.0, 'Kelly') WHERE id = 2;

> SELECT * FROM accounts;
~~~
~~~
+----+----------+----------+
| id | balance  | customer |
+----+----------+----------+
|  1 | 10000.50 | Ilya     |
|  2 |   9000.0 | Kelly    |
|  3 |   8700.0 | Dario    |
|  4 |   3400.0 | Nitin    |
+----+----------+----------+
(4 rows)
~~~

~~~ sql
> UPDATE accounts SET balance = 6300.0, customer = 'Stanley' WHERE id = 3;

> SELECT * FROM accounts;
~~~

~~~
+----+----------+----------+
| id | balance  | customer |
+----+----------+----------+
|  1 | 10000.50 | Ilya     |
|  2 |   9000.0 | Kelly    |
|  3 |   6300.0 | Stanley  |
|  4 |   3400.0 | Nitin    |
+----+----------+----------+
(4 rows)
~~~

### Update Using `SELECT` Statement
~~~ sql
> UPDATE accounts SET (balance, customer) =
    (SELECT balance, customer FROM accounts WHERE id = 2)
     WHERE id = 4;

> SELECT * FROM accounts;
~~~

~~~
+----+----------+----------+
| id | balance  | customer |
+----+----------+----------+
|  1 | 10000.50 | Ilya     |
|  2 |   9000.0 | Kelly    |
|  3 |   6300.0 | Stanley  |
|  4 |   9000.0 | Kelly    |
+----+----------+----------+
(4 rows)
~~~

### Update with Default Values

~~~ sql
> UPDATE accounts SET balance = DEFAULT where customer = 'Stanley';

> SELECT * FROM accounts;
~~~
~~~
+----+----------+----------+
| id | balance  | customer |
+----+----------+----------+
|  1 | 10000.50 | Ilya     |
|  2 |   9000.0 | Kelly    |
|  3 | NULL     | Stanley  |
|  4 |   9000.0 | Kelly    |
+----+----------+----------+
(4 rows)
~~~

### Update All Rows

{{site.data.alerts.callout_danger}}If you don't use the <code>WHERE</code> clause to specify the rows to be updated, the values for all rows will be updated.{{site.data.alerts.end}}

~~~ sql
> UPDATE accounts SET balance = 5000.0;

> SELECT * FROM accounts;
~~~
~~~
+----+---------+----------+
| id | balance | customer |
+----+---------+----------+
|  1 |  5000.0 | Ilya     |
|  2 |  5000.0 | Kelly    |
|  3 |  5000.0 | Stanley  |
|  4 |  5000.0 | Kelly    |
+----+---------+----------+
(4 rows)
~~~

### Update and Return Values

In this example, the `RETURNING` clause returns the `id` value of the row updated. The language-specific versions assume that you have installed the relevant [client drivers](install-client-drivers.html).

{{site.data.alerts.callout_success}}This use of <code>RETURNING</code> mirrors the behavior of MySQL's <code>last_insert_id()</code> function.{{site.data.alerts.end}}

{{site.data.alerts.callout_info}}When a driver provides a <code>query()</code> method for statements that return results and an <code>exec()</code> method for statements that don't (e.g., Go), it's likely necessary to use the <code>query()</code> method for <code>UPDATE</code> statements with <code>RETURNING</code>.{{site.data.alerts.end}}

<section class="filters clearfix">
    <button class="filter-button" data-scope="shell">Shell</button>
    <button class="filter-button" data-scope="python">Python</button>
    <button class="filter-button" data-scope="ruby">Ruby</button>
    <button class="filter-button" data-scope="go">Go</button>
    <button class="filter-button" data-scope="js">Node.js</button>
</section>

<section class="filter-content" markdown="1" data-scope="shell">
<p></p>

~~~ sql
> UPDATE accounts SET balance = DEFAULT WHERE id = 1 RETURNING id;
~~~

~~~
+----+
| id |
+----+
|  1 |
+----+
(1 row)
~~~

</section>

<section class="filter-content" markdown="1" data-scope="python">
<p></p>

~~~ python
# Import the driver.
import psycopg2

# Connect to the "bank" database.
conn = psycopg2.connect(
    database='bank',
    user='root',
    host='localhost',
    port=26257
)

# Make each statement commit immediately.
conn.set_session(autocommit=True)

# Open a cursor to perform database operations.
cur = conn.cursor()

# Update a row in the "accounts" table
# and return the "id" value.
cur.execute(
    'UPDATE accounts SET balance = DEFAULT WHERE id = 1 RETURNING id'
)

# Print out the returned value.
rows = cur.fetchall()
print('ID:')
for row in rows:
    print([str(cell) for cell in row])

# Close the database connection.
cur.close()
conn.close()
~~~

The printed value would look like:

~~~
ID:
['1']
~~~

</section>

<section class="filter-content" markdown="1" data-scope="ruby">
<p></p>

~~~ ruby
# Import the driver.
require 'pg'

# Connect to the "bank" database.
conn = PG.connect(
    user: 'root',
    dbname: 'bank',
    host: 'localhost',
    port: 26257
)

# Update a row in the "accounts" table
# and return the "id" value.
conn.exec(
    'UPDATE accounts SET balance = DEFAULT WHERE id = 1 RETURNING id'
) do |res|

# Print out the returned value.
puts "ID:"
    res.each do |row|
        puts row
    end
end

# Close communication with the database.
conn.close()
~~~

The printed value would look like:

~~~
ID:
{"id"=>"1"}
~~~

</section>

<section class="filter-content" markdown="1" data-scope="go">
<p></p>

~~~ go
package main

import (
        "database/sql"
        "fmt"
        "log"

        _ "github.com/lib/pq"
)

func main() {
        //Connect to the "bank" database.
        db, err := sql.Open(
                "postgres",
                "postgresql://root@localhost:26257/bank?sslmode=disable"
        )
        if err != nil {
                log.Fatal("error connecting to the database: ", err)
        }

        // Update a row in the "accounts" table
        // and return the "id" value.
        rows, err := db.Query(
                "UPDATE accounts SET balance = DEFAULT WHERE id = 1 RETURNING id",
        )
        if err != nil {
                log.Fatal(err)
        }

        // Print out the returned value.
        defer rows.Close()
        fmt.Println("ID:")
        for rows.Next() {
                var id int
                if err := rows.Scan(&id); err != nil {
                        log.Fatal(err)
                }
                fmt.Printf("%d\n", id)
        }
}
~~~

The printed value would look like:

~~~
ID:
1
~~~

</section>

<section class="filter-content" markdown="1" data-scope="js">
<p></p>

~~~ js
var async = require('async');

// Require the driver.
var pg = require('pg');

// Connect to the "bank" database.
var config = {
  user: 'root',
  host: 'localhost',
  database: 'bank',
  port: 26257
};

pg.connect(config, function (err, client, done) {
  // Closes communication with the database and exits.
  var finish = function () {
    done();
    process.exit();
  };

  if (err) {
    console.error('could not connect to cockroachdb', err);
    finish();
  }
  async.waterfall([
    function (next) {
      // Update a row in the "accounts" table
      // and return the "id" value.
      client.query(
        `UPDATE accounts SET balance = DEFAULT WHERE id = 1 RETURNING id`,
        next
      );
    }
  ],
  function (err, results) {
    if (err) {
      console.error('error updating and selecting from accounts', err);
      finish();
    }
    // Print out the returned value.
    console.log('ID:');
    results.rows.forEach(function (row) {
      console.log(row);
    });

    finish();
  });
});
~~~

The printed value would like:

~~~
ID:
{ id: '1' }
~~~

</section>

## See Also

- [`DELETE`](delete.html)
- [`INSERT`](insert.html)
- [`UPSERT`](upsert.html)
- [`TRUNCATE`](truncate.html)
- [`ALTER TABLE`](alter-table.html)
- [`DROP TABLE`](drop-table.html)
- [`DROP DATABASE`](drop-database.html)
- [Other SQL Statements](sql-statements.html)
- [Limiting Query Results](limit-offset.html)
