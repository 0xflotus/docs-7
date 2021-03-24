---
title: 'Scalability, Availability, and Performance'
---

# Scalability, Availability, and Performance

For users that find Dolt's features attractive, this section details important information about Dolt to consider when using it as part of a production stack.

If you have questions feel free to [reach out](https://www.dolthub.com/contact) to us.

## Stability

Dolt is pre-1.0. We expect to release 1.0 version by mid 2021. The 1.0 release signals a commitment to no backwards incompatible changes to the format. That means that versions of Dolt subsequent to the 1.0 release will always be able to read Dolt data written as of 1.0 or later. We are obviously less stable than database solutions like MySQL and Postgres currently. Those solutions have been in market for over twenty five years. We think we can get to that degree of stability by the end of 2021.

In practice Dolt's SQL implementation is relatively full featured, and we constantly improving on it. All breaking changes, even prior to 1.0, come with robust migration mechanisms, so no data will ever be lost in practice.

## Scalability

Dolt is currently constrained to the hard drive size of single machine. Like any relational database once the storage footprint goes past a single machine the only solution is sharding. We have not implemented sharding in Dolt, though it is on our long term roadmap.

Like a relational database, as data grows non-indexed read performance degrades. Unlike relational database Dolt is effectively append-only, though it still looks like a relational database. The larger the history grows, the larger the storage footprint is. There are offline utilities for pruning history when needed.

## High Availability

Dolt's availability story is similar to a single Postgres or MySQL instance, and read-only replicas are on our roadmap. Dolt's multi-instance high availability provides weaker consistency in exchange for total process isolation.

### Single Instance

Dolt's single instance availability model is similar to a single instance of MySQL or Postgres. We are constrained by the performance of our query engine. We do not currently support read-only replicas, but that feature is on our roadmap.

### Multi Instance

Dolt multi-instance is completely decentralized, and new application instances each have their own server instance. New application instances obtain a server by cloning a remote and launching a Dolt SQL Server. Dolt's `pull` and `push` semantics provide a way to periodically reconcile changes. The architecture looks something like this:

![Multi-instance Dolt](../.gitbook/assets/dolt-high-availability-multi-instance.png)

It is on our roadmap to improve the consistency primitives available in such an architecture, as currently users need to make decisions about how to implement consistency.

## Performance

Our approach to performance benchmarking is to use `sysbench`, an industry standard benchmkarking tool.

### Performance Roadmap

Dolt is slower than MySQL. The goal is to get Dolt to within 2-4 times the speed of MySQL for common operations. If a query takes MySQL 1 second, we expect it to take Dolt 2-4 seconds. Or, if MySQL can run 8 queries in 10 seconds, then we want Dolt to run 2-4 queries in 10 seconds. The `multiple` column represents this relationship with regard to a particular benchmark.

It's important recognize that these are industry standard tests, and are OLTP oriented. Many Dolt use-cases are non-OLTP, and Dolt is fast for bulk operations common in, for example, data pipeline contexts.

### Benchmark Data

Below are the results of running `sysbench` MySQL tests against Dolt SQL Server for the most recent release of Dolt. We will update this with every release. The tests attempt to run as many queries as possible in a fixed 2 minute time window. The `Dolt` and `MySQL` columns show the median latency of each test during that 2 minute time window.

The Dolt version is `0.24.3`.

| Read Tests | Dolt | MySQL | Multiple |
| :--- | :--- | :--- | :--- |
| covering\_index\_scan | 10.65 | 1.44 | 7.0 |
| index\_scan | 121.08 | 35.95 | 3.0 |
| oltp\_point\_select | 1.3 | 0.11 | 12.0 |
| oltp\_read\_only | 29.72 | 2.3 | 13.0 |
| select\_random\_points | 2.66 | 0.26 | 10.0 |
| select\_random\_ranges | 2.76 | 0.29 | 10.0 |
| table\_scan | 134.9 | 36.24 | 4.0 |
| mean |  |  | _8.43_ |

| Write Tests | Dolt | MySQL | Multiple |
| :--- | :--- | :--- | :--- |
| bulk\_insert | 0.001 | 0.001 | 1.0 |
| oltp\_delete | 11.45 | 0.11 | 104.0 |
| oltp\_insert | 12.75 | 3.62 | 4.0 |
| oltp\_read\_write | 78.6 | 7.17 | 11.0 |
| oltp\_update\_index | 14.73 | 3.82 | 4.0 |
| oltp\_update\_non\_index | 8.13 | 3.82 | 2.0 |
| oltp\_write\_only | 52.89 | 4.91 | 11.0 |
| mean |  |  | _19.57_ |

| Overall Mean Multiple | _14.0_ |
| :--- | :--- |
<br/>
### SQL Correctness

To measure Dolt's SQL correctness, we test each release of Dolt against a SQL testing suite called the [sqllogictests](https://github.com/dolthub/sqllogictest). More passing tests with a result of "ok" indicate the degree to which Dolt's SQL engine is working correctly. Here are Dolt's correctness results for version `0.24.3`.

| Results | Count |
| :--- | :--- |
| ok | 5466798 |
| not ok | 463624 |
| did not run | 2464 |
| timeout | 2 |
| _total_ _tests_ | 5932888 |
<br/>

| Correctness Percentage | 92.0 |
| :--- | :--- |
<br/>