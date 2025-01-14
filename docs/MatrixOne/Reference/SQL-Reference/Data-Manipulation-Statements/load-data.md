# **LOAD DATA**

## **Description**

The LOAD DATA statement reads rows from a text file into a table at a very high speed. The file can be read from the server host or a [S3 compatible object storage](../../../Develop/import-data/bulk-load/load-s3.md). `LOAD DATA` is the complement of [`SELECT ... INTO OUTFILE`](../../../Develop/export-data/select-into-outfile.md). To write data from a table to a file, use `SELECT ... INTO OUTFILE`. To read the file back into a table, use LOAD DATA. The syntax of the `FIELDS` and `LINES` clauses is the same for both statements.

## **Syntax**

```
> LOAD DATA
    INFILE 'file_name'
    INTO TABLE tbl_name
    [{FIELDS | COLUMNS}
        [TERMINATED BY 'string']
        [[OPTIONALLY] ENCLOSED BY 'char']
        [ESCAPED BY 'char']
    ]
    [LINES
        [STARTING BY 'string']
        [TERMINATED BY 'string']
    ]
    [IGNORE number {LINES | ROWS}]
```

### Input File Location

Currently MatrixOne only supports to load data on server host, and the `file_name` must be an absolute path name for MatrixOne to locate it.

### IGNORE LINES

The IGNORE number LINES clause can be used to ignore lines at the start of the file. For example, you can use `IGNORE 1 LINES` to skip an initial header line containing column names:

```
LOAD DATA INFILE '/tmp/test.txt' INTO TABLE table1 IGNORE 1 LINES;
```

### Field and Line Handling

For both the LOAD DATA and `SELECT ... INTO OUTFILE` statements, the syntax of the FIELDS and LINES clauses is the same. Both clauses are optional, but FIELDS must precede LINES if both are specified.

If you specify a `FIELDS` clause, each of its subclauses (`TERMINATED BY`, `[OPTIONALLY] ENCLOSED BY`, and `ESCAPED BY`) is also optional, except that you must specify at least one of them. Arguments to these clauses are permitted to contain only ASCII characters.

If you specify no `FIELDS` or `LINES` clause, the defaults are the same as if you had written this:

```
FIELDS TERMINATED BY ',' ENCLOSED BY '"' LINES TERMINATED BY '\n'
```

**FIELDS TERMINATED BY**

`FIELDS TERMINATED BY` specifies the delimiter for a field. The `FIELDS TERMINATED BY` values can be more than one character.

For example, to read the comma-delimited file, the correct statement is:

```
LOAD DATA INFILE 'data.txt' INTO TABLE table1
  FIELDS TERMINATED BY ',';
```

If instead you tried to read the file with the statement shown following, it would not work because it instructs `LOAD DATA` to look for tabs between fields:

```
LOAD DATA INFILE 'data.txt' INTO TABLE table1
  FIELDS TERMINATED BY '\t';
```

The likely result is that each input line would be interpreted as a single field. You may encounter an error of `"ERROR 20101 (HY000): internal error: the table column is larger than input data column"`.

**FIELDS ENCLOSED BY**

`FIELDS TERMINATED BY` option specifies the character enclose the input values. `ENCLOSED BY` value must be a single character. If the input values are not necessarily enclosed within quotation marks, use `OPTIONALLY` before the `ENCLOSED BY` option.

For example, if some input values are enclosed within quotation marks, some are not:

```
LOAD DATA INFILE 'data.txt' INTO TABLE table1
  FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"';
```

**FIELDS ESCAPED BY**

`ESCAPE BY` option controls how to read or write special characters. `FIELDS ESCAPED BY` values must be a single character.

For input, if the `FIELDS ESCAPED BY` character is not empty, occurrences of that character are stripped and the following character is taken literally as part of a field value. Some two-character sequences that are exceptions, where the first character is the escape character. These sequences are shown in the following table (using \ for the escape character). The rules for `NULL` handling are described later in this section. If the `FIELDS ESCAPED BY` character is empty, escape-sequence interpretation does not occur.

|  Character   | Escape Sequence  |
|  ----  | ----  |
| \0 | 	An ASCII NUL (X'00') character |
| \b | 	A backspace character |
| \n | 	A newline (linefeed) character |
| \r | 	A carriage return character |
| \t | 	A tab character. |
| \Z | 	ASCII 26 (Control+Z) |
| \N | 	NULL |

For example, if some input values are special character `\`, you can use `escape by` as:

```
LOAD DATA INFILE 'data.txt' INTO TABLE table1
  FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' ESCAPED BY '\\';
```

**LINES TERMINATED BY**

`LINES TERMINATED BY` specifies the delimiter for the a line. The `LINES TERMINATED BY` values can be more than one character.

For example, if the lines in a csv file are terminated by carriage return/newline pairs, you can load it with `LINES TERMINATED BY '\r\n'`:

```
LOAD DATA INFILE 'data.txt' INTO TABLE table1
  FIELDS TERMINATED BY ',' ENCLOSED BY '"'
  LINES TERMINATED BY '\r\n';
```

**LINE STARTING BY**

If all the input lines have a common prefix that you want to ignore, you can use `LINES STARTING BY` 'prefix_string' to skip the prefix and anything before it. If a line does not include the prefix, the entire line is skipped. Suppose that you issue the following statement:

```
LOAD DATA INFILE '/tmp/test.txt' INTO TABLE table1
  FIELDS TERMINATED BY ','  LINES STARTING BY 'xxx';
```

If the data file looks like this:

```
xxx"abc",1
something xxx"def",2
"ghi",3
```

The resulting rows are ("abc",1) and ("def",2). The third row in the file is skipped because it does not contain the prefix.

## Supported file formats

In MatrixOne's current release, `LOAD DATA` supports CSV(comma-separated values) format and JSONLines format file.
See full tutorials for loading [csv](../../../Develop/import-data/bulk-load/load-csv.md) and [jsonline](../../../Develop/import-data/bulk-load/load-jsonline.md).

## **Examples**

The SSB Test is an example of LOAD DATA syntax. [Complete a SSB Test with MatrixOne](../../../Test/performance-testing/SSB-test-with-matrixone.md)

```
> LOAD DATA INFILE '/ssb-dbgen-path/lineorder_flat.tbl ' INTO TABLE lineorder_flat;
```

The above statement means: load the *lineorder_flat.tbl* data set under the directory path */ssb-dbgen-path/* into the MatrixOne data table *lineorder_flat*.

You can also refer to the following syntax examples to quickly understand `LOAD DATA`:

### Example 1: LOAD CSV

#### Simple example

The data in the file locally named *char_varchar.csv* is as follows:

```
a|b|c|d
"a"|"b"|"c"|"d"
'a'|'b'|'c'|'d'
"'a'"|"'b'"|"'c'"|"'d'"
"aa|aa"|"bb|bb"|"cc|cc"|"dd|dd"
"aa|"|"bb|"|"cc|"|"dd|"
"aa|||aa"|"bb|||bb"|"cc|||cc"|"dd|||dd"
"aa'|'||aa"|"bb'|'||bb"|"cc'|'||cc"|"dd'|'||dd"
aa"aa|bb"bb|cc"cc|dd"dd
"aa"aa"|"bb"bb"|"cc"cc"|"dd"dd"
"aa""aa"|"bb""bb"|"cc""cc"|"dd""dd"
"aa"""aa"|"bb"""bb"|"cc"""cc"|"dd"""dd"
"aa""""aa"|"bb""""bb"|"cc""""cc"|"dd""""dd"
"aa""|aa"|"bb""|bb"|"cc""|cc"|"dd""|dd"
"aa""""|aa"|"bb""""|bb"|"cc""""|cc"|"dd""""|dd"
|||
||||
""|""|""|
""""|""""|""""|""""
""""""|""""""|""""""|""""""
```

Create a table named t1 in MatrixOne:

```sql
mysql> drop table if exists t1;
Query OK, 0 rows affected (0.01 sec)

mysql> create table t1(
    -> col1 char(225),
    -> col2 varchar(225),
    -> col3 text,
    -> col4 varchar(225)
    -> );
Query OK, 0 rows affected (0.02 sec)
```

Load the data file into table t1:

```sql
load data infile '<your-local-file-path>/char_varchar.csv' into table t1 fields terminated by'|';
```

The query result is as follows:

```
mysql> select * from t1;
+-----------+-----------+-----------+-----------+
| col1      | col2      | col3      | col4      |
+-----------+-----------+-----------+-----------+
| a         | b         | c         | d         |
| a         | b         | c         | d         |
| 'a'       | 'b'       | 'c'       | 'd'       |
| 'a'       | 'b'       | 'c'       | 'd'       |
| aa|aa     | bb|bb     | cc|cc     | dd|dd     |
| aa|       | bb|       | cc|       | dd|       |
| aa|||aa   | bb|||bb   | cc|||cc   | dd|||dd   |
| aa'|'||aa | bb'|'||bb | cc'|'||cc | dd'|'||dd |
| aa"aa     | bb"bb     | cc"cc     | dd"dd     |
| aa"aa     | bb"bb     | cc"cc     | dd"dd     |
| aa"aa     | bb"bb     | cc"cc     | dd"dd     |
| aa""aa    | bb""bb    | cc""cc    | dd""dd    |
| aa""aa    | bb""bb    | cc""cc    | dd""dd    |
| aa"|aa    | bb"|bb    | cc"|cc    | dd"|dd    |
| aa""|aa   | bb""|bb   | cc""|cc   | dd""|dd   |
|           |           |           |           |
|           |           |           |           |
|           |           |           |           |
| "         | "         | "         | "         |
| ""        | ""        | ""        | ""        |
+-----------+-----------+-----------+-----------+
20 rows in set (0.00 sec)
```

### Add conditional Example

Following the example above, you can modify the `LOAD DATA` statement and add `LINES STARTING BY 'aa' ignore 10 lines;` at the end of the statement to experience the difference:

```sql
delete from t1;
load data infile '<your-local-file-path>/char_varchar.csv' into table t1 fields terminated by'|' LINES STARTING BY 'aa' ignore 10 lines;
```

The query result is as follows:

```sql
mysql> select * from t1;
+---------+---------+---------+---------+
| col1    | col2    | col3    | col4    |
+---------+---------+---------+---------+
| aa"aa   | bb"bb   | cc"cc   | dd"dd   |
| aa""aa  | bb""bb  | cc""cc  | dd""dd  |
| aa""aa  | bb""bb  | cc""cc  | dd""dd  |
| aa"|aa  | bb"|bb  | cc"|cc  | dd"|dd  |
| aa""|aa | bb""|bb | cc""|cc | dd""|dd |
|         |         |         |         |
|         |         |         |         |
|         |         |         |         |
| "       | "       | "       | "       |
| ""      | ""      | ""      | ""      |
+---------+---------+---------+---------+
10 rows in set (0.00 sec)
```

As you can see, the query result ignores the first line and
and ignores the common prefix aa.

For more information on loding *csv*, see [Import the *.csv* data](../../../Develop/import-data/bulk-load/load-jsonline.md)。

### Example 2: LOAD JSONLines

#### Simple example

The data in the file locally named *jsonline_array.jl* is as follows:

```
[true,1,"var","2020-09-07","2020-09-07 00:00:00","2020-09-07 00:00:00","18",121.11,["1",2,null,false,true,{"q":1}],"1qaz",null,null]
["true","1","var","2020-09-07","2020-09-07 00:00:00","2020-09-07 00:00:00","18","121.11",{"c":1,"b":["a","b",{"q":4}]},"1aza",null,null]
```

Create a table named t1 in MatrixOne:

```sql
mysql> drop table if exists t1;
Query OK, 0 rows affected (0.01 sec)

mysql> create table t1(col1 bool,col2 int,col3 varchar(100), col4 date,col5 datetime,col6 timestamp,col7 decimal,col8 float,col9 json,col10 text,col11 json,col12 bool);
Query OK, 0 rows affected (0.03 sec)
```

Load the data file into table t1:

```
load data infile {'filepath'='<your-local-file-path>/jsonline_array.jl','format'='jsonline','jsondata'='array'} into table t1;
```

The query result is as follows:

```sql
mysql> select * from t1;
+------+------+------+------------+---------------------+---------------------+------+--------+---------------------------------------+-------+-------+-------+
| col1 | col2 | col3 | col4       | col5                | col6                | col7 | col8   | col9                                  | col10 | col11 | col12 |
+------+------+------+------------+---------------------+---------------------+------+--------+---------------------------------------+-------+-------+-------+
| true |    1 | var  | 2020-09-07 | 2020-09-07 00:00:00 | 2020-09-07 00:00:00 |   18 | 121.11 | ["1", 2, null, false, true, {"q": 1}] | 1qaz  | NULL  | NULL  |
| true |    1 | var  | 2020-09-07 | 2020-09-07 00:00:00 | 2020-09-07 00:00:00 |   18 | 121.11 | {"b": ["a", "b", {"q": 4}], "c": 1}   | 1aza  | NULL  | NULL  |
+------+------+------+------------+---------------------+---------------------+------+--------+---------------------------------------+-------+-------+-------+
2 rows in set (0.00 sec)
```

#### Add conditional Example

Following the example above, you can modify the `LOAD DATA` statement and add `ignore 1 lines` at the end of the statement to experience the difference:

```
delete from t1;
load data infile {'filepath'='<your-local-file-path>/jsonline_array.jl','format'='jsonline','jsondata'='array'} into table t1 ignore 1 lines;
```

The query result is as follows:

```sql
mysql> select * from t1;
+------+------+------+------------+---------------------+---------------------+------+--------+-------------------------------------+-------+-------+-------+
| col1 | col2 | col3 | col4       | col5                | col6                | col7 | col8   | col9                                | col10 | col11 | col12 |
+------+------+------+------------+---------------------+---------------------+------+--------+-------------------------------------+-------+-------+-------+
| true |    1 | var  | 2020-09-07 | 2020-09-07 00:00:00 | 2020-09-07 00:00:00 |   18 | 121.11 | {"b": ["a", "b", {"q": 4}], "c": 1} | 1aza  | NULL  | NULL  |
+------+------+------+------------+---------------------+---------------------+------+--------+-------------------------------------+-------+-------+-------+
1 row in set (0.00 sec)
```

As you can see, the query result ignores the first line.

For more information on loding *JSONLines*, see [Import the JSONLines data](../../../Develop/import-data/bulk-load/load-jsonline.md).

## **Constraints**

1. `LOAD DATA` doesn't support to load files on the client host yet.
2. The `REPLACE` and `IGNORE` modifiers control handling of new (input) rows that duplicate existing table rows on unique key values (`PRIMARY KEY` or `UNIQUE index` values) are not supported in MatrixOne yet.
3. Only absolute file path is supported.
4. Input pre-pressing with `SET` is supported very limitedly. Only `SET columns_name=nullif(expr1,expr2)` is supported.
