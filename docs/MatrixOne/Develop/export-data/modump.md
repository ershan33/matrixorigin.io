# Export data by `modump`

There are two methods to export data with MatrixOne:

- `SELECT INTO...OUTFILE`
- `modump`

This document will introduce about how to export data with `modump`.

## What is `modump`

Like `mysqldump`, MatrixOne has a client utility tool called `modump` that can perform backups of a MatrixOne database by exporting a “.sql” file type that contains SQL statements can be executed to recreate the original database.

To use the `modump` tool, you must have access to a server running an instance of MatrixOne. You must also have user credentials with the required privileges for the database you want to export.

### Syntax

```
./mo-dump -u ${user} -p ${password} -h ${host} -P ${port} -db ${database} [-tbl ${table}...] > {dumpfilename.sql}
```

The parameters are as following:

- **-u [user]**:  It is a username to connect to the MatrixOne server. Only the users with database and table read privileges can use `modump` utility. Default value: dump

- **-p [password]**: The valid password of the MatrixOne user. Default value: 111

- **-h [host]**: The host ip address of MatrixOne server. Default value: 127.0.0.1

- **-P [port]**: The port of MatrixOne server. Default value: 6001

- **-db [database name]**: Required paratemer. Name of the database that you want to take backup.

- **-tbl [table name]**: Optional parameter. If the parameter is empty, the whole database will be exported. If you want to take the backup specific tables, then you can specify multiple `-tbl` and table names in the command.

## Build the modump binary

To use `modump` utility, we need to build the tool first. `modump` is embedded in the MatrixOne source code. You can build the binary from the source code.

__Tips:__ Same as MatrixOne `modump` is written by Golang, building it will require a <a href="https://go.dev/doc/install" target="_blank">Golang</a> installation and environment setting.

1. Execute the following code to build the `modump` binary from the MatrixOne source code:

    ```
    git clone https://github.com/matrixorigin/matrixone.git
    cd matrixone
    make build modump
    ```

2. Then you can find the `modump` executable file in the MatrixOne folder.

!!! note
    This built `modump` file can also work in a same hardware platform. But a binary built in a x86 platform will not work correctly in a darwin ARM platform. The best practice is to build and use the binary file within the same operating system and hardware platform. `modump` only supports Linux and macOS for now.

## Steps to Export your MatrixOne Database using `modump`

`modump` is easy to use with the command line. Here are the steps to take to export a complete database in the form of SQL commands:

1. Open up a command line or terminal window on your computer.
2. Verify that from this terminal you can connect to your MatrixOne instance.
3. Enter this command:

```
./modump -u username -p password -h host_ip_address -P port -db database > exporteddb.sql
```

For example, if you are launching the terminal in the same server as the MatrixOne instance, and you want to generate the backup of the single database, run the following command. The command will generate the backup of the “**t**” database with structure and data in the `t.sql` file. The `t.sql` file will be located in the same directory as your `modump` executable.

```
./modump -u dump -p 111 -h 127.0.0.1 -P 6001 -db t > t.sql
```

If you want to generate the backup of a single table in a database, run the following command. The command will generate the backup of the `t1` table of  `t` database with structure and data in the `t.sql` file.

```
./modump -u dump -p 111 -db t -tbl t1 > t1.sql
```

## Constraints

* `modump` only supports exporting the backup of a single database, if you have several databases to backup, you need to manually run `modump` for several times.

* `modump` doesn't support exporting only the structure or data of databases. If you want to generate the backup of the data without the database structure or vise versa, you need to manually split the `.sql` file.
