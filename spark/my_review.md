# Introduction
* Propose a RDDS(resilient distributed datasets) optimized to run on iterative machine learning algorithms.
They are fault-tolerant, parallel, persist intermediate result into memory
* 20x faster that Hadoop for iterative apps

# RDDS
* DDSM allows read and writes to each memory location where RDDs is limited which allows us to build more efficient faul tolerance
* Only the lost partition of RDDS needs to be checkpointed

## Not Suitable apps for RDDs
* Good for apps that apply same operation on all elements of dataset
* Bad for finegrained updates to a global state - storage system for web apps or incirment web crawler

## Spark programming interface
* Narrow dependencies - each partition of the parent RDD is used by at most one partition child
* Wide dependencies - where multiple child partition can depend on the parent

The input of Rdds are hdfs files.
* partitions() returns one partition of each block of file
* preferedLocations() return the nodes of the block is on
* 

# Fault tolerance

## Failed worker - in narrow dependency
* Each worker is responsible for several partition
* If narrow dependency fail we need to all the previous graph steps(only lost dependency should be recomputed)

## Failed work - in wide dependency
* When we have wide dependency we need to recompute everything
