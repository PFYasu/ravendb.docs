# Exploration Queries
---

{NOTE: }

* **Exploration Queries** form an additional layer of filtering that can be applied 
  to a dataset after its retrieval by [raw rql](../../client-api/session/querying/how-to-query#session.advanced.rawquery), 
  while the dataset is still held by the server.  

* The **retrieved dataset** is scanned and filtered **without requiring or creating an 
  index**, providing a way to conduct one-time explorations without creating an index that would 
  have to be maintained by the cluster.  

* You can filter the datasets retrieved by both **Index queries** and **Collection queries**.  

* Exploration queries need to be used 
  [with caution](../../indexes/querying/exploration-queries#when-should-exploration-queries-be-used) 
  since scanning and filtering all the data retrieved by a query cost substantial 
  [server resources and user waiting time](../../indexes/querying/exploration-queries#limit-the-query-and-prefer--for-recurring-queries) 
  when large datasets are handled.  
    {WARNING: }

    We recommend that you -  

    * [Limit](../../indexes/querying/exploration-queries#limit-the-query-and-prefer--for-recurring-queries) 
      the number of records that an exploration query filters.  
    * Use [where](../../indexes/querying/filtering) in recurring queries, 
      so the query would [use an index](../../indexes/querying/exploration-queries#limit-the-query-and-prefer--for-recurring-queries).  

    {WARNING/}

* In this page:  
   * [`filter`](../../indexes/querying/exploration-queries#filter)
   * [When should exploration queries be used](../../indexes/querying/exploration-queries#when-should-exploration-queries-be-used)
   * [Syntax](../../indexes/querying/exploration-queries#syntax)
   * [Usage examples](../../indexes/querying/exploration-queries#usage-examples)
      * [With collection queries](../../indexes/querying/exploration-queries#with-collection-queries)
      * [With queries that use an index](../../indexes/querying/exploration-queries#with-queries-that-use-an-index)
      * [With projections](../../indexes/querying/exploration-queries#with-projections)
      * [With user-defined JavaScript functions (`declare`)](../../indexes/querying/exploration-queries#with-user-defined-javascript-functions-)

{NOTE/}

{PANEL: `filter`}

Exploration queries can be applied via RQL using the `filter` keyword.  

The added filtering is parsed and executed by RavenDB's Javascript engine.  

The provided filtering operations resemble those implemented by 
[where](../../indexes/querying/filtering) and can be further enhanced 
by Javascript functions of your own.  
Read [here](../../indexes/querying/exploration-queries#with-user-defined-javascript-functions-) 
about creating and using your own Javascript function in your filters.  

{PANEL/}

{PANEL: When should exploration queries be used}

`filter` can be applied to a Collection query, like in:  
{CODE-BLOCK: sql}
from Employees as e 
filter e.Address.Country = 'USA'
{CODE-BLOCK/}

it can also be applied to queries handled by an index, e.g. -  

{CODE-BLOCK: sql}
// in a dynamic query via an auto-index  
from Employees as e 
where e.Title = 'Sales Representative'  
filter e.Address.Country = 'USA'
{CODE-BLOCK/}

{CODE-BLOCK: sql}
// in a query that uses an index explicitly  
from index 'Orders/ByCompany' 
filter Count > 10
{CODE-BLOCK/}

Both in a collection query and in a query handled by an index, the entire retrieved 
dataset is scanned and filtered.  
This helps understand when exploration queries should be used, why a Limit 
should be set for the number of filtered records, and when `where` should 
be preferred:  

{INFO: }
#### When to use
Use `filter` for an ad-hoc exploration of the retrieved dataset, that matches 
no existing index and is not expected to be repeated much.  

* You gain the ability to filter post-query results on the server side, for 
  both collection queries and when an index was used.  
* The dataset will be filtered without creating an unrequired index that the cluster 
  would continue updating from now on.  
{INFO/}
{WARNING: }
#### Limit the query, and prefer `where` for recurring queries 
Be aware that when a large dataset is retrieved, like the whole collection in 
the case of a collection query, exploring it all using `filter` would tax the server 
in memory and CPU usage while it checks the filter condition for each query result, 
and cost the user a substantial waiting time. Therefore -  

* **Limit** the number of records that an exploration query filters, e.g.:  
  {CODE-BLOCK: sql}
from Employees as e 
filter e.Address.Country = 'USA'
filter_limit 500 // limit the number of filtered records
  {CODE-BLOCK/}
* Use [where](../../indexes/querying/filtering) rather than `filter` for recurring filtering.  
  `where` will use an index, creating it if necessary, to accelerate the filtering 
  in subsequent queries.  
{WARNING/}

{PANEL/}

{PANEL: Syntax}

In an RQL query, use:  
The `filter` keyword, followed by the filtering condition.  
The `filter_limit` option, followed by the max number of records to filter.  

E.g. -  
{CODE-BLOCK: sql}
from Employees as e 
where e.Title = 'Sales Representative'  
filter e.Address.Country = 'USA' // filter the retrieved dataset
filter_limit 500 // limit the number of filter records
     {CODE-BLOCK/}
{PANEL/}

{PANEL: Usage examples}

#### With collection queries

Use `filter` with a collection query to scan and filter the entire collection.  
{CODE-TABS}
{CODE-TAB:php:query exploration-queries_1.1@Indexes\Querying\ExplorationQueries.php /}
{CODE-TAB:php:documentQuery exploration-queries_1.2@Indexes\Querying\ExplorationQueries.php /}
{CODE-TAB:php:rawQuery exploration-queries_1.3@Indexes\Querying\ExplorationQueries.php /}
{CODE-TABS/}

{WARNING: }
Filtering a sizable collection will burden the server and prolong user waiting time.  
Set a `filter_limit` to restrict the number of filtered records.  
{WARNING/}

---

#### With queries that use an index

Use `filter` after a `where` clause to filter the results retrieved by an index query.  
{CODE-TABS}
{CODE-TAB:php:query exploration-queries_2.1@Indexes\Querying\ExplorationQueries.php /}
{CODE-TAB:php:documentQuery exploration-queries_2.2@Indexes\Querying\ExplorationQueries.php /}
{CODE-TAB:php:rawQuery exploration-queries_2.3@Indexes\Querying\ExplorationQueries.php /}
{CODE-TABS/}

---

#### With projections

The filtered results can be projected using `select`, like those of any other query.  
{CODE-TABS}
{CODE-TAB:php:query exploration-queries_3.1@Indexes\Querying\ExplorationQueries.php /}
{CODE-TAB:php:rawQuery exploration-queries_3.3@Indexes\Querying\ExplorationQueries.php /}
{CODE-TABS/}

---

#### With user-defined JavaScript functions (`declare`)

You can define a Javascript function as part of your query using the 
[declare](../../client-api/session/querying/what-is-rql#declare) keyword, and 
use it as part of your `filter` condition to freely adapt the filtering 
to your needs.  

Here is a simple example:  
{CODE-BLOCK: sql}
// declare a Javascript function
declare function titlePrefix(r, prefix) 
{ 
    // Add whatever filtering capabilities you like
    return r.Title.startsWith(prefix)
} 

from Employees as e 

// Filter using the function you've declared
filter titlePrefix(e, $prefix)
filter_limit 100

{CODE-BLOCK/}

{PANEL/}

## Related Articles

### Querying

- [What is RQL](../../client-api/session/querying/what-is-rql)  
- [Query](../../client-api/session/querying/how-to-query#session.query)  
- [DocumentQuery](../../client-api/session/querying/how-to-query#session.advanced.documentquery)  
- [RawQuery](../../client-api/session/querying/how-to-query#session.advanced.rawquery)  
- [Where](../../indexes/querying/filtering)  
- [Querying an Index](../../indexes/querying/query-index)  
- [Sorting](../../indexes/querying/sorting)  
- [declare](../../client-api/session/querying/what-is-rql#declare)  
