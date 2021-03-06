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
<img src="https://github.com/thiago-a-souza/hive/blob/master/img/hive_arch.png"  height="35%" width="35%"> <br>
Figure 1: Apache Hive architecture <a href="https://github.com/thiago-a-souza/hive/blob/master/README.md#references">(1)</a> </p> 
</p>

## Data model

Hive organizes the data into four elements:

- **Database:** represents a namespace to group and uniquely identify objects (e.g. tables, views, etc). Every database creates a folder under the warehouse directory - specified in the configuration file *$HIVE_HOME/conf/hive-site.xml*, which default warehouse directory is */user/hive/warehouse* in HDFS.
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

Unlike traditional databases, which have total control of the data being loaded and enforce that the table definitions are followed at the write time (a.k.a. schema-on-write), Hive does not control the data stored. Users can store files in HDFS and then create the table that references it, regardless of the data format. This behavior, called schema-on-read, can cause null values when they are not in conformance with table definitions, for example, when a column is defined as numeric but a string is found.

## File Formats

Hive supports a wide range of file formats such as *TEXTFILE*, *SEQUENCEFILE*, *ORC*, *PARQUET* and *AVRO*. Choosing an appropriate file format can be challenging and involves several aspects: read/write performance, ability to split files,  compression support and garbage collection . As a result, the tradeoff between performance and storage usually drives the final decision for a particular dataset.

In general, file formats can be divided into two categories: row level and column level. Row level formats store all the data of an individual row in a contiguous disk region. This behaviour enforces processing all columns without avoiding unnecessary data. On the other hand, storing data in a columnar format is more efficient in terms of size and performance. This format allows choosing a suitable encoding according to the column type, reducing the file size. In addition to that, it can skip not requested columns, which results in a better performance.

The optional *STORED AS* clause specifies the file format when the table is created. By default, tables are stored as *TEXTFILE*, but it can be modified in the *hive-site.xml* by setting the property *hive.default.fileformat*. Once created, the  format can be modified using the *ALTER TABLE* statement, but it doesn't change the data.

- **TEXTFILE:** this row level storage format is the default option in Hive. Data can be compressed but not recommended because it does not allow splitting files, impacting parallel processing. Also, this format requires reading the entire row regardless of the columns requested. These factors make this format very slow for reading/writing data, as result it's not recommended for large datasets.

- **SEQUENCEFILE:** row level format designed to store binary key/value pairs for Hadoop's MapReduce. By allowing compressed files on block or record level, it enables splitting files for parallel processing. Although it's better than *TEXTFILE*, this is not a good alternative for Hive because row level formats impact performance.

- **ORC:** column level format designed as an improvement from RCFile to speed up Hive. It's a splittable format that uses encoders according to data types to enhance the compression ratio. For a better performance, it stores metadata like min, max, sum, count and built-in indexes. In addition to that, ORC supports ACID transactions, although it was not built to meet OLTP requirements.

- **PARQUET:** column level format created by Twitter and Cloudera based on Google's Dremel paper. Parquet chooses encodings based on the column type and the  data can be compressed using a specified algorithm (e.g. Snappy, Zlib, LZO) - data is not compressed by default. PARQUET also stores file schema and metadata for performance purposes.

- **AVRO:** this is a data serialization system that makes it easier to exchange large datasets between programs. It creates schemas using a JSON format while the data is serialized in a binary format. Additionally, Avro is a row level format that supports RPC calls.


## Data Definition Language (DDL)
Similarly to RDBMS DDLs, Hive allows creating,  altering or dropping the structure of objects (e.g. databases, tables, columns, views, etc). 

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


Hive provides two types of tables: internal and external. Internal tables are fully managed by Hive, meaning that when a table is dropped, the corresponding data is also removed. External tables are created using the *EXTERNAL* keyword in the *CREATE TABLE* statement. Because dropping external tables don't delete associated files, they are suitable for production environments to prevent removing files by accident. External tables are also used when the dataset is stored outside HDFS filesystem like Amazon S3.

By default, when a table is created, Hive creates a new folder under the current database, which is a subdirectory under */user/hive/warehouse*. However, the *LOCATION* keyword allows specifying an alternative path. This folder stores all files associated with the table. As a result, any file added, modified or removed from that directory change query outputs. In other words, data can be loaded by simply copying files (regardless of the file name) to the table directory, without having to run insert or load statements.

To support processing structured files, Hive adopts control characters (^A for columns, ^B for arrays/structs and ^C for Map key-value pairs) as default delimiters rather than pipes or commas to prevent errors in case these characters are in the fields. But it can also be customized to define alternative delimiters.

**a) Internal table with default settings**

File *emp_std.txt* using standard delimiters:

```
1^Ajohn^A50 Hagiwara Tea Garden Dr^BSan Francisco^BCA^A111-222-3333^B444-555-8888^Aenglish^Cfluent^Bspanish^Cbasic^A100000^A3
2^Apeter^A700 Exposition Park Dr^BLos Angeles^BCA^A222-111-3333^B777-555-4444^Aenglish^Cfluent^Bfrench^Cadvanced^Bspanish^Cintermediate^A80000^A3
3^Ahans^A79th St^BNew York^BNY^A789-012-3456^B999-000-1111^Agerman^Cfluent^Benglish^Cadvanced^A120000^A1
4^Ajacques^A1000 5th Ave^BNew York^BNY^A888-111-4444^Afrench^Cfluent^Benglish^Cadvanced^A90000^A2
5^Afrancesca^A125 I St^BSacramento^BCA^A777-222-3333^Aitalian^Cfluent^Benglish^Cbasic^A85000
```

Creating an internal table using default settings, verifying the folder created and moving *emp_std.txt* to the table folder:

```
hive> CREATE TABLE hr.employees_std (
      id              INT,
      name            STRING,
      address         STRUCT<street:STRING, city:STRING, state:STRING>,
      phones          ARRAY<STRING>,
      languages_level MAP<STRING, STRING>,
      salary          DOUBLE,
      dept_id         INT
      );
    
hive> dfs -ls /user/hive/warehouse/hr.db;
drwxrwxr-x   - root supergroup          0 2018-05-24 15:27 /user/hive/warehouse/hr.db/employees

hive> dfs -put emp_std.txt /user/hive/warehouse/hr.db/employees_std;
```

Dropping internal tables remove metadata from metastore as well as the table folder and all files:

```
hive> dfs -ls -R  /user/hive/warehouse/hr.db;
drwxrwxr-x   - root supergroup          0 2018-05-24 17:56 /user/hive/warehouse/hr.db/employees_std
-rw-r--r--   1 root supergroup        469 2018-05-24 17:56 /user/hive/warehouse/hr.db/employees_std/emp_std.txt
hive> drop table hr.employees_std;
hive> dfs -ls -R  /user/hive/warehouse/hr.db;
hive> 
```

**b) Internal table with customized settings**

File *emp_cust.txt* using customized delimiters:

```
1|john|50 Hagiwara Tea Garden Dr,San Francisco,CA|111-222-3333,444-555-8888|english:fluent,spanish:basic|100000|3
2|peter|700 Exposition Park Dr,Los Angeles,CA|222-111-3333,777-555-4444|english:fluent,french:advanced,spanish:intermediate|80000|3
3|hans|79th St,New York,NY|789-012-3456,999-000-1111|german:fluent,english:advanced|120000|1
4|jacques|1000 5th Ave,New York,NY|888-111-4444|french:fluent,english:advanced|90000|2
5|francesca|125 I St,Sacramento,CA|777-222-3333|italian:fluent,english:basic|85000
```

Creating an alternative folder, copying the file to the new folder and creating a customized table:

```
hive> dfs -mkdir -p /user/thiago/employees_cust;
hive> dfs -put emp_cust.txt /user/thiago/employees_cust;

hive> CREATE TABLE hr.employees_cust (
      id             INT,
      name           STRING,
      address        STRUCT<street:STRING, city:STRING, state:STRING>,
      phones         ARRAY<STRING>,
      languages_level MAP<STRING, STRING>,
      salary          DOUBLE,
      dept_id         INT      
     )
     ROW FORMAT DELIMITED
     FIELDS TERMINATED BY '|'
     COLLECTION ITEMS TERMINATED BY ','
     MAP KEYS TERMINATED BY ':'
     LINES TERMINATED BY '\n'
     LOCATION '/user/thiago/employees_cust';
```

**c) External table**

Creating a folder, moving the file and creating an external table:

```
hive> dfs -mkdir -p /user/thiago/employees_external;
hive> dfs -put emp_ext.txt /user/thiago/employees_external;

hive> CREATE EXTERNAL TABLE hr.employees_ext (
      id              INT,
      name            STRING,
      address         STRUCT<street:STRING, city:STRING, state:STRING>,
      phones          ARRAY<STRING>,
      languages_level MAP<STRING, STRING>,
      salary          DOUBLE,
      dept_id         INT      
     )
     LOCATION '/user/thiago/employees_external';
```

Dropping external tables won't remove folders or files:

```
hive> drop table hr.employees_ext;
hive> dfs -ls -R /user/thiago/employees_external;
-rw-r--r--   1 root supergroup        469 2018-05-24 18:08 /user/thiago/employees_external/emp_ext.txt
```


### Partitions

In RDBMS world, partitions can segment tables based on a column for performance purposes. Hive follows that concept by creating subdirectories under the table's folder. That way, each subdirectory stores files for the corresponding  partition. Hive uses a different syntax to declare partitions, the column being partitioned cannot be in the *CREATE TABLE* statement because it creates columns specified in the *PARTITION BY* clause.

Once the partition is declared in the table structure, the actual partition values should be added to the table. The *LOAD* statement can add them if the partition names and values are provided. Alternatively, the *ALTER TABLE* statement can also add partition values.


**a) Creating partitions with LOAD**

Notice that creating a partitioned table only creates the table's folder. After running the *LOAD* statement, we can see that a partition and a subdirectory *country=USA* was created.

```
hive> CREATE TABLE hr.employees_partitioned (
      id              INT,
      name            STRING,
      address         STRUCT<street:STRING, city:STRING, state:STRING>,
      phones          ARRAY<STRING>,
      languages_level MAP<STRING, STRING>,
      salary          DOUBLE,
      dept_id         INT      
     ) 
     PARTITIONED BY (country STRING);
     
hive> dfs -ls -R /user/hive/warehouse/hr.db;
drwxrwxr-x   - root supergroup          0 2018-05-24 19:48 /user/hive/warehouse/hr.db/employees_partitioned
hive> show partitions hr.employees_partitioned;

hive> LOAD DATA LOCAL INPATH '/staging/emp_part_usa.txt' 
      INTO TABLE hr.employees_partitioned
      PARTITION (country='USA');
hive> show partitions hr.employees_partitioned;
country=USA
    
hive> dfs -ls -R /user/hive/warehouse/hr.db/employees_partitioned;
drwxrwxr-x   - root supergroup          0 2018-05-24 19:08 /user/hive/warehouse/hr.db/employees_partitioned/country=USA
-rwxrwxr-x   1 root supergroup        469 2018-05-24 19:08 /user/hive/warehouse/hr.db/employees_partitioned/country=USA/emp_part_usa.txt     
```

**b) Creating partitions with ALTER**

Instead of using *LOAD*, we can *ALTER* the table to add the partition and then copy the data to the partition.

```
hive> ALTER TABLE hr.employees_partitioned ADD
      PARTITION (country='BRA');
hive> show partitions hr.employees_partitioned;
country=BRA
country=USA
hive> dfs -ls -R /user/hive/warehouse/hr.db/employees_partitioned;
drwxrwxr-x   - root supergroup          0 2018-05-24 19:17 /user/hive/warehouse/hr.db/employees_partitioned/country=BRA
drwxrwxr-x   - root supergroup          0 2018-05-24 19:08 /user/hive/warehouse/hr.db/employees_partitioned/country=USA
-rwxrwxr-x   1 root supergroup        469 2018-05-24 19:08 /user/hive/warehouse/hr.db/employees_partitioned/country=USA/emp_part_usa.txt
hive> dfs -put emp_part_bra.txt /user/hive/warehouse/hr.db/employees_partitioned/country=BRA;
```


### Buckets

Partitions are powerful when a small number of partitions hold a lot of data per partition. In contrast, bucketing allows segmenting  tables when a column has many different values but not many rows per key. This method hashes values of  bucketed columns into user-defined buckets, enabling more efficient queries. In particular, joins between tables that are bucketed on the same column take advantage of this approach.

Populating bucket tables correctly can be performed using an *INSERT* statement. The *LOAD* command does not populate bucket tables correctly because it doesn't verify the metadata, it simply copies/moves files to the corresponding table folder. In addition to that, populating bucket tables also requires either enabling the parameter *hive.enforce.bucketing* or setting the option  *mapred.reduce.tasks* to match the number of buckets.

The following example creates a bucket table, enables the parameter *hive.enforce.bucketing* and the populates the table. Notice that three files are created under the table's folder.

```
hive> CREATE TABLE hr.employees_bucket (
      id              INT,
      name            STRING,
      address         STRUCT<street:STRING, city:STRING, state:STRING>,
      phones          ARRAY<STRING>,
      languages_level MAP<STRING, STRING>,
      salary          DOUBLE,
      dept_id         INT      
     ) 
     CLUSTERED BY (id) INTO 3 BUCKETS;

hive> set hive.enforce.bucketing = true;

hive> insert into hr.employees_bucket
      select 
       id, name, address, phones, languages_level 
      from hr.employees_partitioned;
      
hive> dfs -ls -R /user/hive/warehouse/hr.db/employees_bucket;
-rwxrwxr-x   1 root supergroup         56 2018-05-25 15:33 /user/hive/warehouse/hr.db/employees_bucket/000000_0
-rwxrwxr-x   1 root supergroup        184 2018-05-25 15:33 /user/hive/warehouse/hr.db/employees_bucket/000001_0
-rwxrwxr-x   1 root supergroup        201 2018-05-25 15:33 /user/hive/warehouse/hr.db/employees_bucket/000002_0
```

### Views

Similarly to relational database views, Hive makes it easier to hide complex queries into logical views. 

```
hive> CREATE VIEW hr.employee_eng_level AS
      SELECT 
       name, languages_level['english'] 
      FROM employees_partitioned;
```

## Data Manipulation Language (DML)

### LOAD

The *LOAD* statement simply copies/moves files to the table folder. If the *LOCAL* keyword is provided, the file should be in the local filesystem. If omitted, the file should be in HDFS. *LOAD* is particularly handy when populating partitioned tables since it creates the partition metadata and the corresponding folder.

**Syntax:**

```
LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename
```


**a) Loading from HDFS**


Loading from HDFS moves the data from the source HDFS directory to the table's folder.

```
hive> dfs -ls -R /user/thiago;
-rw-r--r--   1 root supergroup        469 2018-05-25 18:00 /user/thiago/emp.txt

hive> LOAD DATA INPATH '/user/thiago/emp.txt' 
      OVERWRITE INTO TABLE hr.employees;

hive> dfs -ls -R /user/hive/warehouse/hr.db/employees;
-rwxrwxr-x   1 root supergroup        469 2018-05-25 18:00 /user/hive/warehouse/hr.db/employees/emp.txt
```

**b) Loading from local filesystem**

The *LOCAL* keyword copies the file from the local filesystem rather than moving.

```
hive> LOAD DATA LOCAL INPATH '/staging/data/emp2.txt' 
      INTO TABLE hr.employees;
hive> dfs -ls -R /user/hive/warehouse/hr.db/employees;
-rwxrwxr-x   1 root supergroup        469 2018-05-25 18:00 /user/hive/warehouse/hr.db/employees/emp.txt
-rwxrwxr-x   1 root supergroup        469 2018-05-25 18:27 /user/hive/warehouse/hr.db/employees/emp2.txt
```

### INSERT

Similarly to RDBMS inserts, it allows inserting individual rows or sourcing from existing tables. In addition to that, the data can be inserted into a specific HDFS directory.

**a) Insert from existing table**

```
hive> CREATE TABLE hr.name_freq (
       name  STRING,
       freq  INT
      );
      
hive> INSERT INTO hr.name_freq
      SELECT 
       name
       ,count(*)
      FROM employees
      GROUP BY name;
```

**b) Insert values**

Unlike SQL standards, inserting values require the keyword *TABLE*.

```
hive> CREATE TABLE example
      (id int,
       name string
      );
hive> INSERT INTO TABLE 
      example
      values (1, 'one'), (2, 'two'), (3, 'three');
```    

**c) Insert into directory**

```
hive> dfs -mkdir /user/thiago/emp_names;
hive> INSERT OVERWRITE DIRECTORY '/user/thiago/emp_names'
      select distinct name from employees;
hive> dfs -ls -R /user/thiago/emp_names;
-rwxr-xr-x   1 root supergroup         34 2018-05-25 19:19 /user/thiago/emp_names/000000_0
```

### UPDATE

Hive updates are restricted to tables that support transactions, which is not enabled by default.

**Syntax:**

```
UPDATE table
SET column = value
[WHERE exp];
```

### DELETE

Hive deletes are restricted to tables that support transactions, which is not enabled by default.

**Syntax:**

```
DELETE table
[WHERE exp];
```

## Queries

In general, Hive queries are similar to standard SQL. There are some limitations like supporting only equi-joins and not allowing *OR* operators in *ON* clauses. Despite these constraints, Hive supports most commonly used operators, functions and clauses available in standard SQL.

Unlike relational databases, Hive supports complex data types (a.k.a. arrays, structs and maps), and their values are displayed in a JSON format. It's also possible to access a specific value from arrays using a 0-based index, structs using a dot notation or maps using a key - references to values not found return *NULL* rather than an error. In addition to *LIKE*, Hive supports regular expressions with *RLIKE*.



**a) Simple query**

```
hive> SELECT 
       name, address, phones, languages_level 
      FROM employees;
john       {"street":"50 Hagiwara Tea Garden Dr","city":"San Francisco","state":"CA"}  ["111-222-3333","444-555-8888"]  {"english":"fluent","spanish":"basic"}
peter      {"street":"700 Exposition Park Dr","city":"Los Angeles","state":"CA"}       ["222-111-3333","777-555-4444"]  {"english":"fluent","french":"advanced","spanish":"intermediate"}
hans       {"street":"79th St","city":"New York","state":"NY"}                         ["789-012-3456","999-000-1111"]  {"german":"fluent","english":"advanced"}
jacques    {"street":"1000 5th Ave","city":"New York","state":"NY"}                    ["888-111-4444"]                 {"french":"fluent","english":"advanced"}
francesca  {"street":"125 I St","city":"Sacramento","state":"CA"}                      ["777-222-3333"]                 {"italian":"fluent","english":"basic"}
```

**b) Accessing collections**
```
hive> SELECT
       name,
       address.state,
       phones[0],
       languages_level['english']
      from employees;
john      CA 111-222-3333 fluent
peter     CA 222-111-3333 fluent
hans      NY 789-012-3456 advanced
jacques   NY 888-111-4444 advanced
francesca CA 777-222-3333 basic
```

**c) Using regular expressions**

```
hive> SELECT
       name, address.city
      FROM employees
      WHERE address.city RLIKE '(Los Angeles|San Francisco)';
john    San Francisco
peter   Los Angeles
```

**d) Query with group function**

```
hive> SELECT 
       SUM(salary), dept_id 
      FROM employees 
      GROUP BY dept_id 
      HAVING SUM(salary) > 100000 
      ORDER BY 2 DESC;
180000.0   3
120000.0   1
```

**e) Inner Join**

```
hive> SELECT
       e.name as emp_name,
       d.name as dept_name
      FROM departments d
      JOIN employees e
      ON d.id = e.dept_id;
john    Sales
peter   Sales
hans    IT
jacques Marketing
```

**f) Left Outer Join**

```
hive> SELECT 
       d.name, count(e.dept_id)
      FROM departments d
      LEFT OUTER JOIN employees e
      ON d.id = e.dept_id
      GROUP BY d.name;
IT         1
Legal      0
Marketing  1
Sales      2    
```

    
# References

(1) Capriolo, Edward et al. Programming Hive: Data warehouse and query language for Hadoop. O'Reilly Media, 2012.

(2) Hive Language Manual - https://cwiki.apache.org/confluence/display/Hive/LanguageManual
