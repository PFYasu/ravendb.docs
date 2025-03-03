﻿# Replication Overview
---

{NOTE: }

* Replication in RavenDB is the process of __transferring data__ from one database instance to another.  

* __Data is highly available__ as _Reads & Writes_ can be done on any cluster node.  
  _Writes_ are always accepted and are automatically replicated to the other nodes,  
  either within the database-group, or to the nodes defined by a replication task,  
  (see the different types below).

* __Conflicts__, which may be involved when the data replicates,  
  are resolved according to the defined [conflict resolution](../../../server/clustering/replication/replication-conflicts) policy on the receiving end.

* In this page:  
    * [Replication types](../../../server/clustering/replication/replication#replication-types)  
      * [Internal replication](../../../server/clustering/replication/replication#internal-replication)
      * [External replication](../../../server/clustering/replication/replication#external-replication)
      * [Hub/Sink replication](../../../server/clustering/replication/replication#hubsink-replication)
    * [What is replicated](../../../server/clustering/replication/replication#what-is-replicated)  
    * [How replication works](../../../server/clustering/replication/replication#how-replication-works)  
    * [Replication & transaction boundary](../../../server/clustering/replication/replication#replication-&-transaction-boundary)  

{NOTE/}

---

{PANEL: Replication types}

#### Internal replication

* __Replicate between__: All database-group nodes.  
  __Handled by__: Automatically handled by the [Database-Group](../../../studio/database/settings/manage-database-group) nodes.  
  __Direction__: Master-to-master replication among all database-group nodes.  
  __Filtering__: No filtering is available. Data is replicated as exists on the source.  
  __Conflicts__: Handled by the database resolution policy, which is the same for all the database instances.  
  __Delays__: There are no delays, data is immediately replicated.

* __Usage__: This replication keeps the database data in sync across the database-group nodes.  
  The data is highly available as _reads & writes_ can be done on any of the nodes.  

* You can _write_ to any node in the database group,  
  that _write_ will be recorded and automatically replicated to all other nodes in the database-group.

---

#### External replication

* __Replicate between__: Two databases that are typically set on different clusters.   
  __Handled by__: Handled by the ongoing [External Replication Task](../../../server/ongoing-tasks/external-replication) defined by the user.  
  __Direction__: One-way replication.  
  __Filtering__: No filtering is available. Data is replicated as exists on the source.  
  __Conflicts__: Handled as defined by the destination database resolution policy.  
  __Delays__: Replication can be delayed as defined within the external-replication task.  

* __Usage__:  This replication allows you to have a live database replica in another cluster,  
  which can be used as a failover target.

* It is possible to define two such tasks on separate clusters that will replicate to one another.  

---

#### Hub/Sink replication
 
* __Replicate between__: Multiple Sinks that connect to a single Hub on different clusters.  
  __Handled by__: Handled by the ongoing [Hub/Sink Replication Tasks](../../../server/ongoing-tasks/hub-sink-replication) defined by the user.  
  __Direction__: Hub to Sink only, Sink to Hub only, or both directions (as defined by the task).  
  __Filtering__: Documents filtering is available when working with secure servers.  
  __Conflicts__: Handled as defined by the destination database resolution policy.  
  __Delays__: Replication can be delayed as defined within the Hub/Sink tasks.  
  
* __Usage__:  Data is replicated between the Hub and all Sinks connected to that Hub.  
  The connection is always triggered by the Sink.

{PANEL/}

{PANEL: What is replicated}

{NOTE: }
__The following database-items are replicated__:  

* Documents
* Revisions
* Attachments
* Conflicts
* Tombstones
* Counters
* Time Series

__Content replicated__:

* The content of the replicated items data is Not changed.  
* If content change is required then consider using [ETL tasks](../../../studio/database/tasks/ongoing-tasks/ravendb-etl-task#ravendb-etl-task--vs--replication-task) that use transformation scripts.
{NOTE/}

---

{NOTE: }
__The following cluster-level features are Not replicated__:  

* Index definitions and index data
* Ongoing tasks definitions
* Compare-exchange items
* Identities
* Conflict resolution scripts
 
---

* __With internal replication__:  
  When you define a cluster-level behavior, i.e. create an index,  
  then consistency between the database instances in the database-group is achieved by the [Raft Protocol](../../../server/clustering/rachis/what-is-rachis).

* __With a replication task__:    
  Replication controls only the flow of the data without dictating how it's going to be processed on the receiving end,
  thus different configurations can be defined on the source cluster and on the destination cluster.

{NOTE/}
{PANEL/}

{PANEL: How replication works}

* Each database instance holds a __TCP connection__ to each of the other database instance destinations.    
  With internal replication - the destinations are all other database-group nodes.  
  With a replication task - the destinations are the nodes defined in the task.  

* Whenever there is a _'write'_ on one instance,  
  it will be sent to all the other nodes in the database-group immediately over this connection.  
  If a replication task is also defined, then the data will also replicate to the destination database.

* Sending the data is done in an async manner.  
  If the database instance is unable to replicate the data, it will still accept that 'write action' and send it later.  

* Each database instance has its own local __database-ETag__.  
  This Etag increases on every storage write.  
  The item triggered the write will get that next consecutive number.  
  The order by which the items are replicated is set by their __item-ETag__, from low(oldest) to high(newest).  

* The data is sent in batches from the source to the destination.  
  Once the batch is processed successfully on the destination side,  
  the destination records the ETag of the last item it had received in that batch ( __last-accepted-ETag__ ).

* The destination sends a response back to the source with that last-accepted-ETag  
  so that the source will know where to continue sending the next batch from.

* In case of a replication failure, when sending the next batch, replication will start from the item  
  that has this last-accepted-ETag, which is known from the previous successful batch.

{PANEL/}

{PANEL: Replication & transaction boundary}

{NOTE: }
__Transactions atomicity__

* RavenDB guarantees that modifications made in the same transaction will always be replicated  
  to the destination in a __single batch__ and won't be broken into separate batches.  

* This is true for both the internal-replication and the replication-tasks.
{NOTE/}

{NOTE: }
__Replication & cluster-wide transactions__

* A [cluster-wide transaction](../../../client-api/session/cluster-transaction/overview), which is implemented by the [Raft Protocol](../../../server/clustering/rachis/consensus-operations#consensus-operations),  
  is either persisted on all database group nodes or rolled back on all upon failure.  

* After a cluster consensus is reached, the Raft command to be executed is propagated to all nodes.  
  When the command is executed locally on a node, if the items that are persisted are of the [type that replicates](../../../server/clustering/replication/replication#what-is-replicated),  
  then the node will __replicate__ them to the other nodes via the replication process.  

* A node that receives such replication will accept this write,  
  unless it has already committed it through a raft command it received before.
{NOTE/}

{NOTE: }
__Transaction boundaries in single-node transactions__

* If there are several document modifications in the same transaction they will be sent in the same replication
  batch, keeping the transaction boundary on the destination as well.

* However, when a document is modified in two separate transactions,  
  and if replication of the __1st__ transaction has not yet occurred,  
  then that document will Not be sent when the __1st__ transaction is replicated,  
  it will be sent with the __2nd__ transaction.

* If you care about all the modifications that were done then enable revisions:  
  When a revision is created for a document it is written as part of the same transaction as the document.  
  The revision is then replicated along with the document in the same indivisible batch.
{NOTE/}

{INFO: }

#### How revisions replication help data consistency

  Consider a scenario in which two documents, `Users/1` and `Users/2`, are **created in the same transaction**,  
  and then `Users/2` is **modified in a different transaction**.

  * **How will `Users/1` and `Users/2` be replicated?**  
      When RavenDB creates replication batches, it keeps the
      transaction boundary by always sending documents that were modified in the same transaction,
      **in the same batch**.  
      In our scenario, however, `Users/2` was modified after its creation, it
      is now recognized by its Etag as a part of a different transaction than
      that of `Users/1`, and the two documents may be replicated in two different
      batches, `Users/1` first and `Users/2` later.  
      If this happens, `Users/1` will be replicated to the destination without `Users/2`
      though they were created in the same transaction, causing a data inconsistency that
      will persist until the arrival of `Users/2`.

  * **The scenario will be different if revisions are enabled.**  
      In this case the creation of `Users/1` and `Users/2` will also create revisions
      for them both. These revisions will continue to carry the Etag given to them
      at their creation, and will be replicated in the same batch.  
      When the batch arrives at the destination, data consistency will be kept:  
      `Users/1` will be stored, and so will the `Users/2` revision, that will become
      a live `Users/2` document.

{INFO/}

{PANEL/}

## Related Articles

### Replication Within the Cluster

- [Replication Conflicts](../../../server/clustering/replication/replication-conflicts)
- [Change Vector](../../../server/clustering/replication/change-vector)
- [Load Balancing Client Requests](../../../client-api/configuration/load-balance/overview)
- [Advanced Replication Topics](../../../server/clustering/replication/advanced-replication)
- [Using Embedded Instance](../../../server/clustering/replication/replication-and-embedded-instance)
- [Cluster-Wide Transactions](../../../server/clustering/cluster-transactions)

### Replication Between Clusters

- [External Replication](../../../server/ongoing-tasks/external-replication)
- [Hub/Sink Replication](../../../server/ongoing-tasks/hub-sink-replication)

---

### Inside RavenDB

- [RavenDB Clusters](https://ravendb.net/learn/inside-ravendb-book/reader/4.0/6-ravendb-clusters#an-overview-of-a-ravendb-cluster)
- [Replication of data in a database group](https://ravendb.net/learn/inside-ravendb-book/reader/4.0/6-ravendb-clusters#replication-of-data-in-a-database-group)

### Ayende @ Rahien Blog

- [Data ownership in a geo-distributed system](https://ayende.com/blog/196769-B/data-ownership-in-a-distributed-system)
