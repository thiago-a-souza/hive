# Author
Thiago Alexandre Domingues de Souza

# Apache Hive
MapReduce programs allowed processing very large datasets that were not feasible before. However, developing solutions using this programming model can be challenging, taking hours to write algorithms for typical problems. As a result, Facebook introduced Hive, a data warehouse platform that can process petabytes worth of data using a syntax based on SQL called HQL (Hive Query Language). That way, they targeted the community that was already comfortable with traditional database concepts and eliminated the need for Java programs. Nowadays, Hive is an open-source project maintained by the Apache software foundation.

In a high level, the Apache Hive architecture can be divided into four components:

- **Metastore:** required to store metadata (e.g. schema, location, etc) for tables. By default, Hive uses a embedded Derby SQL server that cannot handle multiple processes. As result of that, any local or remote RDBMS is recommended for a complete  support.

- **Driver:** it's responsible for receiving, compiling (e.g. converting into MapReduce jobs), optimizing and executing the query. 

- **CLI:**

- **Thrift Server:**
