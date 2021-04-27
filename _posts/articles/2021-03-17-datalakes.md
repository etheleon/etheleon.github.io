---
layout: post
title: "Not so big queries, hitchhiker’s guide to datawarehousing with datalakes with Spark"
excerpt: "Back to the future: Revisiting HIVE metastore, datalakes and spark"
categories: articles
tags: [spark]
author: wesley_goi
comments: true
share: true
modified: 2021-03-17
mathjax: true
image:
  feature: sparklyr_joy_png.png
  thumb: sparklyr_joy_png.png
  credit: Wesley GOI
  creditlink: http://etheleon.github.io
---


# Introduction
Since the [earlier post](https://etheleon.github.io/articles/spark-joy/), I’ve defended my thesis and moved on from Honestbee to an AirAsia but covid hit and now I’ve found myself in ride hailing / superapp Grab. Time passes quickly. 

> The following post does not represent my employer and reflects my own personal views only. 

Once again, spark makes a return, after using GCP’s BigQuery for a year, as the tool of choice carrying out most of my ETLs as a data scientist.

As a data scientist, I’ve retained mental model for carrying out data analysis and ETL thankfully spark SQL. What did changed is the visible decoupling of (1) storage, (2) query engine and (3) metastore 

Instead of colossus there is S3, where there was dremel you have either presto or spark and table metadata is stored in HIVE MetaStore (HMS) SQL is the only tool which has remained largely unchanged when moving from BQ to this new metadata+file system+QueryEngine(spark|presto) combination. 

> In my opinion this has further solidified **SQL as king** since it is the common tongue amongst data practitioners: Engineers, Scientists and Analysts.

Testing queries and quick and dirty data exploration can done using presto without spinning up a cluster. There’s often many choices to choose from for good SQL workbenches with good formatter and column exploration. 

However if you need to hook this up with an in-depth analysis / feature engineering + model building, basically anything which requires me to fire up a notebook, I’ll switch over to spark. Common team is cluster management is a full time task and should either be outsourced to a 3rd party like Qubole, cloud provider eg Dataproc or managed by a in house team. 

> NOTE: There is no need to parse all data neatly into columns due to the velocity and volume eg. logs and events. Only evaluate when you need, lazy eval. A good example of this is the [GA to BQ export](https://support.google.com/analytics/answer/3416092#zippy=%2Cin-this-article)

When using BigQuery it also became apparent sharing data via tables/views is far superior over files eg. CSV or a google sheets due ue to security and the lack thereof in ability to audit the access of data. 

Similar to BQ, spark allows one to store the the results into a table whose read and query access can be managed securely. Sharing of queries and their requests (either as individual queries or a table is exactly the same as BigQuery. I’ll highly recommend the use of Alation as the web client and it supports enterprise HIVE. Sadly one con is there is no way query against presto views when using spark as your query engine. 

The rest of this post highlights how one would transition from a GCP set-up where everything is managed by HMS+S3+QueryEngine(sparkSQL) which is common for companies running on AWS. 

# SparkSession
Sadly the use of R has also gradually decreased for me except for plotting with GGPLOT2. We will be using PySpark in this tutorial. 

Similar to R’s `sparklyr`  package we create a spark connection of type `SparkSession` (it is often aliased as `spark`) in PySpark

Since spark 2.X, **Spark Session** unifies the **Spark Context** and **Hive Context** classes into a single interface. Its use is recommended over the older APIs for code targeting Spark 2.0.0 and above.

<script src="https://gist.github.com/etheleon/caa944b36077f83b7a448b9b03779216.js?file=create_sparksession.r"></script>

<script src="https://gist.github.com/etheleon/caa944b36077f83b7a448b9b03779216.js?file=create_sparksession.py"></script>


## IO
Spark Queries can either be executed against tables found in HMS whose partitions are known beforehand or directly against files stored in buckets. 

If you’re making queries against a HMS data warehouse using spark, you’ll need to enable HIVE support when creating your `SparkSession`

While creating a `SparkSession` represented here by the `spark` variable, you’ll have to `enableHiveSupport`, see above 

### Input

You can read either directly from files or query against a metastore which supports multiple query engines eg. PRESTO / Spark SQL such as HIVE metastore (HMS). 

Depending on whether you want it as either in-memory spark DataFrame/  temp table/view or a registered table the steps are different. 

#### In-memory

The best interface in this case is to use PySpark.

<script src="https://gist.github.com/etheleon/caa944b36077f83b7a448b9b03779216.js?file=read_partitions.py"></script>

> You’ll still be able to register this as a view using `df.createTempView("<view_name">)` and caching’s find too. Table partitioning is not supported for temporary views although you can still partition the RDDs via PySpark’s  `.repartition()`  method or SQL Hints `SELECT /*+ REPARTITION 3000 */ * FROM table` not when creating a view

### Output

If you would like the results of the ETL/query to persist so you can query it again later sometime in the future, you could save the results in parquet or even in Tensorflow’s TFRecord format.

#### Tensorflow 

The recommended file format is `tf.Data.TFRecordDataset`  when working with Tensorflow framework.

 You can save your results to this format using the following (gzipped to save space)

<script src="https://gist.github.com/etheleon/caa944b36077f83b7a448b9b03779216.js?file=save_to_tfrecord.py"></script>


> 💡 To use the `tfrecords` format, remember to include the connector JAR and place it in the `extra_classpath`. At the point of writing [org.tensorflow:spark-tensorflow-connector_2.11:1.115](https://repo1.maven.org/maven2/org/tensorflow/spark-tensorflow-connector_2.11/1.15.0/spark-tensorflow-connector_2.11-1.15.0.jar) works with the Gzip codec


### Registering tables in HMS with parquet files in S3

Similar to how BigQuery stores the underlying data of tables in `capacitor` , columnar file format stored in google’s file system `colossus`.  (GCS is built on top of colossus). It’s recommend to store the data in `parquet` also a columnar file format in S3.

> TIP: The fastest way to check if the table exists is to run `DESCRIBE schema.table`

In the following example we are going to assume that the parquet files are stored in the following path: `s3://datascience-bucket/wesley.goi/data/pricing/demand_tbl/`

#### Generate Column data type schema

##### Manual
You can prepare the table column [schema](https://cloud.google.com/bigquery/docs/schemas) like BigQuery manually and save it in a JSON file and parse it

<script src="https://gist.github.com/etheleon/caa944b36077f83b7a448b9b03779216.js?file=example_schema.json"></script>

##### Infer

<script src="https://gist.github.com/etheleon/caa944b36077f83b7a448b9b03779216.js?file=infer_col_type.py"></script>

#### Create table 
Since the table exists within HIVE, where the query in `spark.sql(query)`  follow’s HIVE’s SQL dialect.

<script src="https://gist.github.com/etheleon/caa944b36077f83b7a448b9b03779216.js?file=create_hive_table.sql"></script>

#### Insert partitions

In this example we will be adding `s3://datascience-bucket/wesley.goi/data/pricing/demand_tbl/year=2021/month=01/day=11/hour=01`

<script src="https://gist.github.com/etheleon/caa944b36077f83b7a448b9b03779216.js?file=add_partition_to_hive_table.sql"></script>

You can check if the partition has been add by running `SHOW PARTITIONS pricing.demand_tbl`

| partition |
| — |
| year=2021/month=01/day=11/hour=01|

However when you query the table you’ll notice that you cannot query the partition yet. 

You’ll still have to refresh the table for that partition

<script src="https://gist.github.com/etheleon/caa944b36077f83b7a448b9b03779216.js?file=refresh_table.sql"></script>

Remember to refresh

#### Bulk import

If you have multiple partitions and do not wish to rerun the above for each partition, you may wish to run the MSCK command to sync the all files to the HMS. 

<script src="https://gist.github.com/etheleon/caa944b36077f83b7a448b9b03779216.js?file=bulk_update.sql"></script>


## Temp Views / Tables
In the same  spark session, it is possible to create a temp view. Temp views should not be confused with views in BigQuery, these are not registered in HMS and persists only for the duration of the given `SparkSession`.  

Data is stored in memory in-memory columnar format.

These are especially useful if the data manipulation is complicated and multi stepped and you wish to persist some intermediate tables. In BQ, I would just save temp as a table. 

> NOTE: temp tables == temp views. 

From a query:

<script src="https://gist.github.com/etheleon/caa944b36077f83b7a448b9b03779216.js?file=create_temp_view.sql"></script>


## Views
Unfortunately you cannot register a view in HIVE using spark but you can do so in presto. 

## Sampling
Often when training your model, you might need to sample from the existing dataset due to memory constraints. 

You’ll have to set a seed as well when caching

# Caching 
Caching is not lazy with ANSI SQL, and it will be stored in memory immediately. 

<script src="https://gist.github.com/etheleon/caa944b36077f83b7a448b9b03779216.js?file=cache_table.sql"></script>

> Compared to PySpark `df.cache()`  (you’ll have to run `df.count()` to force the table to be loaded into memory), the above SQL statement is not lazy and will store the table in memory once executed. 

## UDFs
User-Defined-Functions (UDFs) are ways to define your own functions. Which you can write in python before declaring it for use in SQL using `spark.udf.register`

<script src="https://gist.github.com/etheleon/caa944b36077f83b7a448b9b03779216.js?file=create_udf.py"></script>

> *NOTE*: If UDF requires C binaries which needs to be compiled, you’ll  need to install in the image used by the worker nodes. 

## External table partitions
When working with spark and HMS, one has to be mindful of the term **partition**,  In spark, the term refers to data partitioning in Resilient Distributed Datasets (RDD), where partitions represent chunks of data sent to workers/executors for parallel processing. In HMS, the term represents how the data is stored in the cloud file system eg. S3 and helps guide queries agains the dataset in an efficient manner which is closer to the partitioned tables in databases.

### How to partition

First you’ll need to be able to save the data in S3, there’s a specific naming conversion for the file path which you’ll need to follow ie. `s3://<bucket>/prefix/key=value`.

As you have seen, one of the most common ways to partition a table is via timestamp  eg. `s3://<bucket>/prefix/date=YYYYMMDD`

<script src="https://gist.github.com/etheleon/caa944b36077f83b7a448b9b03779216.js?file=simple_partition.sql"></script>


One can also partition on multiple columns although in a nested manner 
eg.  `folder/year=2021/month=03/day=21`

Where the folder structure follows:

```
Year=yyyy
 |---Month=mm
 |   |---Day=dd
 |   |   |---<parquet-files>
```


> ⚠️: Check if the external table which you’re querying is already partitioned.  `SHOW PARTITIONS table`

<script src="https://gist.github.com/etheleon/caa944b36077f83b7a448b9b03779216.js?file=nested_partitions.sql"></script>


> 💡You can check the number of partitions scanned if you run `.explain(mode=“formatted”)` to see 


## SQL Hints
Hints go way back as early as spark 2.2, which introduced. These could be grouped into several categories. 

### Repartitioning

By default when repartitioning, it’ll be set to 200 partitions, you might not want this and to optimise the query you might want to *hint* spark otherwise

1.  `REPARTITION`
2.  `COALESCE` only reduces the number of partitions, optimised version of repartition. Data which is kept on the original nodes and only those which needs to be moved are moved  (see example below)
3. `REPARTITION_BY_RANGE`  eg. You have records which has a running id from 0 - 100000 and you’ll want to split them into 3 partitions `repartitionByRange(col, 3)`

When coalescing you’re shrinking the number of nodes on which the data is kept eg. From 4 to 
```
# original 
Node 1 = 1,2,3
Node 2 = 4,5,6
Node 3 = 7,8,9
Node 4 = 10,11,12

# Coalescing from 4 to 2 partitions:
Node 1 = 1,2,3 + (10,11,12)
Node 3 = 7,8,9 + (4,5,6)
```

You can also improve query time by including columns when repartitioning especially if you are joining on these columns. This applies to tables as well as temp views.

<script src="https://gist.github.com/etheleon/caa944b36077f83b7a448b9b03779216.js?file=hints.sql"></script>


> You can also chain multiple repartition hints: repartition(100), coalesce(500) and repartition by range for column `c` into 3 partitions

<script src="https://gist.github.com/etheleon/caa944b36077f83b7a448b9b03779216.js?file=chain_hints.sql"></script>


https://spark.apache.org/docs/3.0.0/sql-ref-syntax-qry-select-hints.html
the optimised plan is as follows:

```
# Repartition to 100 partitions with respect to column c.
== Optimized Logical Plan ==
Repartition 100, true
+- Relation[name#29,c#30] parquet
```

Often the number of records per partition is not equal, especially if you’re partitioning by time and you might end up the number of records per partition following a cyclic pattern. eg. Traffic at night is much lesser than traffic in the day.

[image:60D2C95E-A322-43BC-AC34-A2A4915012C0-2568-000000AF22660FB5/unknown.png]


### Join hints

* **BROADCAST JOIN** replicates the full dataset (*if it can fit into memory* of the workers) into all nodes

These are useful for selective joins (where the output is expected to small), when memory is not an issue and it’s the right table in a left join.

![broadcast](https://1.bp.blogspot.com/-s_HQfPph6z4/WcnjxGVNFkI/AAAAAAAAERM/9HfKO6H_SskkykKa_UaDRCo8URafsjixQCLcBGAs/s1600/Screen%2BShot%2B2017-09-25%2Bat%2B10.19.36%2BPM.png)

* **MERGE** : shuffle sort merge join
* **SHUFFLE_HASH**: shuffle hash join. If both sides have the shuffle hash hints, Spark chooses the smaller side (based on stats) as the build side.
* **SHUFFLE_REPLICATE_NL**: shuffle-and-replicate nested loop join


# Adaptive Query Execution (AQE)
Another new feature which comes with spark3 is the AQE. Previously the query plan is done prior to execution and no optimisation is done thereafter.

## Partitions
One of the areas which sets itself up for optimisation during execution is the to determine the optimum number of partitions.
By default,`spark.sql.shuffle.partitions` is set to 200, in cases when the dataset is small, this number would be too large while the reverse is also true.

### Broadcast Joins
If the table from any side is smaller than broadcast in in hash join threshold, sort merge joins are automatically converted to a broadcast join.

> You can try this [AQE Demo - Databricks](https://docs.databricks.com/_static/notebooks/aqe-demo.html?_ga=2.53314607.646459968.1595449158-1487382839.1592553333)

<script src="https://gist.github.com/etheleon/caa944b36077f83b7a448b9b03779216.js?file=spark_three_opts.py"></script>


`CustomShuffleReader` indicates it’s using AQE and it ends with AdaptiveSparkPlan
