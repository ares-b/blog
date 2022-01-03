---
layout: post
title:  "Spark Query Execution Engine"
author: ares
categories: [ spark ]
image: assets/images/posts/spark-execution-engine/featured.png
toc: true
show-post-image: false
featured: hidden
disclaimer: "/!\ This article is currently being written. Some parts may be incomplete or badly written."
---

I think that we're all familiar with the Diagram below, if it's not your case, well, basically, it represents what Spark
does internally when processing data using Spark SQL.

<p align="center">
    <img alt="Spark Execution Planning" src="../assets/images/posts/spark-execution-engine/spark-execution-planning.png" />
</p>
<p align="center">
    <em>Spark Execution Planning</em>
</p>

First, one thing to know, is that Execution planning isn't lazy evaluated. The code snippet below shows that.

```
case class Car(brand : String , name : String, maxSpeed : Int)
val carsRdd = sc.makeRDD(Seq(Car("Toyota", "Supra", 300), Car("Nissan", "GTR-R32", 300)))
val df = spark.createDataFrame(carsRdd)
df.select(col("brand"))
df.select(col("idk"))
```

Line 4 will work, but, line 5 will throw an Analysis Exception saying that column `idk` can't be resolved.
This means that the Execution plan is evaluated immediately when a transformation is applied.

# Catalyst

Catalyst is a Framework for representing and manipulating a **Dataflow graph** as Trees of Relational Operators and Expressions.

Spark >= 2.0 uses Catalyst Framework to build an extensible execution plan optimizer for Spark SQL based APIs. It supports both rule based and cost bases
optimizations.

Basically, when you process data using Spark SQL, Dataframe or Datasets API, Catalyst will parse your queries into trees, then, 
Catalyst will manipulate those trees to validate, optimize your queries and compile them to JVM's bytecode. 

It may be a bit abstract for now, but don't worry, this article's goal is to explain to you everything going under Spark's hood.

## Catalog

Catalog or `SessionCatalog` is the registry of relational entities (databases, tables, views, functions and partitions) in a `SparkSession`.
It's a layer over the `ExternalCatalog` which allows for different metastore to be used (you can even write your own Catalog).

Catalog enables :
- Metadata centralization
- Providing a single source of truth for the stored data
- Data Governance 

Catalog can be accessed through the `SessionState` of a `SparkSessions` which is the state separation layer between SparkSessions. 
It contains SQL parsers, UDFs and everything else that depends on a `SQLConf`.

Catalog in used by the Catalyst to validate and resolve [Logical QueryPlans](#logical-plan) during the Execution Planning.

## Trees

Trees are the main data type in Spark Execution Planning. They're parsed from a given user code of a Spark Application and 
represent a tree of relational operators of a structures query. 

Catalyst Trees are analyzed and transformed in order to validate and optimize the query execution.

### TreeNodes

TreeNodes are recursive data structures that can have zero, one or many children which themselves are TreeNodes.

It is the base class for [Expressions](#expression) and [Query Plans](#query-plan).

## Expression

Expressions are executable TreeNodes in Catalyst Tree that can evaluate a result given a certain input values. In other terms, 
Expressions can produce a JVM object per `Internal Row`.

Catalyst's Expression abstract class has several trait implementations :
- `Nondeterministic` : a deterministic expression is like a [pure function](https://en.wikipedia.org/wiki/Pure_function) in Functional Programming
- `Unevaluable` : an expression that is not supposed to be evaluated (`eval`, `doGenCode` and `genCode` Expression methods are not Implemented)
- `LeafExpression` : an expression that has no child (i.e : no child TreeNode)
- `UnaryExpression` : an expression with one and only one child
- `BinaryExpression` : an expression with exactly two children
- `NamedExpression` : an Expression that is named 
- `Attribute` : an Expression that represents an Attribute. It's always LeafExpression, NamedExpression and NullIntolerant at the same time

Catalyst Expressions are used almost everywhere. For Instance, Spark's native functions that you can find in [Spark API Reference](https://spark.apache.org/docs/latest/api/scala/org/apache/spark/sql/functions$.html) 
are built using Catalyst Expressions. For example, when you want to add a column holding the literal value `random_string_value` to your Dataset,
you'd probably want to use `lit` function, `lit` is build on-top of `Literal` Catalyst Expression which is a `LeafExpression`.

Expression are also used in [Catalyst Transformations](#trees-transformations), in [Logical Operators](#logical-plan) and [Physical Operators](#physical-plan), etc.

## QueryPlan

Query Plans is an abstract class for Spark's [Logical Planning](#logical-plan) and [Physical Planning](#physical-plan).
It's a tree of operators that have a tree of expressions.

### Logical Plan

Logical Plan is an extension of QueryPlan for Logical Operators used to build a Logical Query Plan. Which is a tree of `TreeNode`s of 
Logical Operators that in turn can have trees of Catalyst `Expression`s.

There are three main Logical Operators, and they can have zero, one or two children :
- `LeafNode` : Logical Operator with no child operators
- `UnaryNode` : Logical Operator with exactly one child operator
- `BinaryNode` : Logical Operator with exactly twi children operators

Examples of Logical Operators are : Join (BinaryNode), Projection (UnaryNode), etc.

TODO : Add Logical Operator Code

### Physical Plan

TODO : Explain Physical Plan / Operators

## Trees transformations

Trees transformations are defined as Partial Functions.

There two types of Tree Transformations : Same plan type and Different plan type Transformation.

### Same plan type Transformation

Transforms the tree without changing its type:
- Expression to expression
- Logical Plan to Logical Plan
- Physical Plan to Physical Plan

<p align="center">
    <img alt="Expressions Tree Transformation" src="../assets/images/posts/spark-execution-engine/catalyst-transform-same-type.png" />
</p>
<p align="center">
    <em>Expressions Tree Transformation</em>
</p>

Transformation applied in order to go from the Left tree to the right one is defined as **Constant folding** rule :
```
transform{
    case Add(Literal(x: Integer), Literal(y: Integer)) =>
        Literal(x+y)
}
```

### Different plan type Transformation

Transforms the tree to another kind of Tree, especially Logical Plan Tree to Physical Plan Tree.
This can be done using **Strategies**, it's a pattern matching that converts a Logical plan to the corresponding Physical plan.

What it actually does, it converts the nodes of the Logical tree to nodes of the Physical tree.

```
object BasicOperators extends Strategy{
    def apply(plan: LogicalPlan): Seq[SparkPlan] = plan match {
        ...
        case logical.Project(projectList, child) => 
            execution.ProjectExec(projectList, planLater(child)) :: Nil
        case logical.Filter(condition, child) =>
            execution.FilterExec(condition, planLater(child)) :: Nil
        ...
    }
}
```

The code above is a snippet of the Spark's BasicOperators Strategy which transforms basic logical tree nodes (such as Filter) to
actual physical nodes.

Often times, a single Strategy isn't enough to convert all kinds of logical plans into physical ones, so we call `planLater` which will
trigger different kind of strategies.

For the next part of the article, let's take an example to better understand what happens in the Spark Execution Planning.

```
val df1 = spark.range(10000000).toDF("id1").withColumn("name", lit("arslane"))
val df2 = spark.range(20000000).toDF("id2").withColumn("value", rand() * 10000000)
val df3 = df1.join(df2, df1("id1") === df2("id2")).filter("value > 50 * 1000").select(col("id1"), col("value").plus(1).plus(2).alias("v")).groupBy("id1").sum("v")
```

### Rule Executor 

Examples showed above are really simple and thus, can be optimized using only few transformation rules. In real world scenarios,
we'd like the Spark Engine to apply multiple rules.

Actually this is what it does, is uses a **Rule Executor** to transform a tree to another tree of the same type by applying many rules in batches.

<p align="center">
    <img alt="Rule Executor" src="../assets/images/posts/spark-execution-engine/rule-executor.png" />
</p>
<p align="center">
    <em>Rule Executor</em>
</p>

**Rule Executor** has two approaches of applying rules :
- Fixed Point : apply each rule in a batch sequentially, over and over again, until the tree does not change anymore
- Apply Once : each rule in a batch is applied only once

# Spark Query Execution in Action

## Logical Planning

Logical planning (c.f figure above) is the first step for the creation of the logical plan.
it describes computation on data without defining how these computations are done physically
(it does not specify join algorithms for example).

<p align="center">
    <img alt="Spark Logical Planning" src="../assets/images/posts/spark-execution-engine/logical-planning.png" />
</p>
<p align="center">
    <em>Spark Logical Planning</em>
</p>

### Unresolved Logical Plan

First Catalyst parses the code written using the Dataframe, Datasets API or in SQL into a tree which will lead to the creation of the **Unresolved Logical Plan**.

`df3` tree will be parsed into the Tree below.

<p align="center">
    <img alt="Unresolved Logical Plan" src="../assets/images/posts/spark-execution-engine/unresolved-logical-plan.png" />
</p>
<p align="center">
    <em>Unresolved Logical Plan</em>
</p>

If we do a `df3.explain(true)` spark will return to us the unresolved/parsed logical plan.

```
== Parsed Logical Plan ==
Aggregate [id1#2L], [id1#2L, sum(v#183) AS sum(v)#189]
+- Project [id1#2L, ((value#11 + cast(1 as double)) + cast(2 as double)) AS v#183]
   +- Filter (value#11 > cast((50 * 1000) as double))
      +- Join Inner, (id1#2L = id2#9L)
         :- Project [id1#2L, arslane AS name#4]
         :  +- Project [id#0L AS id1#2L]
         :     +- Range (0, 10000000, step=1, splits=Some(8))
         +- Project [id2#9L, (rand(-9022958612971169036) * cast(10000000 as double)) AS value#11]
            +- Project [id#7L AS id2#9L]
               +- Range (0, 20000000, step=1, splits=Some(8))
```

At this point, Spark doesn't do any optimization, it just parses the code we wrote into a tree.
Also, no check is done to find out if the columns we specified exists or not, nothing is checked here, that's why it's called Unresolved.

### Resolved Logical PLan

Once we've got the Unresolved logical Plan, **Spark Analyzer** will use the Catalog to figure out where these datasets and columns come from and types of their columns.
Then, the Analyzer will verify and resolve that everything is okay (column names, data types, applicability of transformations, etc) by checking the metadata inside the Catalog.

Analyzer is a **Rule Executor**

For example, if we have these 3 lines below, Analyzer will resolve the first one, while the 2nd and 3rd will be rejected, and it will throw a `AnalysisException`.

```
df.select("id") // OK Column exists in Catalog
df.select("age") // NOT OK Column doesn't exists in Catalog
df.select(max("name")) // NOT OK cannot apply max() to a None numeric column
```

Again, if we run a `df3.explain(true)`, here's the output for the Resolved Logical Plan.

```
== Analyzed Logical Plan ==
id1: bigint, sum(v): double
Aggregate [id1#2L], [id1#2L, sum(v#183) AS sum(v)#189]
+- Project [id1#2L, ((value#11 + cast(1 as double)) + cast(2 as double)) AS v#183]
   +- Filter (value#11 > cast((50 * 1000) as double))
      +- Join Inner, (id1#2L = id2#9L)
         :- Project [id1#2L, arslane AS name#4]
         :  +- Project [id#0L AS id1#2L]
         :     +- Range (0, 10000000, step=1, splits=Some(8))
         +- Project [id2#9L, (rand(-9022958612971169036) * cast(10000000 as double)) AS value#11]
            +- Project [id#7L AS id2#9L]
               +- Range (0, 20000000, step=1, splits=Some(8))
```

As you can see, the Logical Plan slightly changes, Spark now knows what are the types of columns `id1` and `v`.
If there was a problem with the Catalog and the operations we're doing on the dataframes, Analyzer would have thrown an `AnalysisException`.

### Optimized Logical Plan

Once the Logical Plan is resolved, Catalyst Optimizer will take the initiative to actually Optimize the Logical Plan
by applying same plan type transformations.

In a nutshell, Catalyst with the help of [Rule Executor](#rule-executor) applies **Rule-based optimization**, these rules includes : Constant folding,
Predicate push-down, Projection pruning and other rules (even user coded rules).

#### Constant Folding

Which we've already seen in [Same plan type Transformations](#same-plan-type-transformation).

#### Predicate push-down

This kind of rules optimizes where the `filter` operations happens.

By applying this rule to the tree in the `Unresolved Logical Plan` image will be transformed into the tree below :

<p align="center">
    <img alt="Predicate Push-down Transformation" src="../assets/images/posts/spark-execution-engine/catalyst-predicate-pushdown.png" />
</p>
<p align="center">
    <em>Predicate Push-down Transformation</em>
</p>

#### Projection Pruning

This kind of rules optimizes where the `select` operations happens.

By applying this rule to the tree in the `Unresolved Logical Plan` image will be transformed into the tree below :

<p align="center">
    <img alt="Projection Pruning Transformation" src="../assets/images/posts/spark-execution-engine/catalyst-predicate-pushdown.png" />
</p>
<p align="center">
    <em>Predicate Push-down Transformation</em>
</p>    

After applying these transformations to the tree, Catalyst will return the tree below :

<p align="center">
    <img alt="Catalyst Optimized Logical Plan" src="../assets/images/posts/spark-execution-engine/catalyst-optimized.png" />
</p>
<p align="center">
    <em>Catalyst Optimized Logical Plan</em>
</p>

If we do a `df3.explain(true)` spark will return to us the Optimized logical plan.

```
== Optimized Logical Plan ==
Aggregate [id1#2L], [id1#2L, sum(v#183) AS sum(v)#189]
+- Project [id1#2L, ((value#11 + 1.0) + 2.0) AS v#183]
   +- Join Inner, (id1#2L = id2#9L)
      :- Project [id#0L AS id1#2L]
      :  +- Range (0, 10000000, step=1, splits=Some(8))
      +- Filter (value#11 > 50000.0)
         +- Project [id#7L AS id2#9L, (rand(-9022958612971169036) * 1.0E7) AS value#11]
            +- Range (0, 20000000, step=1, splits=Some(8))
```

As you can see in the Optimized Logical Plan and on the figure **Catalyst Optimized Logical Plan**, the filter condition was
moved above the `Scan` node thanks to `Predicate push-down`. Also, the `Projection` is pushed before the `Join` operation thanks to `Projection Pruning`.

## Physical Planning

TODO : Explain how Physical Planning is done

# References

- **Yin Huai** "A Deep Dive into Spark SQL's Catalyst Optimizer" [talk](https://www.youtube.com/watch?v=RmUn5vHlevc)
- **Jacek Laskowski** "The Internals of Spark" [gitbook](https://jaceklaskowski.gitbooks.io/mastering-spark-sql/content/)
