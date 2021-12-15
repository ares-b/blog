---
layout: post
title:  "Spark Core Concepts"
author: ares
categories: [ spark ]
image: assets/images/posts/spark-core-concepts/featured.png
toc: true
toc-min: 2
toc-max: 3
show-post-image: false
featured: true
---
**Apache Spark** is a fast data processing engine dedicated to Big data, it allows to carry out processing on large 
volumes of data in a parallel and distributed manner using Map/Reduce programming paradigm.

Many of us already know about Spark but we often get confused with Spark's architecture and some of the 
key concepts associated with it.

In this article, I'll explain Spark's architecture and its key concepts.

# Partitioning

Before Spark, we used HDFS MapReduce to process Big Data, in a nutshell, data is split into Blocks (or partitions), each block goes into a worker node and is replicated into other nudes for fault tolerance, 
when processing this data, each worker processes the data he stores. 

Spark partitions works the same, they're logical chunks of data split across [executor][executors]'s RAM.

![Spark Paritions](../assets/images/posts/spark-core-concepts/spark-partitions.png "Spark Partitions")

# Spark Memory Abstraction

Hadoop MapReduce used to store intermediate iteration results in disk (as show in the picture below), this process was really slow as it involves read/writes from disk each iteration.
Plus, they are not suitable for some applications, especially those that reuse data through multiple operations such as most statistical learning algorithms, most of which
are iterative, and requires shuffles operations across the cluster (joins for example). Shuffle operations on HDFS MapReduce were so resource consuming that it limited the potential of this distributed architecture.

![HDFS Processing](../assets/images/posts/spark-core-concepts/hdfs-read-write.png "HDFS Processing")

The success of the Spark framework against the MapReduce implementation on Hadoop is due to its in-memory processing which will lead to cheaper Shuffle
steps. Indeed, MapReduce does several disk reads/writes while Spark limits many of them and stores the intermediate step data in memory.

![Spark Processing](../assets/images/posts/spark-core-concepts/spark-mapreduce.png "Spark Processing")

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

Dataframes were introduced at Spark 1.3 (i think), they're based on RDDs and their goal is to overcome the RDDs limitations.

Dataframes are just like RDDs, a distributed collection of data, but, DFs are organized into columns (they must have a schema).

### Key features

Dataframes have key features of RDDs beside TypeSafety. But, they offer more interesting features :
- Column Organization: as I said, Dataframes are distributed collection of data organized into named columns. It is conceptually equivalent to a table in a relational database
- Handles Heterogeneous data: Dataframes comes with an API that allows us to process structured and unstructured data
- Optimization: Dataframes have metadata which allows makes them compatible with optimization frameworks like Tungsten and Catalyst

### Limitations

Dataframes sadly doesn't support compile time type safety, so if you're not sure about the structure of the Data you're manipulating, you'll probably get runtime erros with Dataframes.

Also, they cannot really operate with domain objects, what I mean by that, is when you've class objects that serialize some Data, and then you create a Dataframe from that class objects, you cannot retrieve back your objects.

Below a Scala example illustrating this use case :
```
case class Car(brand : String , name : String, maxSpeed : Int)
val carsRdd = sc.makeRDD(Seq(Car("Toyota", "Supra", 300), Person("Nissan", "GTR-R32", 300)))
val carsDf = sqlContext.createDataframe(carsRdd)
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

# Transformations and Actions

Spark APIs supports two types of operations: transformations and actions.

## Actions

## Transformations
A transformation consists of creating a new RDD from another. Transformations are lazy evaluated,  

Few examples of Spark Transformations,
RDD's `map` function is 
one meanwhile actions collects data from [executors][executors] to the [driver][driver].

# Spark Architecture
Spark Architecture is a **Master-Slave** architecture, and it consists of three components: Master node, Cluster Manager and Worker node(s).

![Spark Architecture](../assets/images/posts/spark-core-concepts/spark-architecture.png "Spark Architecture")

## Driver
The Driver process runs on a [JVM][spark-memory-management], it orchestrates and monitors the execution of a Spark application.

Spark Driver can run into two deployment modes : 
1. Client mode : the driver process lives in the host machine submitting the application, so the driver will be running outside the cluster. For example, when you run spark-shell, you're in client mode and the driver process is attached to your terminal. When you quit the shell, the driver process terminates
2. Cluster mode : driver lives inside the cluster (in a worker machine)

The Driver process is always attached to one single Spark application.

Driver and its components are responsible for:
- Running the main()` method of your Java, Scala or Python Spark Application, it creates a [Spark Context or a Spark Session][spark-session-and-spark-context] which can be used to load data, transform it and write it somewhere
- Breaking the application logic into Stages and Tasks [Tasks][spark-tasks], it check the user code and determines how many stages and how many task will be needed to process the data
- 

# Core Concepts
## Spark Session and Spark Context

## Spark Tasks

Spark can run on **Local** mode,  **Standalone** mode**Cluster** mode or on .

Local mode means that Spark launches all the tasks on a single [JVM][Spark-Memory-Management] and the parallelism will be done according
to the number of threads. There's no task distribution on Local mode.



[driver]: <#driver> "Spark Driver"
[transformations]: <#transformations> "Spark Transformations"
[spark-memory-management]: </spark-memory-management> "Spark Memory Management"
[spark-session-and-spark-context]: <#spark-session-and-spark-context> "Spark Memory Management"
[spark-tasks]: <#spark-tasks> "Spark Memory Management"
