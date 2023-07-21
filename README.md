A lightweight, expressive database query builder for WordPress which can be referred to as a Database Abstraction Layer. WP Fluent uses the same **wpdb** instance and takes care of query sanitization, table prefixing and many other things with a unified API.

It has some advanced features like:

 - Query Events
 - Nested Criteria
 - Sub Queries
 - Nested Queries

The syntax is quite similar to Laravel's query builder.

**Notice:** The global function to access the query builder instance has been changed. You must replace **wpFluent()** with **wpFluentDB()** to work properly.

## Example
```PHP
// You can use the global wpFluentDB() function
$user = wpFluentDB()->table('users')->find(1);

// Or, create a connection using $wpdb, only once.
global $wpdb;
new \WpFluent\Connection($wpdb, ['prefix' => $wpdb->prefix], 'DB');
```

**Simple Query:**

The query below returns the row where id = 3, `null` if no rows.
```PHP
$user = DB::table('users')->find(3);
```

**Full Queries:**

```PHP
$query = DB::table('users')->where('display_name', 'LIKE', '%admin%');

// Get result
$query->get();
```

**Query Events:**

After the code below, every time a select query occurs on `posts` table,
it will add this where criteria, so drafted posts don't surface.

```PHP
DB::registerEvent('before-select', 'posts', function ($qb) {
    $qb->where('psot_status', '!=', 'draft');
});
```

There are so many advanced options documented below. Sold? Let's install.

## Full Usage API

### Table of Contents

 - [Connection](#connection)
    - [Alias](#alias)
 - [Query](#query)
 - [**Select**](#select)
    - [Get Easily](#get-easily)
    - [Multiple Selects](#multiple-selects)
    - [Select Distinct](#select-distinct)
    - [Get All](#get-all)
    - [Get First Row](#get-first-row)
    - [Get Rows Count](#get-rows-count)
 - [**Where**](#where)
    - [Where In](#where-in)
    - [Where Between](#where-between)
    - [Where Null](#where-null)
    - [Grouped Where](#grouped-where)
 - [Group By and Order By](#group-by-and-order-by)
 - [Having](#having)
 - [Limit and Offset](#limit-and-offset)
 - [Join](#join)
    - [Multiple Join Criteria](#multiple-join-criteria)
 - [Raw Query](#raw-query)
    - [Raw Expressions](#raw-expressions)
 - [**Insert**](#insert)
    - [Batch Insert](#batch-insert)
    - [Insert with ON DUPLICATE KEY statement](#insert-with-on-duplicate-key-statement)
 - [**Update**](#update)
 - [**Delete**](#delete)
 - [Transactions](#transactions)
 - [Get Built Query](#get-built-query)
 - [Sub Queries and Nested Queries](#sub-queries-and-nested-queries)
 - [Get PDO Instance](#get-pdo-instance)
 - [Fetch results as objects of specified class](#fetch-results-as-objects-of-specified-class)
 - [Query Events](#query-events)
    - [Available Events](#available-events)
    - [Registering Events](#registering-events)
    - [Removing Events](#removing-events)
    - [Some Use Cases](#some-use-cases)
    - [Notes](#notes)

___


## Connection
WP Fluent supports multiple database connections but you can use alias 
for only one connection at a time. Just pass the global wpdb and
necessary configurations during connection.

```PHP
// Get the global wpdb instance.
global $wpdb;

new \WpFluent\Connection($wpdb, ['prefix' => $wpdb->prefix], 'DB');

// Run query
$query = DB::table('my_table')->where('name', '=', 'admin');

// Or, simply use the global wpFluentDB() function.
// It handles all the ncessary initial setup.
wpFluentDB()->table('my_table')->where('name', '=', 'admin');
```


### Alias
When you create a connection:
```PHP
new \WpFluent\Connection($wpdb, ['prefix' => $wpdb->prefix], 'MyAlias');
```
`MyAlias` is the name for the class alias you want to use e.g. `MyAlias::table(...)`
 
You can use any name (with Namespace also, `MyNamespace\\MyClass`) you like or
you may skip it if you don't need an alias. Alias gives you the ability
to easily access the QueryBuilder class across your application.

When not using an alias you can instantiate the QueryBuilder handler 
separately, helpful for Dependency Injection and Testing.

```PHP
$connection = new \WpFluent\Connection($wpdb, ['prefix' => $wpdb->prefix]);

$db = new \WpFluent\QueryBuilder\QueryBuilderHandler($connection);

$query = $db->table('my_table')->where('name', '=', 'admin');

var_dump($query->get());
```


## Query
You **must** use `table()` method before every query, except raw `query()`.
To select from multiple tables just pass an array.
```PHP
DB::table(array('mytable1', 'mytable2'));
```


### Get Easily
The query below returns the first row where id = 3, `null` if no rows.
```PHP
$row = DB::table('my_table')->find(3);
```
Access your row like, `echo $row->name`. If your field name is not `id` then 
pass the field name as second parameter `DB::table('my_table')->find(3, 'person_id');`

The query below returns all the rows where `name = 'Frost'`, `null` if no rows.

```PHP
$result = DB::table('my_table')->findAll('name', 'admin');
```


### Select
```PHP
$query = DB::table('my_table')->select('*');
```


#### Multiple Selects
```PHP
->select(array('mytable.myfield1', 'mytable.myfield2', 'another_table.myfield3'));
```

Using select method multiple times `select('a')->select('b')` will also select `a` 
and `b`. It can be useful if you want to do conditional selects (within a PHP `if`).


#### Select Distinct
```PHP
->selectDistinct(array('mytable.myfield1', 'mytable.myfield2'));
```


#### Get All
Return an array.
```PHP
$query = DB::table('my_table')->where('name', '=', 'admin');
$result = $query->get();
```
You can loop through it like:
```PHP
foreach ($result as $row) {
    echo $row->name;
}
```

#### Get First Row
```PHP
$query = DB::table('my_table')->where('name', '=', 'admin');
$row = $query->first();
```
Returns the first row, or `null` if there is no record. Using this method you can also
make sure if a record exists. Access it like `$row->name`


#### Get Rows Count
```PHP
$query = DB::table('my_table')->where('name', '=', 'admin');
$query->count();
```

### Where
Basic syntax is `(fieldname, operator, value)`, if you give two parameters then `=`
operator is assumed. So `where('name', 'admin')` and `where('name', '=', 'admin')`
are the same.

```PHP
DB::table('my_table')
    ->where('name', '=', 'admin')
    ->whereNot('age', '>', 25)
    ->orWhere('type', '=', 'admin')
    ->orWhereNot('description', 'LIKE', '%query%');
```


#### Where In
```PHP
DB::table('my_table')
    ->whereIn('name', array('rabindranath', 'najrul'))
    ->orWhereIn('name', array('homer', 'frost'));

DB::table('my_table')
    ->whereNotIn('name', array('homer', 'frost'))
    ->orWhereNotIn('name', array('rabindranath', 'najrul'));
```

#### Where Between
```PHP
DB::table('my_table')
    ->whereBetween('id', 10, 100)
    ->orWhereBetween('status', 5, 8);
```

#### Where Null
```PHP
DB::table('my_table')
    ->whereNull('modified')
    ->orWhereNull('field2')
    ->whereNotNull('field3')
    ->orWhereNotNull('field4');
```

#### Grouped Where
Sometimes queries get complex, where you need grouped criteria, for example
`WHERE age = 10 and (name like '%frost%' or description LIKE '%najrul%')`

WP Fluent allows you to do so, you can nest as many closures as you need, like below.
```PHP
DB::table('my_table')
    ->where('my_table.age', 10)
    ->where(function ($q) {
        $q->where('name', 'LIKE', '%najrul%');
        
        // You can provide a closure on these wheres too, to nest further.
        $q->orWhere('description', 'LIKE', '%frost%');
    });
```

### Group By and Order By
```PHP
$query = DB::table('my_table')->groupBy('age')->orderBy('created_at', 'ASC');
```

#### Multiple Group By
```PHP
->groupBy(array('mytable.myfield1', 'mytable.myfield2', 'another_table.myfield3'));

->orderBy(array('mytable.myfield1', 'mytable.myfield2', 'another_table.myfield3'));
```

Using `groupBy()` or `orderBy()` methods multiple times `groupBy('a')->groupBy('b')`
will also group by first `a` and than `b`. Can be useful if you want to do conditional
grouping (within a PHP `if`). Same applies to `orderBy()`.

### Having
```PHP
->having('total_count', '>', 2)
->orHaving('type', '=', 'admin');
```

### Limit and Offset
```PHP
->limit(30);

->offset(10);
```

### Join
```PHP
DB::table('my_table')
    ->join('another_table', 'another_table.person_id', '=', 'my_table.id')

```

Available methods,

 - join() or innerJoin
 - leftJoin()
 - rightJoin()

If you need `FULL OUTER` join or any other join, just pass it as 5th parameter to
`join` method.
```PHP
->join('another_table', 'another_table.person_id', '=', 'my_table.id', 'FULL OUTER')
```

#### Multiple Join Criteria
If you need more than one criterion to join a table then pass a closure as second
parameter.

```PHP
->join('another_table', function ($table) {
    $table->on('another_table.person_id', '=', 'my_table.id');
    $table->on('another_table.person_id2', '=', 'my_table.id2');
    $table->orOn('another_table.age', '>', DB::raw(1));
})
```

### Raw Query
You can always use raw queries if you need,
```PHP
$query = DB::query('select * from cb_my_table where age = 12');

var_dump($query->get());
```

You can also pass your bindings
```PHP
DB::query('select * from cb_my_table where age = ? and name = ?', array(10, 'najrul'));
```

#### Raw Expressions

When you wrap an expression with `raw()` method, WP Fluent doesn't try to sanitize these.
```PHP
DB::table('my_table')
    ->select(DB::raw('count(cb_my_table.id) as tot'))
    ->where('value', '=', 'Frost')
    ->where(DB::raw('DATE(?)', 'now'))
```


___
**NOTE:** Queries that run through `query()` method are not sanitized until you pass
all values through bindings. Queries that run through `raw()` method are not sanitized
either, you have to do it yourself. And of course these don't add table prefix too,
but you can use the `addTablePrefix()` method.

### Insert
```PHP
$data = [
    'name'        => 'Najrul',
    'description' => 'Famous Bengali poet.'
];

$insertId = DB::table('my_table')->insert($data);
```

`insert()` method returns the insert id.

#### Batch Insert
```PHP
$data = array(
    [
        'name'        => 'Najrul',
        'description' => 'Famous Bengali poet.'
    ],
    [
        'name'        => 'Rabindranath',
        'description' => 'Nobel winning Bengali poet.'
    ],
);

$insertIds = DB::table('my_table')->insert($data);
```

In case of batch insert, it will return an array of insert ids.

#### Insert with ON DUPLICATE KEY statement
```PHP
$data = [
    'name'    => 'Najrul',
    'counter' => 1
];

$dataUpdate = [
    'name'    => 'Najrul',
    'counter' => 2
];

$insertId = DB::table('my_table')->onDuplicateKeyUpdate($dataUpdate)->insert($data);
```

### Update
```PHP
$data = [
    'name'        => 'Najrul',
    'description' => 'Famous Bengali poet.'
];

DB::table('my_table')->where('id', 5)->update($data);
```

It will update the `name` field to `Najrul` and `description` field to
`Famous Bengali poet.` where `id` = `5`.

### Delete
```PHP
DB::table('my_table')->where('id', '>', 5)->delete();
```
It will delete all the rows where `id` is greater than `5`.

### Transactions

WP Fluent has the ability to run database "transactions", in which all database
changes are not saved until committed. That way, if something goes wrong or
differently then you intend, the database changes are not saved and no changes
are made.

Here's a basic transaction:

```PHP
DB::transaction(function ($qb) {
    $qb->table('my_table')->insert([
        'name' => 'Test',
        'url'  => 'example.com'
    ]);

    $qb->table('my_table')->insert([
        'name' => 'Test2',
        'url'  => 'example.com'
    ]);
});
```

If this were to cause any errors (such as a duplicate name or some other such
error), neither data set would show up in the database. If not, the changes would
be successfully saved.

If you wish to manually commit or rollback your changes, you can use the
`commit()` and `rollback()` methods accordingly:

```PHP
DB::transaction(function ($qb) {
    $qb->table('my_table')->insert(array(/* data... */));

    $qb->commit(); // to commit the changes (data would be saved)
    $qb->rollback(); // to rollback the changes (data would be rejected)
});
```

### Get Built Query
Sometimes you may need to get the query string, its possible.
```PHP
$query = DB::table('my_table')->where('id', '=', 3);
$queryObj = $query->getQuery();
```
`getQuery()` will return a query object, from this you can get sql, bindings or raw sql.


```PHP
$queryObj->getSql();
// Returns: SELECT * FROM my_table where `id` = ?
```
```PHP
$queryObj->getBindings();
// Returns: array(3)
```

```PHP
$queryObj->getRawSql();
// Returns: SELECT * FROM my_table where `id` = 3
```

### Sub Queries and Nested Queries
Rarely but you may need to run sub queries or nested queries. WP Fluent is powerful 
enough to do this for you. You can create different query objects and use the
`DB::subQuery()` method.

```PHP
$subQuery = DB::table('person_details')->select('details')->where('person_id', '=', 3);

$query = DB::table('my_table')
            ->select('my_table.*')
            ->select(DB::subQuery($subQuery, 'table_alias1'));

$nestedQuery = DB::table(DB::subQuery($query, 'table_alias2'))->select('*');
$nestedQuery->get();
```

This will produce a query like this:

    SELECT * FROM (SELECT `cb_my_table`.*, (SELECT `details` FROM `cb_person_details` WHERE `person_id` = 3) as table_alias1 FROM `cb_my_table`) as table_alias2

**NOTE:** WP Fluent doesn't use bindings for sub queries and nested queries.

### Get wpdb Instance
If you need to get the wpdb instance you can do so.

```PHP
DB::db();
```


### Query Events
WP Fluent comes with powerful query events to supercharge your application. These events
are like database triggers, you can perform some actions when an event occurs. For
example you can hook `after-delete` event of a table and delete related data from
another table.

#### Available Events

 - before-select
 - after-select
 - before-insert
 - after-insert
 - before-update
 - after-update
 - before-delete
 - after-delete

#### Registering Events

```PHP
DB::registerEvent('before-select', 'users', function ($qb) {
    $qb->where('status', '!=', 'banned');
});
```
Now every time a select query occurs on `users` table, it will add this where criteria,
so banned users don't get access.

The syntax is `registerEvent('event type', 'table name', action in a closure)`.

If you want the event to be performed when **any table is being queried**, provide 
`':any'` as table name.

**Other examples:**

After inserting data into `my_table`, details will be inserted into another table
```PHP
DB::registerEvent('after-insert', 'my_table', function ($queryBuilder, $insertId) {
    $data = array('person_id' => $insertId, 'details' => 'Meh', 'age' => 5);
    
    $queryBuilder->table('person_details')->insert($data);
});
```

Whenever data is inserted into `person_details` table, set the timestamp field 
`created_at`, so we don't have to specify it everywhere:
```PHP
DB::registerEvent('after-insert', 'person_details', function ($queryBuilder, $insertId) {
    $queryBuilder->table('person_details')
                 ->where('id', $insertId)
                 ->update([
                     'created_at' => date('Y-m-d H:i:s')
                 ]);
});
```

After deleting from `my_table` delete the relations:
```PHP
DB::registerEvent('after-delete', 'my_table', function ($queryBuilder, $queryObject) {
    $bindings = $queryObject->getBindings();
    
    $queryBuilder->table('person_details')->where('person_id', $binding[0])->delete();
});
```


WP Fluent passes the current instance of query builder as first parameter of your
closure so you can build queries with this object, you can do anything like usual 
query builder (`DB`).

If something other than `null` is returned from the `before-*` query handler, the value
will be result of execution and DB will not be actually queried (and thus, corresponding
`after-*` handler will not be called either).

Only on `after-*` events you get three parameters: **first** is the query builder, 
**third** is the execution time as float and **the second** varies:

 - On `after-select` you get the `results` obtained from `select`.
 - On `after-insert` you get the insert id (or array of ids in case of batch insert)
 - On `after-delete` you get the [query object](#get-built-query)
 (same as what you get from `getQuery()`), from it you can get SQL and Bindings.
 - On `after-update` you get the [query object](#get-built-query) like `after-delete`.

#### Removing Events
```PHP
DB::removeEvent('event-name', 'table-name');
```

#### Some Use Cases

Here are some cases where Query Events can be extremely helpful:

 - Restrict banned users.
 - Get only `deleted = 0` records.
 - Implement caching of all queries.
 - Trigger user notification after every entry.
 - Delete relationship data after a delete query.
 - Insert relationship data after an insert query.
 - Keep records of modification after each update query.
 - Add/edit created_at and updated _at data after each entry.

#### Notes
 - Query Events go recursively, for example after inserting into `table_a` your event 
 inserts into `table_b`, now you can have another event registered with `table_b` 
 which inserts into `table_c`.
 - Of course Query Events don't work with raw queries.
 - **This is forked from awesome @usmanhalalit vai's [Pixie](https://github.com/usmanhalalit/pixie)
 and modified to support WordPress.**

___
