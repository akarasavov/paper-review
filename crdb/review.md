# CockroachDB: The Resilient Geo-Distributed
SQL Database

## Requirements to the system
- Fault tolerance and high availability
- Geo-distributed partitioning and replica placement
- High-performance transactions


## Architecture

- SQL - interface for all user interactions with DB
- Transaction KV - responsible for atomicity and isolation guarantees
- Distribution - Abstraction of a monolithic logical key space ordered by key.
    * Range - Ordered chunk of keys(size 64Mb) that are replicated across the cluster. It is used by query optimizer to route the query  
- Replication - layer responsible for replication the ranges(Raft)
- Storage - represent local disk(RocksDB)

### How CRDB achieves fault tolerance and high availability?
* Raft group - replicas of a range(raft group - 1l and 2 followers)
* Replica placement - there is a manual and automatically placement strategies. Attributes that are used to make a decision - CPU,RAM,DISK and country, region, av. zone


#### Data placement policies
* Geo-partitioned replicas - partitioned by access location. Fast intra-region reads and intra-region writes
* Geo-partitioned leaseholders - slow cross-region writes, fast intra-region reads
* Duplicated indexes - duplicate indexes and pining each index leaseholder to a specific region -> server fast local reads while retaining the ability to survive regional failure.

## Transactions
//TODO
