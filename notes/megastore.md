# Megastore
Megastore blends the scalability of a NoSQL datastore with the convenience of a traditional RDBMS in a novel way, and
provides both strong consistency guarantees and high availability.
[CIDR11_Paper](https://www.cidrdb.org/cidr2011/Papers/CIDR11_Paper32.pdf)

## Replication
**Replica Types**
* full replicas
* Read-only replicas：the inverse of witnesses: they are
non-voting replicas that contain full snapshots of the data.
Reads at these replicas reflect a consistent view of some
point in the recent past.
* witness replicas: Witnesses vote in Paxos rounds and store the writeahead log, but do not apply the log and do not store entity
data or indexes, so they have lower storage costs.

## Partitioning and Locality
Entity Group

To scale throughput and localize outages, we partition our
data into a collection of entity groups, each independently and synchronously replicated over a wide area. The
underlying data is stored in a scalable NoSQL datastore in
each datacenter (see Figure 1).
Entities within an entity group are mutated with singlephase ACID transactions (for which the commit record is replicated via Paxos). Operations across entity groups could
rely on expensive two-phase commits, but typically leverage
Megastore’s efficient asynchronous messaging.

<img src="https://github.com/seast/system-design/blob/master/images/megastore_entity_group.png" align="middle" width="60%">

## Read and Write
**Write**

Bigtable provides the ability to store multiple values in the
same row/column pair with different timestamps. We use
this feature to implement multiversion concurrency control
(MVCC): when mutations within a transaction are applied,
the values are written at the timestamp of their transaction.
Readers use the timestamp of the last fully applied transaction to avoid seeing partial updates. Readers and writers
don’t block each other, and reads are isolated from writes
for the duration of a transaction.

**Read**

Megastore provides current, snapshot, and inconsistent reads.

