# The JSON Data Type

MatrixOne supports a native JSON data type defined by RFC 7159 that enables efficient access to data in JSON (JavaScript Object Notation) documents. The JSON data type provides these advantages over storing JSON-format strings in a string column:

Automatic validation of JSON documents stored in JSON columns. Invalid documents produce an error.

Optimized storage format. JSON documents stored in JSON columns are converted to an internal format that permits quick read access to document elements. When the server later must read a JSON value stored in this binary format, the value need not be parsed from a text representation. The binary format is structured to enable the server to look up subobjects or nested values directly by key or array index without reading all values before or after them in the document.

The space required to store a JSON document is roughly the same as for `LONGBLOB` or `LONGTEXT`.

## Creating JSON Values

A JSON array contains a list of values separated by commas and enclosed within [ and ] characters:

```
["abc", 10, null, true, false]
```

A JSON object contains a set of key-value pairs separated by commas and enclosed within { and } characters:

```
{"k1": "value", "k2": 10}
```

As the examples illustrate, JSON arrays and objects can contain scalar values that are strings or numbers, the JSON null literal, or the JSON boolean true or false literals. Keys in JSON objects must be strings. Temporal (date,  datetime) scalar values are also permitted:

```
["12:18:29.000000", "2015-07-29", "2015-07-29 12:18:29.000000"]
```

Nesting is permitted within JSON array elements and JSON object key values:

```
[99, {"id": "HK500", "cost": 75.99}, ["hot", "cold"]]
{"k1": "value", "k2": [10, 20]}
```

## Normalization of JSON Values

When a string is parsed and found to be a valid JSON document, it is also normalized. This means that members with keys that duplicate a key found later in the document, reading from left to right, are discarded.

Normalization is performed when values are inserted into JSON columns, as shown here:

```sql
> CREATE TABLE t1 (c1 JSON);
> INSERT INTO t1 VALUES
     ('{"x": 17, "x": "red"}'),
     ('{"x": 17, "x": "red", "x": [3, 5, 7]}');
> SELECT c1 FROM t1;
+------------------+
| c1               |
+------------------+
| {"x": "red"}     |
| {"x": [3, 5, 7]} |
+------------------+
```

## Searching and Modifying JSON Values

A JSON path expression selects a value within a JSON document.

Path expressions are useful with functions that extract parts of or modify a JSON document, to specify where within that document to operate. For example, the following query extracts from a JSON document the value of the member with the name key:

```sql
> SELECT JSON_EXTRACT('{"id": 14, "name": "Aztalan"}', '$.name');
+---------------------------------------------------------+
| JSON_EXTRACT('{"id": 14, "name": "Aztalan"}', '$.name') |
+---------------------------------------------------------+
| "Aztalan"                                               |
+---------------------------------------------------------+
```

Path syntax uses a leading $ character to represent the JSON document under consideration, optionally followed by selectors that indicate successively more specific parts of the document:

- A period followed by a key name names the member in an object with the given key. The key name must be specified within double quotation marks if the name without quotes is not legal within path expressions (for example, if it contains a space).

- [N] appended to a *path* that selects an array names the value at position `N` within the array. Array positions are integers beginning with zero.

- Paths can contain * or ** wildcards:

   + .[*] evaluates to the values of all members in a JSON object.

   + [*] evaluates to the values of all elements in a JSON array.

   + prefix**suffix evaluates to all paths that begin with the named prefix and end with the named suffix.

- A path that does not exist in the document (evaluates to nonexistent data) evaluates to NULL.

**Example**:

```
[3, {"a": [5, 6], "b": 10}, [99, 100]]
```

- $[0] evaluates to 3.

- $[1] evaluates to {"a": [5, 6], "b": 10}.

- $[2] evaluates to [99, 100].

- $[3] evaluates to NULL (it refers to the fourth array element, which does not exist).

Because $[1] and $[2] evaluate to nonscalar values, they can be used as the basis for more-specific path expressions that select nested values. Examples:

- $[1].a evaluates to [5, 6].

- $[1].a[1] evaluates to 6.

- $[1].b evaluates to 10.

- $[2][0] evaluates to 99.

As mentioned previously, path components that name keys must be quoted if the unquoted key name is not legal in path expressions. Let $ refer to this value:

```
{"a fish": "shark", "a bird": "sparrow"}
```

The keys both contain a space and must be quoted:

- $."a fish" evaluates to shark.

- $."a bird" evaluates to sparrow.

Paths that use wildcards evaluate to an array that can contain multiple values:

```sql
> SELECT JSON_EXTRACT('{"a": 1, "b": 2, "c": [3, 4, 5]}', '$.*');
+---------------------------------------------------------+
| JSON_EXTRACT('{"a": 1, "b": 2, "c": [3, 4, 5]}', '$.*') |
+---------------------------------------------------------+
| [1, 2, [3, 4, 5]]                                       |
+---------------------------------------------------------+

> SELECT JSON_EXTRACT('{"a": 1, "b": 2, "c": [3, 4, 5]}', '$.c[*]');
+------------------------------------------------------------+
| JSON_EXTRACT('{"a": 1, "b": 2, "c": [3, 4, 5]}', '$.c[*]') |
+------------------------------------------------------------+
| [3, 4, 5]                                                  |
+------------------------------------------------------------+
```

In the following example, the path $**.b evaluates to multiple paths ($.a.b and $.c.b) and produces an array of the matching path values:

```sql
> SELECT JSON_EXTRACT('{"a": {"b": 1}, "c": {"b": 2}}', '$**.b');
+---------------------------------------------------------+
| JSON_EXTRACT('{"a": {"b": 1}, "c": {"b": 2}}', '$**.b') |
+---------------------------------------------------------+
| [null, 1, 2]                                                  |
+---------------------------------------------------------+
```

In the following example, showes the querying JSON values from columns:

```sql
> create table t1 (a json,b int);
> insert into t1(a,b) values ('{"a":1,"b":2,"c":3}',1);
> select json_extract(t1.a,'$.a') from t1 where t1.b=1;
+-------------------------+
| json_extract(t1.a, $.a) |
+-------------------------+
| 1                       |
+-------------------------+
1 row in set (0.00 sec)

> insert into t1(a,b) values ('{"a":4,"b":5,"c":6}',2);
> select json_extract(t1.a,'$.b') from t1 where t1.b=2;
+-------------------------+
| json_extract(t1.a, $.b) |
+-------------------------+
| 5                       |
+-------------------------+
1 row in set (0.00 sec)

> select json_extract(t1.a,'$.a') from t1;
+-------------------------+
| json_extract(t1.a, $.a) |
+-------------------------+
| 1                       |
| 4                       |
+-------------------------+
2 rows in set (0.00 sec)

> insert into t1(a,b) values ('{"a":{"q":[1,2,3]}}',3);
> select json_extract(t1.a,'$.a.q[1]') from t1 where t1.b=3;
+------------------------------+
| json_extract(t1.a, $.a.q[1]) |
+------------------------------+
| 2                            |
+------------------------------+
1 row in set (0.01 sec)

> insert into t1(a,b) values ('[{"a":1,"b":2,"c":3},{"a":4,"b":5,"c":6}]',4);
> select json_extract(t1.a,'$[1].a') from t1 where t1.b=4;
+----------------------------+
| json_extract(t1.a, $[1].a) |
+----------------------------+
| 4                          |
+----------------------------+
1 row in set (0.00 sec)
```
