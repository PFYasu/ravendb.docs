# Enable Index Operation

 ---

{NOTE: }

* Use `EnableIndexOperation` to enable a specific index.  

* The index can be enabled either:  
    * On a single node, or  
    * Cluster wide - on all database-group nodes.  

* When index is enabled indexing will take place, new data will be indexed.  

* To disable the index use [disable index operation](../../../../client-api/operations/maintenance/indexes/disable-index).  
  Disabling/enabling an index can also be done from the [indexes list view](../../../../studio/database/indexes/indexes-list-view#indexes-list-view---actions) in the Studio.

* In this page:
    * [Enable index - single node](../../../../client-api/operations/maintenance/indexes/enable-index#enable-index---single-node)
    * [Enable index - cluster wide](../../../../client-api/operations/maintenance/indexes/enable-index#enable-index---cluster-wide)
    * [Syntax](../../../../client-api/operations/maintenance/indexes/enable-index#syntax)

{NOTE/}

---

{PANEL: Enable index - single node}

* With this option, the index will be enabled on the [preferred node](../../../../client-api/configuration/load-balance/overview#the-preferred-node) only.  
  The preferred node is simply the first node in the [database group topology](../../../../studio/database/settings/manage-database-group).

* Note: When enabling an index from the [Studio](../../../../studio/database/indexes/indexes-list-view#indexes-list-view---actions),  
  the index will be enabled on the local node the browser is opened on, even if it is Not the preferred node.

{CODE-TABS}
{CODE-TAB:csharp:Sync enable_1@ClientApi\Operations\Maintenance\Indexes\EnableIndex.cs /}
{CODE-TAB:csharp:Async enable_1_async@ClientApi\Operations\Maintenance\Indexes\EnableIndex.cs /}
{CODE-TABS/}

{PANEL/}

{PANEL: Enable index - cluster wide}

{CODE-TABS}
{CODE-TAB:csharp:Sync enable_2@ClientApi\Operations\Maintenance\Indexes\EnableIndex.cs /}
{CODE-TAB:csharp:Async enable_2_async@ClientApi\Operations\Maintenance\Indexes\EnableIndex.cs /}
{CODE-TABS/}

{PANEL/}

{PANEL: Syntax}

{CODE:csharp syntax@ClientApi\Operations\Maintenance\Indexes\EnableIndex.cs /}

| Parameters | Type | Description |
| - | - | - |
| **indexName** | string | Name of index to enable |
| **clusterWide** | bool | `true` - Enable index on all database-group nodes<br>`false` - Enable index only on a single node (the preferred node) |

{PANEL/}

## Related Articles

### Indexes

- [What are Indexes](../../../../indexes/what-are-indexes)
- [Creating and Deploying Indexes](../../../../indexes/creating-and-deploying)
- [Index Administration](../../../../indexes/index-administration)

### Operations

- [What are operations](../../../../client-api/operations/what-are-operations)
- [How to Disable Index](../../../../client-api/operations/maintenance/indexes/disable-index)
- [How to Pause Index Until Restart](../../../../client-api/operations/maintenance/indexes/stop-index)
