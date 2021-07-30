Main benefits:

- Writers don't block readers
- Readonly txns can read a consistent snapshot without acquiring locks
- Time-travel quaries

## Introduction

- Most of DBMS use tuples because it provides a good balance between parallelism versus the overhead of version tracking ?

## MVCC Overview

- Allows greater concurrency that in single version system - allows a transaction to read an older version on an object at the same time that another transaction  updates that same object.

### DBMS Meta-data

**Transactions** - every transaction has a monotonically increasing timestamp as it identifier(Tid) when they first enter the system.

**Tuples**

txn-id - server as a write lock. Any transaction that attempts to update A is aborted if this txn-id field is neither zero or not equal to its Tid. 

begin-ts, end-ts - represent the lifetime of the tuple 

pointer - store the address of neighbouring version

## Concurrency control protocol

**Timestamp Ordering(MVTO)**

In addition to all tuples described in DBMS Meta-data section we add read-ts(last tx that read it).

1. Transaction T trie to read logical tuple A, then DBMS search for physical version where Tid is between begin-ts and end-ts.
2. T is allowed to read A tuple if its write lock is not held by another active tx, because MVTO never allows to read uncommited versions.
3. Upon read A the DBMS sets Ax's read-ts field to Tid if its current value is less then Tid. Otherwise the tx read older version

**Optimistic concurrency control(MVOCC)**

Split transaction on the phases

Read phase:

1. Search for visible version Ax based on begin-tx and end-tx
2. If A write lock is available T can update the it
3. If transaction is updates version Bx then DBMS create version Bx+1 with its txn-id set to Tid???

Validation phase:

Before commiting tx enters in validation phase. 

1. DBMS assigns the transaction another timestamp(Tcommit) to determine the serialisation order of transaction.
2. The DBMS then determines whether the tuples in the transaction read set was updated by transaction that already commited???

Write phase:

If the transaction passes these checks it then enters the write phase where the DBMS installs all the new versions and sets their begin-ts to Tcommit and end-ts to INF.

### Two-phase locking(MV2PL)

For disk based DBMS locks are stored separately from a logical representation.

**Tuples structure:**

Tuples write lock is tx-id field. For the read lock transaction use read-cnt field to count the number of active transactions that have read the tuple.

**Read operation:**

To perform read operation on tuple A, DBMS search visible version by comparing transaction Tid with tuples begin-ts fied. If it finds a valid version then DBMS incriments read-cnt if txn-id filed is equals to zero.

**Write operation:**

We can update tuple only if both read-cnt and tnx-id are set to zero. When transaction commits the DBMS assigns it a unique timestamp(Tcommit) that is used to update the begin-ts field for the versions created by that transaction.

### Snapshot isolation

When Tx starts it see only a data that was committed before its start.

- If two txns update the same object then first writer wins

## Version Storage

Dbms use the tuples pointer to create latch-free version chain per logical tuple.

- This allows the DBMS to find the version that is visible to a particular txn at runtime
- Indexes always point to the head of the chain

### **Append-only storage**

All of the physical versions of a logical tuple are stored in the same table space. The versions are mixed together.

**Oldest-to-Newest**

- If you have a workloads that search always the oldest version (benefits)

**Newest-to-oldest**

- You have to update the header every time when you have a new tx(downside)
- Good if you always want to find the newest tupple

## Time-travel storage

We support two tables. Main-table and time-travel table

Main table always store the latest version. Time-travel table store the old copied versions.

Benefits:

- When you do gardbadge collection you can blow the time-travel table
- Sequential scan are very easy on the main table because main table contains only latest versions

## Delta storage

We support two table. Main-table contains master value and delta-storage contain records where every record is delta, pointer. Pointer points to prev value. Delta is value before modification.

On every update copy only the values that were modified to the delta storage and overwrite the master version.

Benefits:

- If I update one field I need to record in delta storage only the change of this field. In time-travel storage I need to copy all the filed and change the value that was changes == Less copying

Disadvantage:

- More work to recreate the original values

 

## Garbage collection

The DBMS needs to remove reclaimable physical versions from the database over [time.](http://time.To) To do this we need to solve this problems

- No active tx in DBMS can see the version that we want to delete?
- The version was created by an aborted txn

Design decisions:

1. How to look for expired versions?
2. How to decide when it is safe to reclaim memory?
3. Where to look for expired versions?

### Tuple-level

**Background Vacuuming:**

Separate threads periodically scan the table and look for reclaimable versions.  

**Cooperative Cleaning:**

Every transaction when read some data in addition of reading data is responsible for cleaning data. There is a shared data  structure between all tx for this purpose. While traversing transaction can make a decision to delete an tupple based on - old version of tuple, no one use this tuple.

## Index menagment

### **Primary indexes**

Primary indexes always point to version chain head.

### **Secondary Index**

**Logical pointers**

- Use a fix identifier per tuple that does not change
- Requires and extra indirection layer - use a value from the primary index or use cash(map<TupleId, Address>) to do a lookup

**Physical pointers**

Only applicable for append-only storage.

- Store the physical address of versions in the index entries.

Discussion:

Like the other design decisions, these index management schemes perform differently on varying workloads. The logical pointer ap- proach is better for write-intensive workloads, as the DBMS updates the secondary indexes only when a transaction modifies the in- dexes attributes. Reads are potentially slower, however, because the DBMS traverses version chains and perform additional key compar- isons. Likewise, using physical pointers is better for read-intensive workloads because an index entry points to the exact version. But it is slower for update operations because this scheme requires the DBMS to insert an entry into every secondary index for each new version, which makes update operations slower.

## MVCC design decisions

CC protocol: ?

Version storage: deltas

Garbadge collection: Tuple-level vacuuming

Indexes: Logical Pointers

## Experimental results

Short transactions

The short transaction workload results in Fig. 6a show that all but one of the protocols scales almost linearly up to 24 threads. The main bottleneck for all of these protocols is the cache coherence traffic from updating the memory manager’s counters and checking for conflicts when transactions commit (even though there are no writes)

[Morning talk ](https://www.notion.so/Morning-talk-c31c31973baa4a998195f43bd0952122)