---
title: "SQL DDL"
weight: 2
type: docs
aliases:
- /spark/sql-ddl.html
---
<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

# SQL DDL

## Catalog

### Create Catalog

Paimon catalogs currently support three types of metastores:

* `filesystem` metastore (default), which stores both metadata and table files in filesystems.
* `hive` metastore, which additionally stores metadata in Hive metastore. Users can directly access the tables from Hive.
* `jdbc` metastore, which additionally stores metadata in relational databases such as MySQL, Postgres, etc.

See [CatalogOptions]({{< ref "maintenance/configurations#catalogoptions" >}}) for detailed options when creating a catalog.

#### Create Filesystem Catalog

The following Spark SQL registers and uses a Paimon catalog named `my_catalog`. Metadata and table files are stored under `hdfs:///path/to/warehouse`.

The following shell command registers a paimon catalog named `paimon`. Metadata and table files are stored under `hdfs:///path/to/warehouse`.

```bash
spark-sql ... \
    --conf spark.sql.catalog.paimon=org.apache.paimon.spark.SparkCatalog \
    --conf spark.sql.catalog.paimon.warehouse=hdfs:///path/to/warehouse
```

You can define any default table options with the prefix `spark.sql.catalog.paimon.table-default.` for tables created in the catalog.

After `spark-sql` is started, you can switch to the `default` database of the `paimon` catalog with the following SQL.

```sql
USE paimon.default;
```

#### Creating Hive Catalog

By using Paimon Hive catalog, changes to the catalog will directly affect the corresponding Hive metastore. Tables created in such catalog can also be accessed directly from Hive.

To use Hive catalog, Database name, Table name and Field names should be **lower** case.

Your Spark installation should be able to detect, or already contains Hive dependencies. See [here](https://spark.apache.org/docs/latest/sql-data-sources-hive-tables.html) for more information.

The following shell command registers a Paimon Hive catalog named `paimon`. Metadata and table files are stored under `hdfs:///path/to/warehouse`. In addition, metadata is also stored in Hive metastore.

```bash
spark-sql ... \
    --conf spark.sql.catalog.paimon=org.apache.paimon.spark.SparkCatalog \
    --conf spark.sql.catalog.paimon.warehouse=hdfs:///path/to/warehouse \
    --conf spark.sql.catalog.paimon.metastore=hive \
    --conf spark.sql.catalog.paimon.uri=thrift://<hive-metastore-host-name>:<port>
```

You can define any default table options with the prefix `spark.sql.catalog.paimon.table-default.` for tables created in the catalog.

After `spark-sql` is started, you can switch to the `default` database of the `paimon` catalog with the following SQL.

```sql
USE paimon.default;
```

Also, you can create [SparkGenericCatalog]({{< ref "spark/quick-start" >}}).

**Synchronizing Partitions into Hive Metastore**

By default, Paimon does not synchronize newly created partitions into Hive metastore. Users will see an unpartitioned table in Hive. Partition push-down will be carried out by filter push-down instead.

If you want to see a partitioned table in Hive and also synchronize newly created partitions into Hive metastore, please set the table property `metastore.partitioned-table` to true. Also see [CoreOptions]({{< ref "maintenance/configurations#coreoptions" >}}).

#### Creating JDBC Catalog

By using the Paimon JDBC catalog, changes to the catalog will be directly stored in relational databases such as SQLite, MySQL, postgres, etc.

Currently, lock configuration is only supported for MySQL and SQLite. If you are using a different type of database for catalog storage, please do not configure `lock.enabled`.

Paimon JDBC Catalog in Spark needs to correctly add the corresponding jar package for connecting to the database. You should first download JDBC  connector bundled jar and add it to classpath. such as MySQL, postgres

| database type | Bundle Name          | SQL Client JAR                                                             |
|:--------------|:---------------------|:---------------------------------------------------------------------------|
| mysql         | mysql-connector-java | [Download](https://mvnrepository.com/artifact/mysql/mysql-connector-java)  |
| postgres      | postgresql           | [Download](https://mvnrepository.com/artifact/org.postgresql/postgresql)   |

```bash
spark-sql ... \
    --conf spark.sql.catalog.paimon=org.apache.paimon.spark.SparkCatalog \
    --conf spark.sql.catalog.paimon.warehouse=hdfs:///path/to/warehouse \
    --conf spark.sql.catalog.paimon.metastore=jdbc \
    --conf spark.sql.catalog.paimon.uri=jdbc:mysql://<host>:<port>/<databaseName> \
    --conf spark.sql.catalog.paimon.jdbc.user=... \
    --conf spark.sql.catalog.paimon.jdbc.password=...
    
```

```sql
USE paimon.default;
```
#### Creating REST Catalog

By using the Paimon REST catalog, changes to the catalog will be directly stored in remote server.

##### bear token
```bash
spark-sql ... \
    --conf spark.sql.catalog.paimon=org.apache.paimon.spark.SparkCatalog \
    --conf spark.sql.catalog.paimon.metastore=rest \
    --conf spark.sql.catalog.paimon.uri=<catalog server url> \
    --conf spark.sql.catalog.paimon.token.provider=bear \
    --conf spark.sql.catalog.paimon.token=<token>
    
```

##### dlf ak
```bash
spark-sql ... \
    --conf spark.sql.catalog.paimon=org.apache.paimon.spark.SparkCatalog \
    --conf spark.sql.catalog.paimon.metastore=rest \
    --conf spark.sql.catalog.paimon.uri=<catalog server url> \
    --conf spark.sql.catalog.paimon.token.provider=dlf \
    --conf spark.sql.catalog.paimon.dlf.access-key-id=<access-key-id> \
    --conf spark.sql.catalog.paimon.dlf.access-key-secret=<security-token>
    
```

##### dlf sts token
```bash
spark-sql ... \
    --conf spark.sql.catalog.paimon=org.apache.paimon.spark.SparkCatalog \
    --conf spark.sql.catalog.paimon.metastore=rest \
    --conf spark.sql.catalog.paimon.uri=<catalog server url> \
    --conf spark.sql.catalog.paimon.token.provider=dlf \
    --conf spark.sql.catalog.paimon.dlf.access-key-id=<access-key-id> \
    --conf spark.sql.catalog.paimon.dlf.access-key-secret=<access-key-secret> \
    --conf spark.sql.catalog.paimon.dlf.security-token=<security-token>
    
    
```

```sql
USE paimon.default;
```

## Table

### Create Table

After use Paimon catalog, you can create and drop tables. Tables created in Paimon Catalogs are managed by the catalog.
When the table is dropped from catalog, its table files will also be deleted.

The following SQL assumes that you have registered and are using a Paimon catalog. It creates a managed table named 
`my_table` with five columns in the catalog's `default` database, where `dt`, `hh` and `user_id` are the primary keys.

```sql
CREATE TABLE my_table (
    user_id BIGINT,
    item_id BIGINT,
    behavior STRING,
    dt STRING,
    hh STRING
) TBLPROPERTIES (
    'primary-key' = 'dt,hh,user_id'
);
```

You can create partitioned table:

```sql
CREATE TABLE my_table (
    user_id BIGINT,
    item_id BIGINT,
    behavior STRING,
    dt STRING,
    hh STRING
) PARTITIONED BY (dt, hh) TBLPROPERTIES (
    'primary-key' = 'dt,hh,user_id'
);
```

### Create External Table

When the catalog's `metastore` type is `hive`, if the `location` is specified when creating a table, that table will be considered an external table; otherwise, it will be a managed table. 

When you drop an external table, only the metadata in Hive will be removed, and the actual data files will not be deleted; whereas dropping a managed table will also delete the data.

```sql
CREATE TABLE my_table (
    user_id BIGINT,
    item_id BIGINT,
    behavior STRING,
    dt STRING,
    hh STRING
) PARTITIONED BY (dt, hh) TBLPROPERTIES (
    'primary-key' = 'dt,hh,user_id'
) LOCATION '/path/to/table';
```

Furthermore, if there is already data stored in the specified location, you can create the table without explicitly specifying the fields, partitions and props or other information. 
In this case, the new table will inherit them all from the existing table’s metadata. 

However, if you manually specify them, you need to ensure that they are consistent with those of the existing table (props can be a subset). Therefore, it is strongly recommended not to specify them.

```sql
CREATE TABLE my_table LOCATION '/path/to/table';
```

### Create Table As Select

Table can be created and populated by the results of a query, for example, we have a sql like this: `CREATE TABLE table_b AS SELECT id, name FORM table_a`,
The resulting table `table_b` will be equivalent to create the table and insert the data with the following statement:
`CREATE TABLE table_b (id INT, name STRING); INSERT INTO table_b SELECT id, name FROM table_a;`

We can specify the primary key or partition when use `CREATE TABLE AS SELECT`, for syntax, please refer to the following sql.

```sql
CREATE TABLE my_table (
     user_id BIGINT,
     item_id BIGINT
);
CREATE TABLE my_table_as AS SELECT * FROM my_table;

/* partitioned table*/
CREATE TABLE my_table_partition (
      user_id BIGINT,
      item_id BIGINT,
      behavior STRING,
      dt STRING,
      hh STRING
) PARTITIONED BY (dt, hh);
CREATE TABLE my_table_partition_as PARTITIONED BY (dt) AS SELECT * FROM my_table_partition;

/* change TBLPROPERTIES */
CREATE TABLE my_table_options (
       user_id BIGINT,
       item_id BIGINT
) TBLPROPERTIES ('file.format' = 'orc');
CREATE TABLE my_table_options_as TBLPROPERTIES ('file.format' = 'parquet') AS SELECT * FROM my_table_options;


/* primary key */
CREATE TABLE my_table_pk (
     user_id BIGINT,
     item_id BIGINT,
     behavior STRING,
     dt STRING,
     hh STRING
) TBLPROPERTIES (
    'primary-key' = 'dt,hh,user_id'
);
CREATE TABLE my_table_pk_as TBLPROPERTIES ('primary-key' = 'dt') AS SELECT * FROM my_table_pk;

/* primary key + partition */
CREATE TABLE my_table_all (
    user_id BIGINT,
    item_id BIGINT,
    behavior STRING,
    dt STRING,
    hh STRING
) PARTITIONED BY (dt, hh) TBLPROPERTIES (
    'primary-key' = 'dt,hh,user_id'
);
CREATE TABLE my_table_all_as PARTITIONED BY (dt) TBLPROPERTIES ('primary-key' = 'dt,hh') AS SELECT * FROM my_table_all;
```

## View

Views are based on the result-set of an SQL query, when using `org.apache.paimon.spark.SparkCatalog`, views are managed by paimon itself. 
And in this case, views are supported when the `metastore` type is `hive`, and temporary views are not supported yet.

### Create Or Replace View

CREATE VIEW constructs a virtual table that has no physical data.

```sql
-- create a view.
CREATE VIEW v1 AS SELECT * FROM t1;

-- create a view, if a view of same name already exists, it will be replaced.
CREATE OR REPLACE VIEW v1 AS SELECT * FROM t1;
```

### Drop View

DROP VIEW removes the metadata associated with a specified view from the catalog.

```sql
-- drop a view
DROP VIEW v1;
```

## Tag
### Create Or Replace Tag
Create or replace a tag syntax with the following options.
- Create a tag with or without the snapshot id and time retention.
- Create an existed tag is not failed if using `IF NOT EXISTS` syntax.
- Update a tag using `REPLACE TAG` or `CREATE OR REPLACE TAG` syntax.

```sql
-- create a tag based on the latest snapshot and no retention.
ALTER TABLE T CREATE TAG `TAG-1`;

-- create a tag based on the latest snapshot and no retention if it doesn't exist.
ALTER TABLE T CREATE TAG IF NOT EXISTS `TAG-1`;

-- create a tag based on the latest snapshot and retain it for 7 day.
ALTER TABLE T CREATE TAG `TAG-2` RETAIN 7 DAYS;

-- create a tag based on snapshot-1 and no retention.
ALTER TABLE T CREATE TAG `TAG-3` AS OF VERSION 1;

-- create a tag based on snapshot-2 and retain it for 12 hour.
ALTER TABLE T CREATE TAG `TAG-4` AS OF VERSION 2 RETAIN 12 HOURS;

-- replace a existed tag with new snapshot id and new retention
ALTER TABLE T REPLACE TAG `TAG-4` AS OF VERSION 2 RETAIN 24 HOURS;

-- create or replace a tag, create tag if it not exist, replace tag if it exists.
ALTER TABLE T CREATE OR REPLACE TAG `TAG-5` AS OF VERSION 2 RETAIN 24 HOURS;
```
NOTE: If tag.automatic-creation is set, only one auto-tag could be created for one snapshot.

### Delete Tag
Delete a tag or multiple tags of a table.
```sql
-- delete a tag.
ALTER TABLE T DELETE TAG `TAG-1`;

-- delete a tag if it exists.
ALTER TABLE T DELETE TAG IF EXISTS `TAG-1`

-- delete multiple tags, delimiter is ','.
ALTER TABLE T DELETE TAG `TAG-1,TAG-2`;
```

### Rename Tag
Rename an existing tag with a new tag name.
```sql
ALTER TABLE T RENAME TAG `TAG-1` TO `TAG-2`;
```

### Show Tags
List all tags of a table.
```sql
SHOW TAGS T;
```

