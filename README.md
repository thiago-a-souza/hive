# Author
Thiago Alexandre Domingues de Souza

# Apache Hive
MapReduce programs allowed processing very large datasets that were not feasible before. However, developing solutions using this programming model can be challenging, taking hours to write algorithms for typical problems. As a result, Facebook introduced Hive, a data warehouse platform that can process petabytes worth of data using a syntax based on SQL called HQL (Hive Query Language). Behind the scenes, Hive converts queries into MapReduce jobs, but it can be configured to use faster execution engines such as Apache Spark or Apache Tez. By leveraging standard query languages, Hive targeted a large  community that was already familiar with traditional database concepts and eliminated the need for Java programs. Nowadays, Hive is an open-source project maintained by Apache software foundation.

It's important to highlight that Hive was not designed to be an on-line transaction processing system (OLTP), characterized by frequent low-latency transactions (e.g. insert, update, delete). On the contrary, it's suitable for data warehouse tasks, where large volumes of data are maintained for analytical purposes like data mining, data analysis, building reports, etc.

Hive's architecture, illustrated in Figure 1, comprises four major components:

- **Metastore:** required to store metadata (e.g. schema, location, etc) for tables. By default, Hive uses a embedded Derby SQL server that cannot handle multiple processes. As result of that, any local or remote RDBMS is recommended for a complete support.

- **Driver:** it's responsible for receiving, compiling (e.g. converting into MapReduce, Apache Spark or Apache Tez jobs), optimizing and executing the query. 

- **CLI:** most popular solution to interact with Hive via a provided command-line interface (CLI)

- **Thrift Server:** allows interacting with Hive using any programming language that supports this service, similarly to JDBC or ODBC protocols.


<p align="center">
<img src="https://github.com/thiago-a-souza/tmp/blob/master/img/hive_arch.png"  height="35%" width="35%"> <br>
Figure 1: Apache Hive architecture <a href="https://github.com/thiago-a-souza/tmp/blob/master/README.md#references">(1)</a> </p> 
</p>

# References

(1) Capriolo, Edward et al. Programming Hive: Data warehouse and query language for Hadoop. O'Reilly Media, 2012.
