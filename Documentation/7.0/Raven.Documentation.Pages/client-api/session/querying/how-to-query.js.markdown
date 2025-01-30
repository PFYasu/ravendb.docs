# Query Overview

---

{NOTE: }

* Queries in RavenDB can be written with:
    * The session's `query` method - rich API is provided
    * The session's  `rawQuery` method - using RQL
    * The [Query view](../../../studio/database/queries/query-view) in Studio - using RQL

* Queries defined with the `query` method are translated by the RavenDB client to [RQL](../../../client-api/session/querying/what-is-rql)  
  when sent to the server.

---

* All queries in RavenDB use an **index** to provide results, even when you don't specify one.  
  Learn more [below](../../../client-api/session/querying/how-to-query#queries-always-provide-results-using-an-index).

* Queries that do Not specify which index to use are called **Dynamic Queries**.  
  This article displays examples of dynamic queries only.  
  For examples showing how to query an index see [querying an index](../../../indexes/querying/query-index).

---

* The entities returned by the query are 'loaded' and **tracked** by the [Session](../../../client-api/session/what-is-a-session-and-how-does-it-work).  
  Entities will Not be tracked when:  
    * Query returns a [projection](../../../client-api/session/querying/how-to-project-query-results)  
    * Tracking is [disabled](../../../client-api/session/configuration/how-to-disable-tracking#disable-tracking-query-results)  

* Query results are **cached** by default. To disable query caching see [noCaching](../../../client-api/session/querying/how-to-customize-query#nocaching).  

* Queries are timed out after a configurable time period.  See [query timeout](../../../server/configuration/database-configuration#databases.querytimeoutinsec).

---

* In this page:
  * [Queries always provide results using an index](../../../client-api/session/querying/how-to-query#queries-always-provide-results-using-an-index)  
  * [session.query](../../../client-api/session/querying/how-to-query#session.query)  
  * [session.advanced.rawQuery](../../../client-api/session/querying/how-to-query#session.advanced.rawquery)
  * [query API](../../../client-api/session/querying/how-to-query#query-api)  
  * [Syntax](../../../client-api/session/querying/how-to-query#syntax)

{NOTE/}

---

{PANEL: Queries always provide results using an index} 

* Queries always use an index to provide fast results regardless of the size of your data.

* When a query reaches a RavenDB instance, the instance calls its query optimizer to analyze the query  
  and determine which index should be used to retrieve the requested data.

* Indexes allow to provide query results without scanning the entire dataset each and every time.
    * Learn more about indexes, their general concept, and the different **index types** in this [indexes overview](../../../studio/database/indexes/indexes-overview) article.
    * You can choose the underlying **search engine** that will be used by the RavenDB indexes.  
      Learn more in [selecting the search engine](../../../indexes/search-engine/corax#selecting-the-search-engine).

{INFO: }

We differentiate between the following **3 query scenarios**:

1. [Index query](../../../client-api/session/querying/how-to-query#indexQuery)
2. [Dynamic query](../../../client-api/session/querying/how-to-query#dynamicQuery)
3. [Full collection query](../../../client-api/session/querying/how-to-query#collectionQuery)

For each scenario, a different index type is used, as described below.

{INFO/}

---

{NOTE: }

<a id="indexQuery" /> 
**1. Query an existing index**:

*  **Query type**: Index query  
   **Index used**: Static-index

* You can specify which **STATIC-static index** the query will use.

* Static indexes are defined by the user, as opposed to auto-indexes that are created by the server  
  when querying a collection with some filtering applied. See [Static-index vs Auto-index](../../../studio/database/indexes/indexes-overview#auto-indexes--vs--static-indexes).

* Example RQL: &nbsp; `from index "Employees/ByFirstName" where FirstName == "Laura"`  
  See more examples in [querying an index](../../../indexes/querying/query-index).

{NOTE/}
{NOTE: }

<a id="dynamicQuery" /> 
**2. Query a collection - with filtering (Dynamic Query)**:

*  **Query type**: Dynamic Query  
   **Index used**: Auto-index

* When querying a collection without specifying an index and with some filtering condition  
  (other than just the document ID) the query-optimizer will analyze the query to see if an **AUTO-index**  
  that can answer the query already exists, i.e. an auto-index on the collection queried with index-fields that match those queried.

* If such auto-index (Not a static one...) is found, it will be used to fetch the results.

* Else, if no relevant auto-index is found,  
  the query-optimizer will create a new auto-index with fields that match the query criteria.  
  At this time, and only at this time, the query will wait for the auto-indexing process to complete.  
  Subsequent queries that target this auto-index will be served immediately.

* Note: if there exists an auto-index that is defined on the collection queried  
  but is indexing a different field than the one queried on,  
  then the query-optimizer will create a new auto-index that merges both the  
  fields from the existing auto-index and the new fields queried.

* Once the newly created auto-index is done indexing the data,  
  the old auto-index is removed in favor of the new one.

* Over time, an optimal set of indexes is generated by the query optimizer to answer your queries.

* Example RQL: &nbsp; `from Employees where FirstName == "Laura"`  
  See more examples [below](../../../client-api/session/querying/how-to-query#session.query).

---

* Note: Counters and Time series are an exception to this flow.  
  Dynamic queries on counters and time series values don't create auto-indexes.  
  However, a static-index can be defined on [Time series](../../../document-extensions/timeseries/indexing) and [Counters](../../../document-extensions/counters/indexing).

{NOTE/}
{NOTE: }

<a id="collectionQuery" /> 
**3. Query a collection - no filtering**:

* **Query type**: Full collection Query  
  **Index used**: The raw collection (internal storage indexes)

* Full collection query:

  * When querying a collection without specifying an index and with no filtering condition,  
    then all documents from the specified collection are returned.

  * RavenDB uses the raw collection documents in its **internal storage indexes** as the source for this query.  
    No auto-index is created.

  * Example RQL: &nbsp; `from Employees`

* Query by document ID:
 
  * When querying a collection only by document ID or IDs,  
    then similar to the full collection query, no auto-index is created.  
  
  * RavenDB uses the raw collection documents as the source for this query.
  
  * Example RQL: &nbsp; `from Employees where id() == "employees/1-A"`  
    See more examples [below](../../../client-api/session/querying/how-to-query#session.query).

{NOTE/}
{PANEL/}

{PANEL: session.query}

* The simplest way to issue a query is by using the session's `query` method.    
  Customize your query with these [API methods](../../../client-api/session/querying/how-to-query#query-api).

* The following examples show **dynamic queries** that do not specify which index to use.  
  Please refer to [querying an index](../../../indexes/querying/query-index) for other examples.

{NOTE: }

**Query collection - no filtering** 

{CODE-TABS}
{CODE-TAB:nodejs:Query query_1_0@client-api\session\querying\howToQuery.js /}
{CODE-TAB:nodejs:Query_overload query_1_1@client-api\session\querying\howToQuery.js /}
{CODE-TAB-BLOCK:sql:RQL}
// This RQL is a Full Collection Query
// No auto-index is created since no filtering is applied

from "employees"
{CODE-TAB-BLOCK/}
{CODE-TABS/}

{NOTE/}

{NOTE: }

**Query collection - by ID**

{CODE-TABS}
{CODE-TAB:nodejs:Query query_2_0@client-api\session\querying\howToQuery.js /}
{CODE-TAB:nodejs:Query_overload query_2_1@client-api\session\querying\howToQuery.js /}
{CODE-TAB-BLOCK:sql:RQL}
// This RQL queries the 'Employees' collection by ID
// No auto-index is created when querying only by ID

from "employees" where id() == "employees/1-A"
{CODE-TAB-BLOCK/}
{CODE-TABS/}

{NOTE/}

{NOTE: }

**Query collection - with filtering** 

{CODE-TABS}
{CODE-TAB:nodejs:Query query_3_0@client-api\session\querying\howToQuery.js /}
{CODE-TAB:nodejs:Query_overload query_3_1@client-api\session\querying\howToQuery.js /}
{CODE-TAB-BLOCK:sql:RQL}
// Query collection - filter by document field

// An auto-index will be created if there isn't already an existing auto-index
// that indexes the requested field

from "employees" where firstName == "Robert"
{CODE-TAB-BLOCK/}
{CODE-TABS/}

{NOTE/}

{NOTE: }

**Query collection - with paging** 

{CODE-TABS}
{CODE-TAB:nodejs:Query query_4_0@client-api\session\querying\howToQuery.js /}
{CODE-TAB:nodejs:Query_overload query_4_1@client-api\session\querying\howToQuery.js /}
{CODE-TAB-BLOCK:sql:RQL}
// Query collection - page results
// No auto-index is created since no filtering is applied

from "products" limit 5, 10 // skip 5, take 10
{CODE-TAB-BLOCK/}
{CODE-TABS/}

* By default, if the page size is not specified, all matching records will be retrieved from the database.

{NOTE/}

{PANEL/}

{PANEL: session.advanced.rawQuery}

* Queries defined with [query](../../../client-api/session/querying/how-to-query#session.query) are translated by the RavenDB client to [RQL](../../../client-api/session/querying/what-is-rql) when sent to the server.  

* The session also gives you a way to express the query directly in RQL using the `rawQuery` method.

**Example**:

{CODE:nodejs query_5@client-api\session\querying\howToQuery.js /}

{PANEL/}

{PANEL: query API}

Available methods for the session's [query](../../../client-api/session/querying/how-to-query#session.query) method:

- addOrder
- addParameter
- aggregateBy
- aggregateUsing
- andAlso
- any
- [boost](../../../client-api/session/querying/text-search/boost-search-results)
- closeSubclause
- containsAll
- containsAny
- [count](../../../client-api/session/querying/how-to-count-query-results)
- [countLazily](../../../client-api/session/querying/how-to-perform-queries-lazily#lazy-count-query)
- distinct
- first
- firstOrNull
- fuzzy
- getIndexQuery
- getQueryResult
- [groupBy](../../../client-api/session/querying/how-to-perform-group-by-query)
- [highlight](../../../client-api/session/querying/text-search/highlight-query-results)
- include
- [includeExplanations](../../../client-api/session/querying/debugging/include-explanations)
- [intersect](../../../client-api/session/querying/how-to-use-intersect)
- lazily
- [longCount](../../../client-api/session/querying/how-to-count-query-results)
- [moreLikeThis](../../../client-api/session/querying/how-to-use-morelikethis)
- negateNext
- [noCaching](../../../client-api/session/querying/how-to-customize-query#nocaching)
- [noTracking](../../../client-api/session/querying/how-to-customize-query#notracking)
- not
- [ofType](../../../client-api/session/querying/how-to-project-query-results#oftype)
- [on("afterQueryExecuted")](../../../client-api/session/querying/how-to-customize-query#on-("afterqueryexecuted"))
- [on("beforeQueryExecuted")](../../../client-api/session/querying/how-to-customize-query#on-("beforequeryexecuted"))
- openSubclause
- [orderBy](../../../client-api/session/querying/sort-query-results)
- [orderByDescending](../../../client-api/session/querying/sort-query-results)
- [orderByDistance](../../../client-api/session/querying/how-to-make-a-spatial-query#orderByDistance)
- [orderByDistanceDescending](../../../client-api/session/querying/how-to-make-a-spatial-query#orderByDistanceDesc)
- [orderByScore](../../../client-api/session/querying/sort-query-results#order-by-score)
- [orderByScoreDescending](../../../client-api/session/querying/sort-query-results#order-by-score)
- orElse
- proximity
- [randomOrdering](../../../client-api/session/querying/how-to-customize-query#randomordering)
- relatesToShape
- search
- [selectFields](../../../indexes/querying/projections#selectfields)
- selectTimeSeries
- single
- singleOrNull
- skip
- spatial
- [statistics](../../../client-api/session/querying/how-to-get-query-statistics)
- [suggestUsing](../../../client-api/session/querying/how-to-work-with-suggestions)
- take
- timings
- usingDefaultOperator
- [waitForNonStaleResults](../../../client-api/session/querying/how-to-customize-query#waitfornonstaleresults)
- whereBetween
- [whereEndsWith](../../../client-api/session/querying/text-search/ends-with-query)
- whereEquals
- [whereExists](../../../client-api/session/querying/how-to-filter-by-field)
- whereGreaterThan
- whereGreaterThanOrEqual
- whereIn
- whereLessThan
- whereLessThanOrEqual
- [whereLucene](../../../client-api/session/querying/document-query/how-to-use-lucene)
- whereNotEquals
- [whereRegex](../../../client-api/session/querying/text-search/using-regex)
- [whereStartsWith](../../../client-api/session/querying/text-search/starts-with-query)
- withinRadiusOf

{PANEL/}

{PANEL: Syntax}

{CODE:nodejs syntax@client-api\session\Querying\howToQuery.js /}

| Parameter        | Type                          | Description                  |
|------------------|-------------------------------|------------------------------|
| **documentType** | object                        | The type of entities queried |
| **index**        | object                        | The index class              |
| **opts**         | `DocumentQueryOptions` object | Query options                |
| **query**        | string                        | The RQL query string         |

| `DocumentQueryOptions` | | |
| - | - | - |
|  **collection** | string | <ul><li>Collection name queried</li></ul> |
|  **indexName** | string | <ul><li>Index name queried</li></ul> |
|  **index** | object | <ul><li>Index object queried</li><li>Note:<br>`indexName` & `index` are mutually exclusive with `collection`.<br>See examples in [querying an index](../../../indexes/querying/query-index).</li></ul> |

| Return Value | |
| - | - |
| `object` | Instance implementing `IDocumentQuery` exposing the additional [query methods](../../../client-api/session/querying/how-to-query#query-api). |

* Note:  
  Use `await` when executing the query, e.g. when calling `.all`, `.single`, `.first`, `.count`, etc.  
{PANEL/}

## Related Articles

### Session

- [How to Project Query Results](../../../client-api/session/querying/how-to-project-query-results)
- [How to Stream Query Results](../../../client-api/session/querying/how-to-stream-query-results)
- [What is a Document Query](../../../client-api/session/querying/document-query/what-is-document-query)

### Querying

- [Querying an Index](../../../indexes/querying/query-index)
- [Filtering](../../../indexes/querying/filtering)
- [Paging](../../../indexes/querying/paging)
- [Sorting](../../../indexes/querying/sorting)
- [Projections](../../../indexes/querying/projections)

### Indexes

- [What are Indexes](../../../indexes/what-are-indexes)  
- [Indexing Basics](../../../indexes/indexing-basics)
- [Map Indexes](../../../indexes/map-indexes)
