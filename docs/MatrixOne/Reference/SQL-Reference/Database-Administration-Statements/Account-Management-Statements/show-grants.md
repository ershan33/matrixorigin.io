# **SHOW GRANTS**

## **Description**

Use the `SHOW GRANTS` statement to display all grant information for a user. This would display privileges that were assigned to the user using the `GRANT` command.

`SHOW GRANTS` requires the `SELECT` privilege for the MatrixOne system schema, except to display privileges and roles for the current user.

To name the account or role for `SHOW GRANTS`, use the same format as for the `GRANT` statement, for example:

```
show grants for 'root'@'localhost';
```

## **Syntax**

```
> SHOW GRANTS FOR {username[@hostname] | rolename};
```

## **Examples**

```sql
> create role role1;
> grant all on table *.* to role1;
> grant create table, drop table on database *.* to role1;
> create user user1 identified by 'pass1';
> grant role1 to user1;
> show grants for 'user1'@'localhost';
+--------------------------------------------------------+
| Grants for user1@localhost                             |
+--------------------------------------------------------+
| GRANT connect ON account  `user1`@`localhost`          |
| GRANT table all ON table *.* `user1`@`localhost`       |
| GRANT create table ON database *.* `user1`@`localhost` |
| GRANT drop table ON database *.* `user1`@`localhost`   |
+--------------------------------------------------------+
4 rows in set (0.00 sec)
```

## **Constraints**

MatrixOne does not support on `SHOW GRANTS FOR {rolename}` for now.
