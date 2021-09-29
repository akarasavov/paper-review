# Abstract
* key-value store
* parallel snapshot isolation, resolution of write-write conflicts

# Introduction
* system designed for a social network
* geo-replicated key-value store that provides PSI. Avoid write-write conflicts wihout cross-site comm. using - prefered sites and cstets

# Overview

Snapshot isolation - if two transactions have write-write conflict one must be aborted.

PSI 
- transactions can have different commit order.
- guarantee casual order if T2 reads from T1 then T1 < T2
- well suited for web applications

Preferred sites - place where writes to the object can be committed

Conflict-free counting set objects - object that can be modified frequently from many places. 
It is similar to multi-set and can have negative values. Add(x) - incriment X, Remove(x) - decrement X

# Parallel snapshot isolation

## Snapshot isolation

Specification of snapshot isolation level:
* (Snapshot read) In the moment when transaction start all operations read the most recent committed version
* (No write-write conflicts) The write sets foreach pair of committed concurent transactions must be disjoint

Problems with SI
* We need a total ordering of commit time of all tx
* The writes of committed tx must be immediately. Writes are asynch so we need to wait until they are propagated to all replicas.  

## Specification of Parallel snapshot isolation

Specification PSI
* (Site snapshot read) All operations read the most recent committed version at the transaction's sites as of the time when the transaction began
* (No write-write conflicts) The writes sets of each pair of committed somewhere-concurent transactions must be disjoint
* (Commit causality across sites ) If Tx1 commits before start of Tx2, then T1 cannot commit after T2

## Using PSI

Anomalies that ca be seen in Walter:
* write skew(short fork)
* long fork


Benefits for using PSI:
* Multi-object atomic updates
* Snapshots - read from a fixed consistent snapshot
* Read-modify-write operations - CAS
* Conditional writes - modification of CAS that write values if it is exactly version

# Services

Walter stores key-value grouped in containers that a replicated across multiple sites.

## Objects and containers

Walter stores object, where object{key,value}. Object can be a regular(sequence of bytes) or cset(vector).
Objects are stored in container, logical organization unit.

## Interface

Every container is replicated on several sites depending on the configuration of the administrator.

## Durability and availability
No usage of consistent protocol.

Normal durability - writes of a tx is logged to durable distributed file system

Disaster durability - tx is disaster safe if its have been logged to f+1 sites.


# Design and algorithms

## Basic architecture

There a multiple sites in the system. Each site contain a Walter server and set of clients.
There is configuration server that keep track of the current active sites and mapping {container, site}.

## Versions and vector timestamps

 
