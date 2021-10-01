# Postgres Cheatsheet

This is a collection of the most common commands I run while administering Postgres databases. The variables shown between the open and closed tags, "<" and ">", should be replaced with a name you choose. Postgres has multiple shortcut functions, starting with a forward slash, "\". Any SQL command that is not a shortcut, must end with a semicolon, ";". You can use the keyboard UP and DOWN keys to scroll the history of previous commands you've run.


## Setup

##### installation, Ubuntu
http://www.postgresql.org/download/linux/ubuntu/
https://help.ubuntu.com/community/PostgreSQL
``` shell
sudo echo "deb http://apt.postgresql.org/pub/repos/apt/ wily-pgdg main" > \
  /etc/apt/sources.list.d/pgdg.list
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get install -y postgresql-9.5 postgresql-client-9.5 postgresql-contrib-9.5

sudo su - postgres
psql
```

##### connect
http://www.postgresql.org/docs/current/static/app-psql.html
```sql
psql

psql -U <username> -d <database> -h <hostname>

psql --username=<username> --dbname=<database> --host=<hostname>
```

##### disconnect
```sql
\q
\!
```

##### clear the screen
```sql
(CTRL + L)
```

##### info
```sql
\conninfo
```

##### configure

http://www.postgresql.org/docs/current/static/runtime-config.html

```shell
sudo nano $(locate -l 1 main/postgresql.conf)
sudo service postgresql restart
```

##### debug logs
```shell
# print the last 24 lines of the debug log
sudo tail -24 $(find /var/log/postgresql -name 'postgresql-*-main.log')
```
<br/><br/><br/>





## Recon

##### show version
```
SHOW SERVER_VERSION;
```

##### show system status
```sql
\conninfo
```

##### show environmental variables
```sql
SHOW ALL;
```

##### list users
```sql
SELECT rolname FROM pg_roles;
```

##### show current user
```sql
SELECT current_user;
```

##### show current user's permissions
```
\du
```

##### list databases
```sql
\l
```

##### show current database
```sql
SELECT current_database();
```

##### show all tables in database
```sql
\dt
```

##### list functions
```sql
\df <schema>
```
<br/><br/><br/>





## Databases

##### list databasees
```sql
\l
```

##### connect to database
```sql
\c <database_name>
```

##### show current database
```sql
SELECT current_database();
```

##### create database
http://www.postgresql.org/docs/current/static/sql-createdatabase.html
```sql
CREATE DATABASE <database_name> WITH OWNER <username>;
```
##### delete database
http://www.postgresql.org/docs/current/static/sql-dropdatabase.html
```sql
DROP DATABASE IF EXISTS <database_name>;
```

##### rename database
http://www.postgresql.org/docs/current/static/sql-alterdatabase.html
```sql
ALTER DATABASE <old_name> RENAME TO <new_name>;
```
<br/><br/><br/>




## Users

##### list roles
```sql
SELECT rolname FROM pg_roles;
```

##### create user
http://www.postgresql.org/docs/current/static/sql-createuser.html
```sql
CREATE USER <user_name> WITH PASSWORD '<password>';
```

##### drop user
http://www.postgresql.org/docs/current/static/sql-dropuser.html
```sql
DROP USER IF EXISTS <user_name>;
```

##### alter user password
http://www.postgresql.org/docs/current/static/sql-alterrole.html
```sql
ALTER ROLE <user_name> WITH PASSWORD '<password>';
```
<br/><br/><br/>





## Permissions

##### become the postgres user, if you have permission errors
```shell
sudo su - postgres
psql
```

##### grant all permissions on database
http://www.postgresql.org/docs/current/static/sql-grant.html
```sql
GRANT ALL PRIVILEGES ON DATABASE <db_name> TO <user_name>;
```

##### grant connection permissions on database
```sql
GRANT CONNECT ON DATABASE <db_name> TO <user_name>;
```

##### grant permissions on schema
```sql
GRANT USAGE ON SCHEMA public TO <user_name>;
```

##### grant permissions to functions
```sql
GRANT EXECUTE ON ALL FUNCTIONS IN SCHEMA public TO <user_name>;
```

##### grant permissions to select, update, insert, delete, on a all tables
```sql
GRANT SELECT, UPDATE, INSERT ON ALL TABLES IN SCHEMA public TO <user_name>;
```

##### grant permissions, on a table
```sql
GRANT SELECT, UPDATE, INSERT ON <table_name> TO <user_name>;
```

##### grant permissions, to select, on a table
```sql
GRANT SELECT ON ALL TABLES IN SCHEMA public TO <user_name>;
```
<br/><br/><br/>





## Schema

#####  list schemas
```sql
\dn

SELECT schema_name FROM information_schema.schemata;

SELECT nspname FROM pg_catalog.pg_namespace;
```

#####  create schema
http://www.postgresql.org/docs/current/static/sql-createschema.html
```sql
CREATE SCHEMA IF NOT EXISTS <schema_name>;
```

#####  drop schema
http://www.postgresql.org/docs/current/static/sql-dropschema.html
```sql
DROP SCHEMA IF EXISTS <schema_name> CASCADE;
```
<br/><br/><br/>





## Tables

##### list tables, in current db
```sql
\dt

SELECT table_schema,table_name FROM information_schema.tables ORDER BY table_schema,table_name;
```

##### list tables, globally
```sql
\dt *.*.

SELECT * FROM pg_catalog.pg_tables
```

##### list table schema
```sql
\d <table_name>
\d+ <table_name>

SELECT column_name, data_type, character_maximum_length
FROM INFORMATION_SCHEMA.COLUMNS
WHERE table_name = '<table_name>';
```

##### create table
http://www.postgresql.org/docs/current/static/sql-createtable.html
```sql
CREATE TABLE <table_name>(
  <column_name> <column_type>,
  <column_name> <column_type>
);
```

##### create table, with an auto-incrementing primary key
```sql
CREATE TABLE <table_name> (
  <column_name> SERIAL PRIMARY KEY
);
```

##### delete table
http://www.postgresql.org/docs/current/static/sql-droptable.html
```sql
DROP TABLE IF EXISTS <table_name> CASCADE;
```
<br/><br/><br/>





## Columns

##### add column
http://www.postgresql.org/docs/current/static/sql-altertable.html
```sql
ALTER TABLE <table_name> IF EXISTS
ADD <column_name> <data_type> [<constraints>];
```

##### update column
```sql
ALTER TABLE <table_name> IF EXISTS
ALTER <column_name> TYPE <data_type> [<constraints>];
```

##### delete column
```sql
ALTER TABLE <table_name> IF EXISTS
DROP <column_name>;
```

##### update column to be an auto-incrementing primary key
```sql
ALTER TABLE <table_name>
ADD COLUMN <column_name> SERIAL PRIMARY KEY;
```

##### insert into a table, with an auto-incrementing primary key
```sql
INSERT INTO <table_name>
VALUES (DEFAULT, <value1>);


INSERT INTO <table_name> (<column1_name>,<column2_name>)
VALUES ( <value1>,<value2> );
```
<br/><br/><br/>





## Data

##### read all data
http://www.postgresql.org/docs/current/static/sql-select.html
```sql
SELECT * FROM <table_name>;
```

##### read one row of data
```sql
SELECT * FROM <table_name> LIMIT 1;
```

##### search for data
```sql
SELECT * FROM <table_name> WHERE <column_name> = <value>;
```

##### insert data
http://www.postgresql.org/docs/current/static/sql-insert.html
```sql
INSERT INTO <table_name> VALUES( <value_1>, <value_2> );
```

##### edit data
http://www.postgresql.org/docs/current/static/sql-update.html
```sql
UPDATE <table_name>
SET <column_1> = <value_1>, <column_2> = <value_2>
WHERE <column_1> = <value>;
```

##### delete all data
http://www.postgresql.org/docs/current/static/sql-delete.html
```sql
DELETE FROM <table_name>;
```

##### delete specific data
```sql
DELETE FROM <table_name>
WHERE <column_name> = <value>;
```
<br/><br/><br/>





## Scripting

##### run local script, on remote host
http://www.postgresql.org/docs/current/static/app-psql.html
```shell
psql -U <username> -d <database> -h <host> -f <local_file>

psql --username=<username> --dbname=<database> --host=<host> --file=<local_file>
```

##### backup database data, everything
http://www.postgresql.org/docs/current/static/app-pgdump.html
```shell
pg_dump <database_name>

pg_dump <database_name>
```

##### backup database, only data
```shell
pg_dump -a <database_name>

pg_dump --data-only <database_name>
```

##### backup database, only schema
```shell
pg_dump -s <database_name>

pg_dump --schema-only <database_name>
```

##### restore database data
http://www.postgresql.org/docs/current/static/app-pgrestore.html
```shell
pg_restore -d <database_name> -a <file_pathway>

pg_restore --dbname=<database_name> --data-only <file_pathway>
```

##### restore database schema
```shell
pg_restore -d <database_name> -s <file_pathway>

pg_restore --dbname=<database_name> --schema-only <file_pathway>
```

##### export table into CSV file
http://www.postgresql.org/docs/current/static/sql-copy.html
```sql
\copy <table_name> TO '<file_path>' CSV
```

##### export table, only specific columns, to CSV file
```sql
\copy <table_name>(<column_1>,<column_1>,<column_1>) TO '<file_path>' CSV
```

##### import CSV file into table
http://www.postgresql.org/docs/current/static/sql-copy.html
```sql
\copy <table_name> FROM '<file_path>' CSV
```

##### import CSV file into table, only specific columns
```sql
\copy <table_name>(<column_1>,<column_1>,<column_1>) FROM '<file_path>' CSV
```
<br/><br/><br/>





## Debugging

http://www.postgresql.org/docs/current/static/using-explain.html

http://www.postgresql.org/docs/current/static/runtime-config-logging.html
<br/><br/><br/>





## Advanced Features
http://www.tutorialspoint.com/postgresql/postgresql_constraints.htm



## PSQL

Magic words:
```bash
psql -U postgres
```
Some interesting flags (to see all, use `-h` or `--help` depending on your psql version):
- `-E`: will describe the underlaying queries of the `\` commands (cool for learning!)
- `-l`: psql will list all databases and then exit (useful if the user you connect with doesn't has a default database, like at AWS RDS)

Most `\d` commands support additional param of `__schema__.name__` and accept wildcards like `*.*`

- `\?`: Show help (list of available commands with an explanation)
- `\q`: Quit/Exit
- `\c __database__`: Connect to a database
- `\d __table__`: Show table definition (columns, etc.) including triggers
- `\d+ __table__`: More detailed table definition including description and physical disk size
- `\l`: List databases
- `\dy`: List events
- `\df`: List functions
- `\di`: List indexes
- `\dn`: List schemas
- `\dt *.*`: List tables from all schemas (if `*.*` is omitted will only show SEARCH_PATH ones)
- `\dT+`: List all data types
- `\dv`: List views
- `\dx`: List all extensions installed
- `\df+ __function__` : Show function SQL code. 
- `\x`: Pretty-format query results instead of the not-so-useful ASCII tables
- `\copy (SELECT * FROM __table_name__) TO 'file_path_and_name.csv' WITH CSV`: Export a table as CSV
- `\des+`: List all foreign servers
- `\dE[S+]`: List all foreign tables
- `\! __bash_command__`: execute `__bash_command__` (e.g. `\! ls`)

User Related:
- `\du`: List users
- `\du __username__`: List a username if present.
- `create role __test1__`: Create a role with an existing username.
- `create role __test2__ noinherit login password __passsword__;`: Create a role with username and password.
- `set role __test__;`: Change role for current session to `__test__`.
- `grant __test2__ to __test1__;`: Allow `__test1__` to set its role as `__test2__`.
- `\deu+`: List all user mapping on server

## Configuration

- Service management commands:
```
sudo service postgresql stop
sudo service postgresql start
sudo service postgresql restart
```

- Changing verbosity & querying Postgres log:
  <br/>1) First edit the config file, set a decent verbosity, save and restart postgres:
```
sudo vim /etc/postgresql/9.3/main/postgresql.conf

# Uncomment/Change inside:
log_min_messages = debug5
log_min_error_statement = debug5
log_min_duration_statement = -1

sudo service postgresql restart
```
  2) Now you will get tons of details of every statement, error, and even background tasks like VACUUMs
```
tail -f /var/log/postgresql/postgresql-9.3-main.log
```
  3) How to add user who executed a PG statement to log (editing `postgresql.conf`):
```
log_line_prefix = '%t %u %d %a '
```

- Check Extensions enabled in postgres: `SELECT * FROM pg_extension;`

- Show available extensions: `SELECT * FROM pg_available_extension_versions;`

## Create command

There are many `CREATE` choices, like `CREATE DATABASE __database_name__`, `CREATE TABLE __table_name__` ... Parameters differ but can be checked [at the official documentation](https://www.postgresql.org/search/?u=%2Fdocs%2F9.1%2F&q=CREATE).

## Handy queries
- `SELECT * FROM pg_proc WHERE proname='__procedurename__'`: List procedure/function
- `SELECT * FROM pg_views WHERE viewname='__viewname__';`: List view (including the definition)
- `SELECT pg_size_pretty(pg_total_relation_size('__table_name__'));`: Show DB table space in use
- `SELECT pg_size_pretty(pg_database_size('__database_name__'));`: Show DB space in use
- `show statement_timeout;`: Show current user's statement timeout
- `SELECT * FROM pg_indexes WHERE tablename='__table_name__' AND schemaname='__schema_name__';`: Show table indexes
- Get all indexes from all tables of a schema:
```sql
SELECT
   t.relname AS table_name,
   i.relname AS index_name,
   a.attname AS column_name
FROM
   pg_class t,
   pg_class i,
   pg_index ix,
   pg_attribute a,
    pg_namespace n
WHERE
   t.oid = ix.indrelid
   AND i.oid = ix.indexrelid
   AND a.attrelid = t.oid
   AND a.attnum = ANY(ix.indkey)
   AND t.relnamespace = n.oid
    AND n.nspname = 'kartones'
ORDER BY
   t.relname,
   i.relname
```
- Execution data:
  - Queries being executed at a certain DB:
```sql
SELECT datname, application_name, pid, backend_start, query_start, state_change, state, query 
  FROM pg_stat_activity 
  WHERE datname='__database_name__';
```
  - Get all queries from all dbs waiting for data (might be hung): 
```sql
SELECT * FROM pg_stat_activity WHERE waiting='t'
```
  - Currently running queries with process pid:
```sql
SELECT 
  pg_stat_get_backend_pid(s.backendid) AS procpid, 
  pg_stat_get_backend_activity(s.backendid) AS current_query
FROM (SELECT pg_stat_get_backend_idset() AS backendid) AS s;
```
  - Get Connections by Database: `SELECT datname, numbackends FROM pg_stat_database;`

Casting:
- `CAST (column AS type)` or `column::type`
- `'__table_name__'::regclass::oid`: Get oid having a table name

Query analysis:
- `EXPLAIN __query__`: see the query plan for the given query
- `EXPLAIN ANALYZE __query__`: see and execute the query plan for the given query
- `ANALYZE [__table__]`: collect statistics  

Generating random data ([source](https://www.citusdata.com/blog/2019/07/17/postgres-tips-for-average-and-power-user/)):
- `INSERT INTO some_table (a_float_value) SELECT random() * 100000 FROM generate_series(1, 1000000) i;`

Get sizes of tables, indexes and full DBs:
```sql
select current_database() as database,
  pg_size_pretty(total_database_size) as total_database_size,
  schema_name,
  table_name,
  pg_size_pretty(total_table_size) as total_table_size,
  pg_size_pretty(table_size) as table_size,
  pg_size_pretty(index_size) as index_size
  from ( select table_name,
          table_schema as schema_name,
          pg_database_size(current_database()) as total_database_size,
          pg_total_relation_size(table_name) as total_table_size,
          pg_relation_size(table_name) as table_size,
          pg_indexes_size(table_name) as index_size
          from information_schema.tables
          where table_schema=current_schema() and table_name like 'table_%'
          order by total_table_size
      ) as sizes;
```

- [COPY command](https://www.postgresql.org/docs/9.2/sql-copy.html): Import/export from CSV to tables:
```sql 
COPY table_name [ ( column_name [, ...] ) ]
FROM { 'filename' | STDIN }
[ [ WITH ] ( option [, ...] ) ]

COPY { table_name [ ( column_name [, ...] ) ] | ( query ) }
TO { 'filename' | STDOUT }
[ [ WITH ] ( option [, ...] ) ]
```

- List all grants for a specific user
```sql
SELECT table_catalog, table_schema, table_name, privilege_type
FROM   information_schema.table_privileges
WHERE  grantee = 'user_to_check' ORDER BY table_name;
```

- List all assigned user roles
```sql
SELECT
    r.rolname,
    r.rolsuper,
    r.rolinherit,
    r.rolcreaterole,
    r.rolcreatedb,
    r.rolcanlogin,
    r.rolconnlimit,
    r.rolvaliduntil,
    ARRAY(SELECT b.rolname
      FROM pg_catalog.pg_auth_members m
      JOIN pg_catalog.pg_roles b ON (m.roleid = b.oid)
      WHERE m.member = r.oid) as memberof, 
    r.rolreplication
FROM pg_catalog.pg_roles r
ORDER BY 1;
```

- Check permissions in a table:
```sql
SELECT grantee, privilege_type
FROM information_schema.role_table_grants
WHERE table_name='name-of-the-table';
```

- Kill all Connections:
```sql
SELECT pg_terminate_backend(pg_stat_activity.pid)
FROM pg_stat_activity
WHERE datname = current_database() AND pid <> pg_backend_pid();
```


## Keyboard shortcuts
- `CTRL` + `R`: reverse-i-search

## Tools
- `ptop` and `pg_top`: `top` for PG. Available on the APT repository from `apt.postgresql.org`.
- [pg_activity](https://github.com/julmon/pg_activity): Command line tool for PostgreSQL server activity monitoring.
- [Unix-like reverse search in psql](https://dba.stackexchange.com/questions/63453/is-there-a-psql-equivalent-of-bashs-reverse-search-history):
```bash
$ echo "bind "^R" em-inc-search-prev" > $HOME/.editrc
$ source $HOME/.editrc
``` 
- Show IP of the DB Instance: `SELECT inet_server_addr();`
- File to save PostgreSQL credentials and permissions (format: `hostname:port:database:username:password`): `chmod 600 ~/.pgpass`
- Collect statistics of a database (useful to improve speed after a Database Upgrade as previous query plans are deleted): `ANALYZE VERBOSE;`

## Resources & Documentation
- [Postgres Weekly](https://postgresweekly.com/) newsletter: The best way IMHO to keep up to date with PG news
- [100 psql Tips](https://mydbanotebook.org/psql_tips_all.html): Name says all, lots of useful tips!
- [PostgreSQL Exercises](https://pgexercises.com/): An awesome resource to learn to learn SQL, teaching you with simple examples in a great visual way. **Highly recommended**.
- [A Performance Cheat Sheet for PostgreSQL](https://severalnines.com/blog/performance-cheat-sheet-postgresql): Great explanations of `EXPLAIN`, `EXPLAIN ANALYZE`, `VACUUM`, configuration parameters and more. Quite interesting if you need to tune-up a postgres setup.
- [annotated.conf](https://github.com/jberkus/annotated.conf): Annotations of all 269 postgresql.conf settings for PostgreSQL 10.
- `psql -c "\l+" -H -q postgres > out.html`: Generate a html report of your databases (source: [Daniel Westermann](https://twitter.com/westermanndanie/status/1242117182982586372))
