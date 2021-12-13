---
layout: post
title:  "Spark Core Concepts"
author: ares
categories: [ spark ]
image: assets/images/posts/spark-core-concepts/featured.png
toc: true
show-post-image: false
featured: true
---
**Apache Spark** is a fast data processing engine dedicated to Big data, it allows to carry out processing on large 
volumes of data in a parallel and distributed manner using Map/Reduce programming paradigm.

Many of us already know about Spark but we often get confused with Spark's architecture and some of the 
key concepts associated with it.

In this article, I'll explain Spark's architecture and its key concepts.

# RDDs

Before Spark, HDFS MapReduce used to store intermediate iteration results in disk (as show in the picture below), this process was really slow as it involves read/writes from disk each iteration.
Plus, they are not suitable for some applications, especially those that reuse data through multiple operations such as most statistical learning algorithms, most of which 
are iterative, and requires shuffles operations across the cluster (joins for example). Shuffle operations on HDFS MapReduce were so resource consuming that it limited the potential of this distributed architecture.

![HDFS Processing](../assets/images/posts/spark-core-concepts/hdfs-read-write.png "HDFS Processing")

The success of the Spark framework against the MapReduce implementation on Hadoop is due to its in-memory processing which will lead to cheaper Shuffle 
steps. Indeed, MapReduce does several disk reads/writes while Spark limits many of them and stores the intermediate step data in memory.

![Spark Processing](../assets/images/posts/spark-core-concepts/spark-mapreduce.png "Spark Processing")

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

Spark being designed in Scala, it was obvious that RDDs, as collections, inherits the characteristics of the Scala language collections (array, list, tuple, etc) :
1. Lazy computations: computations performed on RDDs are "lazy". This means that spark executes expressions only when they are needed. Technically, it is when an [action][actions] is triggered on the RDD that it is executed. This greatly improves the performance of the application
2. Immutable: RDDs are immutable. This means that they are only accessible in read mode. It is therefore not possible to modify an RDD. This feature is very useful when managing concurrent access to data, especially in a context of large-scale data valuation
3. In-memory: RDDs are executed in memory. They can also be persisted in cache for more speed. Spark developers have provided the ability to choose where to persist RDDs (either on disk, in cache, or in memory and on disk) using the Storage.LEVEL property

# Spark Architecture
Spark Architecture is a **Master-Slave** architecture and it consists of three components: Master node, Cluster Manager and Worker node(s).

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




[spark-memory-management]: </spark-memory-management> "Spark Memory Management"
[spark-session-and-spark-context]: <#spark-session-and-spark-context> "Spark Memory Management"
[spark-tasks]: <#spark-tasks> "Spark Memory Management"
