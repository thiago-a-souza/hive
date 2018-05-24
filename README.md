# Author
Thiago Alexandre Domingues de Souza

# Apache Hive
MapReduce programs allowed processing very large datasets that were not possible before. However, developing solutions using this programming model can be challenging, taking hours to write algorithms for typical problems. As a result, Facebook introduced Hive, a data warehouse platform that can process petabytes worth of data using a syntax based on SQL called HQL (Hive Query Language). Behind the scenes, Hive converts queries into MapReduce jobs, but it can be configured to use faster execution engines such as Apache Spark or Apache Tez. By leveraging standard query languages, Hive targeted a large  community that was already familiar with traditional database concepts and eliminated the need for Java programs. Nowadays, Hive is an open-source project maintained by Apache software foundation.

It's important to highlight that Hive was not designed to be an on-line transaction processing system (OLTP), characterized by frequent low-latency transactions (e.g. insert, update, delete). In fact, it's not even a database, but a user-friendly interface to query data stored in HDFS files. Consequently, it's suitable for data warehouse tasks, where large datasets are maintained for analytical purposes like data mining, data analysis, building reports, etc.

Hive's architecture, illustrated in Figure 1, comprises four major components:

- **Metastore:** required to store metadata (e.g. schema, location, etc) for tables. By default, Hive uses a embedded Derby SQL server that cannot handle multiple processes. As result of that, any local or remote RDBMS is recommended for a complete support.

- **Driver:** it's responsible for receiving, compiling (e.g. converting into MapReduce, Apache Spark or Apache Tez jobs), optimizing and executing the query. 

- **CLI:** most popular solution to interact with Hive via a provided command-line interface (CLI)

- **Thrift Server:** allows interacting with Hive using any programming language that supports this service, similarly to JDBC or ODBC protocols.


<p align="center">
<img src="https://github.com/thiago-a-souza/tmp/blob/master/img/hive_arch.png"  height="35%" width="35%"> <br>
Figure 1: Apache Hive architecture <a href="https://github.com/thiago-a-souza/tmp/blob/master/README.md#references">(1)</a> </p> 
</p>

## Data model

Hive organizes the data into four elements:

- **Database:** represents a namespace to group and uniquely identify objects (e.g. tables, views, etc). Every database creates under the warehouse directory - specified in the configuration file *$HIVE_HOME/conf/hive-site.xml*, which default warehouse directory is */user/hive/warehouse* in HDFS.
- **Table:** similar to database tables with columns, rows, etc. Every table creates a  subdirectory under the corresponding database directory.
- **Partition:** tables can be split into partitions for faster scans. Every partition creates a subdirectory under the corresponding table directory in HDFS.
- **Buckets:** tables can be further optimized with buckets based on the hash function of a given column.


## Data types

Hive provides primitive data types available in most relational databases, but also supports complex data types, which are not supported by traditional databases because they would break the normal form.  For a complete and up-to-date reference check out the language manual [(2)](#references).

**Primitive Data Types:** 

- **Numeric:** *double, float, bigint, int, smallint, tinyint*

- **Date/time:** *date, timestamp, interval*

- **Alphanumeric:** *char, varchar, string*

- **Boolean:** *boolean*

- **Binary:** *binary*

**Complex Data Types:** 

- **Array:** stores values of a given type and can be accessed using their index. 
Example: ARRAY< DOUBLE >


- **Maps:** stores key-value pairs and can be accessed using their keys. Example: *MAP<STRING, INT>*

- **Structs:** similar to structs from C/C++ and can be accessed using a dot notation. Example: *STRUCT<city:STRING, state:STRING>*

Unlike traditional databases, which have total control of the data being loaded and enforce that the table definitions are followed at the write time (a.k.a. schema-on-write), Hive does not control the data stored. Users can store files in HDFS and then create the table that references it, regardless of the data format. This behavior, called schema-on-read, can cause null values when they are not in conformance with table defitions, for example, when a column is defined as numeric but a string is found.

## Data Definition Language (DDL)
Similarly to RDBMS DDLs, Hive DDLs allow creating,  altering or dropping the structure of objects (e.g. databases, tables, columns, views, etc).

### Databases

Hive provides a standard database called *default*. Every new database creates a directory under the path defined in the configuration file *$HIVE_HOME/conf/hive-site.xml*, the default HDFS location is */user/hive/warehouse*.


Creating a new database in the standard directory:

```
hive> CREATE DATABASE hr;
hive> dfs -ls /user/hive/warehouse;
drwxrwxr-x   - root supergroup          0 2018-05-23 19:08 /user/hive/warehouse/hr.db
```

Creating a new database in a custom directory:

```
hive> CREATE DATABASE it
    > LOCATION '/user/thiago/it.db';
hive> dfs -ls /user/thiago;
drwxr-xr-x   - root supergroup          0 2018-05-23 19:44 /user/thiago/it.db
```

Dropping a database also removes the corresponding directory:

```
hive> DROP DATABASE it;
hive> dfs -ls /user/thiago;
```

Displaying all databases available:

```
hive> SHOW DATABASES;
default
hr
```

Switching to a specified database:

```
hive> USE hr;
```

### Tables


Hive provides two types of tables: internal and external. Internal tables are fully managed by Hive, meaning that when the table is dropped, the corresponding data is also removed. External tables are created using the *EXTERNAL* keyword in the *CREATE TABLE* statement. Because dropping external tables don't delete associated files, they are suitable for production environments to prevent removing files by accident. External tables are also used when the dataset is stored outside HDFS filesystem like Amazon S3.

By default, when a table is created, Hive creates a new folder under the current database, which is a subdirectory under */user/hive/warehouse*. However, the *LOCATION* keyword allows specifying an alternative path. This folder stores all files associated with the table. As a result, any file added, modified or removed from that directory change query outputs. In other words, data can be loaded by simply copying files (regardless of the file name) to the table directory, without having to run insert or load statements.

To support processing structured files, Hive adopts control characters (^A for columns, ^B for arrays/structs and ^C for Map key-value pairs) as default delimiters rather than pipes or commas to prevent errors in case these characters are in the fields. But it can also be customized to define alternative delimiters.


```
CREATE TABLE employees (
 id             INT,
 name           STRING,
 address        STRUCT<street:STRING, city:STRING, state:STRING>,
 phones         ARRAY<STRING>,
 languages_level MAP<STRING, STRING>
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '|'
COLLECTION ITEMS TERMINATED BY ','
MAP KEYS TERMINATED BY ':'
LINES TERMINATED BY '\n';
```

### Partitions

### Buckets

### Views
    
# References

(1) Capriolo, Edward et al. Programming Hive: Data warehouse and query language for Hadoop. O'Reilly Media, 2012.

(2) Hive Language Manual - https://cwiki.apache.org/confluence/display/Hive/LanguageManual
