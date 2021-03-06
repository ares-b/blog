---
layout: post
title:  "Spark Core Concepts"
author: ares
categories: [ ]
image: assets/images/posts/spark-core-concepts/featured.png
toc: true
show-post-image: false
featured: false
hidden: true
---
**Apache Spark** is a fast data processing engine dedicated to Big data, it allows carrying out processing on large
volumes of data in a parallel and distributed manner using Map/Reduce programming paradigm.

Many of us already know about Spark, but we often get confused with Spark's architecture and some
key concepts associated with it.

# Spark Basic Architecture

Spark is based on a Master-Slave Architecture, and it consists of three components : Driver, Cluster Manager and Executors.

<p align="center">
    <img alt="Spark Basic Architecture" src="../assets/images/posts/spark-core-concepts/spark-basic-architecture.png" />
</p>
<p align="center">
    <em>Spark Basic Architecture</em>
</p>

Driver has multiple responsibilities :
- Analyzes, distributes and schedules the different workloads across the executors
- Monitors Spark Application

While executors are responsible for processing the data assigned them.

In the future, I'll be writing an article about in-depth Spark Architecture to explain the role of the cluster manager on keeping harmony between all the other components.

# Partitioning

before Spark, we used to process Big Data with HDFS MapReduce which is splitting data into blocks (or partitions), each block is stored and processed on a slave node (data node) and replicated on other data nodes to assure availability.

<p align="center">
    <img alt="Spark Partitions" src="../assets/images/posts/spark-core-concepts/spark-partitions.png" />
</p>
<p align="center">
    <em>Spark Partitions</em>
</p>

Spark partitions works the same, they're logical chunks of data split across executor's RAM.

So basically, each partition will be processed by one executor, and each executor will process one or more partitions.

# Shuffling

Data shuffling is when operation require data exchange between Spark Workers.
For example, a join operation between two datasets would require a Shuffle if these two datasets partitions are not on the
same worker node.

One of the performance issues in parallelized processing relies on shuffles because they require network transfer.

# Transformations and Actions

We've seen that RDDs are lazy evaluated which makes the execution of processing fast and thus divides the operations to be executed into two groups: transformations and actions.

A transformation is a lazy evaluated function that takes an RDD, Dataframe or Dataset and returns another RDD, Dataframe or Dataset.

Transformations can have either Wide or Narrow dependencies.

An action is a Spark operation that triggers the evaluation of the transformations and thus of the partitions.
For example, returning data to the driver (with operations like count or collect) or writing data to an external storage system.

## Narrow Dependency Transformations

A narrow transformation is one that doesn't require data shuffling, it can be applied to a single partition.

For instance, `map` and `withColumn` operations are narrow.

<p align="center">
    <img alt="Spark Narrow Transformation" src="../assets/images/posts/spark-core-concepts/narrow-transformation.png" />
</p>
<p align="center">
    <em>Spark Narrow Transformation</em>
</p>

## Wide Dependency Transformations

Wide dependencies in the other hand often requires data shuffling, moves the data in a particular way between the workers.

Examples of wide dependencies transformations are `groupByKey` and `join`, these transformations moves the data in a particular way between the workers,
for example, according to the value of the keys. The data is partitioned so that the ones who shares the same key is in the same partition.

<p align="center">
    <img alt="Spark Wide Transformation" src="../assets/images/posts/spark-core-concepts/wide-transformation.png" />
</p>
<p align="center">
    <em>Spark Wide Transformation</em>
</p>

# Spark Memory Abstraction

Hadoop MapReduce used to store intermediate iteration results in disk (as show in the picture below), this process was really slow as it involves read/writes from disk each iteration.
Plus, it is not suitable for some applications, especially those that reuse data through multiple operations such as most statistical learning algorithms, most of which
are iterative, and requires shuffles operations across the cluster (joins for example). Shuffle operations on HDFS MapReduce were so resource consuming that it limited the potential of this distributed architecture.

<p align="center">
    <img alt="HDFS Data Processing" src="../assets/images/posts/spark-core-concepts/hdfs-read-write.png" />
</p>
<p align="center">
    <em>HDFS Data Processing</em>
</p>

The success of the Spark framework against the MapReduce implementation on Hadoop is due to its in-memory processing which will lead to cheaper Shuffle
steps. Indeed, MapReduce does several disk reads/writes while Spark limits many of them and stores the intermediate step data in memory.

<p align="center">
    <img alt="Spark Data Processing" src="../assets/images/posts/spark-core-concepts/spark-mapreduce.png" />
</p>
<p align="center">
    <em>Spark Data Processing</em>
</p>

Spark does that using a memory abstraction called RDD, or Resilient Distributed Dataset.

## RDDs

Spark does that using a memory abstraction called RDD, or Resilient Distributed Dataset. It's a read-only data collection
partitioned and distributed across cluster nodes.
RDD was created to solve the problem that iterative algorithms and interactive computations pose to MapReduce.

It acts as a data sharing abstraction in a cluster. When we talk about "distributed" in the definition of RDD,
we are actually referring to "shared", atomic, because it uses cache memory to persist data in RAM for reuse,
thus avoiding data replication to disk, which is necessary in Hadoop to ensure cluster availability. Thanks to
this mechanism, Spark is able to provide high availability and fault tolerance.

Since an RDD is an abstraction, it has no real existence, so it must be explicitly created or instantiated
through deterministic operations on existing data files or on other RDD instances. These operations are
called [transformations][transformations].

### Key features

Spark being designed in Scala, it was obvious that RDDs, as collections, inherits the characteristics of the Scala language collections (array, list, tuple, etc) :
- Lazy computations: computations performed on RDDs are "lazy". This means that spark executes expressions only when they are needed. Technically, it is when an action operation is triggered on the RDD that it is executed. This greatly improves the performance of the application
- Immutable: RDDs are immutable. This means that they are only accessible in read mode. It is therefore not possible to modify an RDD. This feature is very useful when managing concurrent access to data, especially in a context of large-scale data valuation
- In-memory: RDDs are executed in memory. They can also be persisted in cache for more speed. Spark developers have provided the ability to choose where to persist RDDs (either on disk, in cache, or in memory and on disk) using the Storage.LEVEL property
- Functional Programming: Operations done on RDD are functional. Plus, RDDs are Monads (I'll be writing an article about monads soon)
- Type Safety: RDDs are type-safe

In addition to that, RDDs have some other features that you have probably guessed from my explanations :
- Distributed: RDDs uses MapReduce paradigm to process large collections of data in parallel and in a distributed manner
- Fault Tolerant: Since RDDs are distributed into **partitions**, it's quite easy to restore lost data in case of node failure. Indeed, instead of replicating the data as HDFS does, RDDs rely on **[Data Lineage]** to restore lost data.

### Limitations

The biggest limitation of RDDs is that they don't support any Optimization engine, developers needs to optimize each operation done on the RDDs.

Plus, RDDs doesn't handle Schemas, if you want your data to be structured, you have to handle that yourself.

## Dataframes

Dataframes were introduced at Spark 1.3 (I think), they're based on RDDs and their goal is to overcome the RDDs limitations.

Dataframes are just like RDDs, a distributed collection of data, but, DFs are organized into columns (they must have a schema).

### Key features

Dataframes have key features of RDDs beside TypeSafety. But, they offer more interesting features :
- Column Organization: as I said, Dataframes are distributed collection of data organized into named columns. It is conceptually equivalent to a table in a relational database
- Handles Heterogeneous data: Dataframes comes with an API that allows us to process structured and unstructured data
- Optimization: Dataframes have metadata which makes them compatible with optimization frameworks like Tungsten and Catalyst

### Limitations

Dataframes sadly doesn't support compile time type safety, so if you're not sure about the structure of the Data you're manipulating, you'll probably get runtime errors with Dataframes.

Also, they cannot really operate with domain objects, what I mean by that, is when you've class objects that serialize some Data, and then you create a Dataframe from that class objects, you cannot retrieve back your objects.

Below a Scala example illustrating this use case :
```
case class Car(brand : String , name : String, maxSpeed : Int)
val carsRdd = sc.makeRDD(Seq(Car("Toyota", "Supra", 300), Car("Nissan", "GTR-R32", 300)))
val carsDf = sqlContext.createDataFrame(carsRdd)
carsDf.rdd // returns RDD[Row] , does not returns RDD[Car]
```

## Datasets

Dataset is an extension of the Dataframe API and it provides a type-safe, object-oriented programming interface.
It is a strongly-typed, immutable collection of objects that are mapped to a relational schema.

### Key features

Dataset provides best of both worlds from RDDs and Datasets:
- RDDs: functional programming, type-safety, etc
- Dataframes : Schema (relational model), query optimization, tungsten, etc

Plus, they handle domain objects, thanks to **Encoders** (I'll dedicate an article to encoders). Using Encoders, Datasets can convert JVM objects into Datasets and vice-versa.

### Limitations

The downside of Datasets is that they require specifying the classes fields as Strings. Once you query the data, you can convert it to the proper data type.

``` 
dsCars.select(col("brand").as[String], $"maxSpeed".as[Int]).show
```

# Spark In Depth Architecture
Spark Architecture is a **Master-Slave** architecture, and it consists of three components: Master node, Cluster Manager and Worker node(s).

![Spark Architecture](../assets/images/posts/spark-core-concepts/spark-architecture.png "Spark Architecture")

## Driver
The Driver process runs on a [JVM][spark-memory-management], it orchestrates and monitors the execution of a Spark application.

Spark Driver can run into two deployment modes :
1. Client mode : the driver process lives in the host machine submitting the application, so the driver will be running outside the cluster. For example, when you run spark-shell, you're in client mode and the driver process is attached to your terminal. When you exit the shell, the driver process terminates
2. Cluster mode : driver lives inside the cluster (in a worker machine)

The Driver process is always attached to one single Spark application.

Driver and its components are responsible for:
- Running the main()` method of your Java, Scala or Python Spark Application, it creates a [Spark Context or a Spark Session][spark-session-and-spark-context] which can be used to load data, transform it and write it somewhere
- Breaking the application logic into Stages and Tasks [Tasks][spark-tasks], it checks the user code and determines how many stages and how many tasks will be needed to process the data
-

# Core Concepts
## Spark Session and Spark Context

## Spark Tasks

Spark can run on **Local** mode,  **Standalone** mode**Cluster** mode or on .

Local mode means that Spark launches all the tasks on a single [JVM][Spark-Memory-Management] and the parallelism will be done according
to the number of threads. There's no task distribution on Local mode.



[driver]: <#driver> "Spark Driver"
[Same plan type Transformations]: <#same-plan-type-transformation> "Same plan type Transformation"
[Spark In Depth Architecture]: <#spark-in-depth-architecture> "Spark Driver"
[transformations]: <#transformations> "Spark Transformations"
[spark-memory-management]: </spark-memory-management> "Spark Memory Management"
[spark-session-and-spark-context]: <#spark-session-and-spark-context> "Spark Memory Management"
[spark-tasks]: <#spark-tasks> "Spark Memory Management"
