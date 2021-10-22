1.Motivation/idea/key observation: what is the key idea this paper is introducing or leveraging?
Motivation - build a geo-replicate key-value store that can be used to build a social network
Idea - Use PSI to implement distributed tx
Introducing - Parallel snapshot isolation - more relaxed isolation level that Snapshot isolation level with a better performance because PSI prevent transaction to have a write conflicts

2. System model
The paper target geo-replicate deployment.
There a multiple sites in the system. Each site contain a Walter server and set of clients.
There is configuration server that keep track of the current active sites and mapping {container, site}.

Before tx commits its write to a transaction log that is replicated trough distributed file system

3. Concurrency control 
PSI

4.Replication


System model: is the paper targeting geo-replicated deployments connected via WAN or within data center? Additionally, how is data assigned to shards? Is each shard an independently replicated Raft group, i.e., are we coordinating across raft clusters?
Concurrency control: what concurrency control approach is used, 2PL, OCC, TO, etc.?
Replication: are shards replicated either with Raft/Paxos/Viewstamped Replication or is a "2 in 1" approach used? Are they assuming synchronous replication (e.g. Raft) or asynchronous?
Atomic commitment/Determinism: is 2PC used or is a Calvin-style deterministic approach adopted? Assume 2PC, but no central co-ordinators
Workload assumptions/Transaction model: what transaction model is used (general interactive transactions, one-shot stored procedures)? Are read/write sets assumed known? Is the protocol designed for a low or high contention workload? Are assumptions made about transaction length and size, e.g., VoltDB assumes transactions are short-lived, access a small number of records at a time, and are repeatedly executed with different input parameters (fixed set of transaction profiles). Assume high contention/long running queries but need data to back this up
Isolation/consistency guarantees: what guarantees does the protocol provide? Two dimensions to consider (see Daniel Abadi's blog post).Happy with SI/RR. Would be nice a path to optionally higher isolation. Consistency still a question mark.
Time and clocks: are specific assumptions made around time, such as, use of atomic clocks (Spanner), or hybrid logical clocks (Cockroach)? No specials clocks yet. Perhaps we need them for leader leases?

