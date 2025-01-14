# **Data Types Overview**

MatrixOne Data types conforms with MySQL Data types definition.

Reference: <https://dev.mysql.com/doc/refman/8.0/en/data-types.html>

## **Integer Numbers**

|  Data Type   | Size  |  Min Value   | Max Value  |
|  ----  | ----  |  ----  | ----  |
| TINYINT  | 1 byte | 	-128  | 127 |
| SMALLINT  | 2 bytes | -32768  | 32767 |
| INT  | 4 bytes | 	-2147483648	  | 2147483647 |
| BIGINT  | 8 bytes | -9223372036854775808	  | 9223372036854775807 |
| TINYINT UNSIGNED | 1 byte | 0	  | 255 |
| SMALLINT UNSIGNED | 2 bytes | 0	  | 65535 |
| INT UNSIGNED | 4 bytes | 0	  | 4294967295 |
| BIGINT UNSIGNED | 8 bytes | 0	  | 18446744073709551615 |

### **Examples**

- TINYINT and TINYINT UNSIGNED

```sql
//Create a table named "inttable" with 2 attributes of a "tinyint", a "tinyint unsigned",
create table inttable ( a tinyint not null default 1, tinyint8 tinyint unsigned primary key);
insert into inttable (tinyint8) values (0),(255), (0xFE), (253);
mysql> select * from inttable order by 2 asc;
+------+----------+
| a    | tinyint8 |
+------+----------+
|    1 |        0 |
|    1 |      253 |
|    1 |      254 |
|    1 |      255 |
+------+----------+
4 rows in set (0.03 sec)
```

- SMALLINT and SMALLINT UNSIGNED

```sql
////Create a table named "inttable" with 2 attributes of a "smallint", a "smallint unsigned",
drop table inttable;
create table inttable ( a smallint not null default 1, smallint16 smallint unsigned);
insert into inttable (smallint16) values (0),(65535), (0xFFFE), (65534), (65533);
mysql> select * from inttable;
+------+------------+
| a    | smallint16 |
+------+------------+
|    1 |          0 |
|    1 |      65535 |
|    1 |      65534 |
|    1 |      65534 |
|    1 |      65533 |
+------+------------+
5 rows in set (0.01 sec)
```

- INT and INT UNSIGNED

```sql
//Create a table named "inttable" with 2 attributes of a "int", a "int unsigned",
drop table inttable;
create table inttable ( a int not null default 1, int32 int unsigned primary key);
insert into inttable (int32) values (0),(4294967295), (0xFFFFFFFE), (4294967293), (4294967291);
mysql> select * from inttable order by a desc, 2 asc;
+------+------------+
| a    | int32      |
+------+------------+
|    1 |          0 |
|    1 | 4294967291 |
|    1 | 4294967293 |
|    1 | 4294967294 |
|    1 | 4294967295 |
+------+------------+
5 rows in set (0.01 sec)
```

- BIGINT and BIGINT UNSIGNED

```sql
//Create a table named "inttable" with 2 attributes of a "bigint", a "bigint unsigned",
drop table inttable;
create table inttable ( a bigint, big bigint primary key );
insert into inttable values (122345515, 0xFFFFFFFFFFFFE), (1234567, 0xFFFFFFFFFFFF0);
mysql> select * from inttable;
+-----------+------------------+
| a         | big              |
+-----------+------------------+
| 122345515 | 4503599627370494 |
|   1234567 | 4503599627370480 |
+-----------+------------------+
2 rows in set (0.01 sec)
```

## **Real Numbers**

|  Data Type   | Size  |  Precision   | Syntax |
|  ----  | ----  |  ----  | ----  |
| FLOAT32  | 4 bytes | 	23 bits  | FLOAT |
| FLOAT64  | 8 bytes |  53 bits  | DOUBLE |

### **Examples**

```sql
//Create a table named "floattable" with 1 attributes of a "float"
create table floattable ( a float not null default 1, big float(20,5) primary key);
insert into floattable (big) values (-1),(12345678.901234567),(92233720368547.75807);
mysql> select * from floattable order by a desc, big asc;
+------+----------------+
| a    | big            |
+------+----------------+
|    1 |             -1 |
|    1 |       12345679 |
|    1 | 92233720000000 |
+------+----------------+
3 rows in set (0.01 sec)

mysql> select min(big),max(big),max(big)-1 from floattable;
+----------+----------------+----------------+
| min(big) | max(big)       | max(big) - 1   |
+----------+----------------+----------------+
|       -1 | 92233720000000 | 92233718038527 |
+----------+----------------+----------------+
1 row in set (0.01 sec)
```

## **String Types**

|  Data Type   | Size |Length | Syntax | Description|
|  ----  | ----  |  ---  | ----  | ---- |
| char      | 24 bytes| 0 ~ 4294967295 |CHAR| Fixed length string |
| varchar   | 24 bytes| 0 ~ 4294967295 |VARCHAR| Variable length string|
| text      | 1 GB|other types mapping |TEXT |Long text data, TINY TEXT, MEDIUM TEXT, and LONG TEXT are not distinguished|
| blob      | 1 GB| other types mapping|BLOB |Long text data in binary form, TINY BLOB, MEDIUM BLOB, and LONG BLOB are not distinguished|

### **Examples**

- CHAR and VARCHAR

```sql
//Create a table named "names" with 2 attributes of a "varchar" and a "char"
create table names(name varchar(255),age char(255));
insert into names(name, age) values('Abby', '24');
insert into names(name, age) values("Bob", '25');
insert into names(name, age) values('Carol', "23");
insert into names(name, age) values("Dora", "29");
mysql> select name,age from names;
+-------+------+
| name  | age  |
+-------+------+
| Abby  | 24   |
| Bob   | 25   |
| Carol | 23   |
| Dora  | 29   |
+-------+------+
4 rows in set (0.00 sec)
```

- TEXT

```sql
//Create a table named "texttest" with 1 attribute of a "text"
create table texttest (a text);
insert into texttest values('abcdef');
insert into texttest values('_bcdef');
insert into texttest values('a_cdef');
insert into texttest values('ab_def');
insert into texttest values('abc_ef');
insert into texttest values('abcd_f');
insert into texttest values('abcde_');
> select * from texttest where a like 'ab\_def' order by 1 asc;
+--------+
| a      |
+--------+
| ab_def |
+--------+
1 row in set (0.01 sec)
```

- BLOB

```sql
// Create a table named "blobtest" with 1 attribute of a "blob"
create table blobtest (a blob);
insert into blobtest values('abcdef');
insert into blobtest values('_bcdef');
insert into blobtest values('a_cdef');
insert into blobtest values('ab_def');
insert into blobtest values('abc_ef');
insert into blobtest values('abcd_f');
insert into blobtest values('abcde_');
> select * from blobtest where a like 'ab\_def' order by 1 asc;
+----------------+
| a              |
+----------------+
| 0x61625F646566 |
+----------------+
1 row in set (0.01 sec)
```

## **JSON Types**

|JSON Data Type| Syntax |
|---|---|
|Object|Object is enclosed by `{}`, separated by commas between key-value pairs, and separated by colons `:` between keys and values.<br>The value/key can be String, Number, Bool, Time and date.|
|Array|Array is enclosed by `[]`, separated by commas between key-value pairs, and separated by colons `:` between keys and values. <br>The value can be String, Number, Bool, Time and date.|

### **Examples**

```sql
//Create a table named "jsontest" with 1 attribute of a "json"
create table jsontest (a json,b int);
insert into jsontest values ('{"t1":"a"}',1),('{"t1":"b"}',2);
> select * from jsontest;
+-------------+------+
| a           | b    |
+-------------+------+
| {"t1": "a"} |    1 |
| {"t1": "b"} |    2 |
+-------------+------+
2 rows in set (0.01 sec)
```

## **Time and Date Types**

|  Data Type   | Size  | Resolution |  Min Value   | Max Value  | Precision |
|  ----  | ----  |   ----  |  ----  | ----  |   ----  |
|  Time  | 8 bytes  |   microsecond  |  -2562047787:59:59.999999 | 2562047787:59:59.999999  |   hh:mm:ss.ssssss  |
| Date  | 4 bytes | day | 0001-01-01  | 9999-12-31 | YYYY-MM-DD/YYYYMMDD |
| DateTime  | 8 bytes | microsecond | 0001-01-01 00:00:00.000000  | 9999-12-31 23:59:59.999999 | YYYY-MM-DD hh:mi:ssssss |
| TIMESTAMP|8 bytes|microsecond|0001-01-01 00:00:00.000000|9999-12-31 23:59:59.999999|YYYYMMDD hh:mi:ss.ssssss|

### **Examples**

- TIME

```sql
//Create a table named "timetest" with 1 attributes of a "time"
create table time_02(t1 time);
insert into time_02 values(200);
insert into time_02 values("");
select * from time_02;
+----------+
| t1       |
+----------+
| 00:02:00 |
| NULL     |
+----------+
2 rows in set (0.00 sec)
```

- DATE

```sql
//Create a table named "datetest" with 1 attributes of a "date"
create table datetest (a date not null, primary key(a));
insert into datetest values ('2022-01-01'), ('20220102'),('2022-01-03'),('20220104');
select * from datetest order by a asc;
+------------+
| a          |
+------------+
| 2022-01-01 |
| 2022-01-02 |
| 2022-01-03 |
| 2022-01-04 |
+------------+
```

- DATETIME

```sql
//Create a table named "datetimetest" with 1 attributes of a "datetime"
create table datetimetest (a datetime(0) not null, primary key(a));
insert into datetimetest values ('20200101000000'), ('2022-01-02'), ('2022-01-02 00:00:01'), ('2022-01-02 00:00:01.512345');
> select * from datetimetest order by a asc;
+---------------------+
| a                   |
+---------------------+
| 2020-01-01 00:00:00 |
| 2022-01-02 00:00:00 |
| 2022-01-02 00:00:01 |
| 2022-01-02 00:00:02 |
+---------------------+
4 rows in set (0.02 sec)
```

- TIMESTAMP

```sql
//Create a table named "timestamptest" with 1 attribute of a "timestamp"
create table timestamptest (a timestamp(0) not null, primary key(a));
insert into timestamptest values ('20200101000000'), ('2022-01-02'), ('2022-01-02 00:00:01'), ('2022-01-02 00:00:01.512345');
> select * from timestamptest;
+---------------------+
| a                   |
+---------------------+
| 2020-01-01 00:00:00 |
| 2022-01-02 00:00:00 |
| 2022-01-02 00:00:01 |
| 2022-01-02 00:00:02 |
+---------------------+
```

## **Bool**

|  Data Type   | Size  |
|  ----  | ----  |
| True  | 1 byte |
|False|1 byte|

### **Examples**

```sql
//Create a table named "booltest" with 2 attribute of a "boolean" and b "bool"
> create table booltest (a boolean,b bool);
> insert into booltest values (0,1),(true,false),(true,1),(0,false),(NULL,NULL);
> select * from booltest;
+-------+-------+
| a     | b     |
+-------+-------+
| false | true  |
| true  | false |
| true  | true  |
| false | false |
| NULL  | NULL  |
+-------+-------+
5 rows in set (0.00 sec)
```

## **Decimal Types(Beta)**

|  Data Type   | Size  |  Precision   | Syntax |
|  ----  | ----  |  ----  | ----  |
| Decimal  | 8 bytes | 	19 digits  | Decimal(N,S) <br> N is the total number of digits, the range is(1 ~ 18). The decimal point and (for negative numbers) the - sign are not counted in N. <br>S is the number of digits after the decimal point (the scale), the range is(0 ~ N)<br> If S is 0, values have no decimal point or fractional part. If S is omitted, the default is 0. If N is omitted, the default is 1. <br>For example, Decimal(10,8) represents a number with a total length of 10 and a decimal place of 8. |
| Decimal  | 16 bytes | 	38 digits  |  Decimal(N,S) <br> N is the total number of digits, the range is(18 ~ 38). The decimal point and (for negative numbers) the - sign are not counted in N. <br>S is the number of digits after the decimal point (the scale), the range is(0 ~ N)<br> If S is 0, values have no decimal point or fractional part. If S is omitted, the default is 0. If N is omitted, the default is 18. <br>For example, Decimal(20,9) represents a number with a total length of 20 and a decimal place of 9.  |

### **Examples**

```sql
//Create a table named "decimalTest" with 2 attribute of a "decimal" and b "decimal"
> create table decimalTest(a decimal(6,3), b decimal(24,18));
> insert into decimalTest values(123.4567, 123456.1234567891411241355);
> select * from decimalTest;
+---------+---------------------------+
| a       | b                         |
+---------+---------------------------+
| 123.456 | 123456.123456789141124135 |
+---------+---------------------------+
```
