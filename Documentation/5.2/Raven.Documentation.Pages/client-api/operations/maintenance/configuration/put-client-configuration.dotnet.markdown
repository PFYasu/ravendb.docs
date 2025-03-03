# Put Client Configuration Operation <br> (for database)

---

{NOTE: }

* __What is the Client-Configuration__:  
  The Client-Configuration is a set of configuration options that apply to the client when communicating with the database.
  See [what can be configured](../../../../client-api/operations/maintenance/configuration/put-client-configuration#what-can-be-configured) below.  

* __Initializing the Client-Configuration__ (on the client):  
  This configuration can be initially customized from the client code when creating the Document Store via the [Conventions](../../../../client-api/configuration/conventions).
  
* __Overriding the initial Client-Configuration for the database__ (on the server):  

    * From the client code:  
      Use `PutClientConfigurationOperation` to set the client-configuration options on the server.  
      See the example below.
    
    * From the Studio:  
      Set the Client-Configuration from the [Client-Configuration view](../../../../studio/database/settings/client-configuration-per-database).

* __Updating the running client__:  

  * Once the Client-Configuration is modified on the server, the running client will [receive the updated settings](../../../../client-api/configuration/load-balance/overview#keeping-the-client-topology-up-to-date)
    the next time it makes a request to the database.  

  * Setting the Client-Configuration on the server enables administrators to dynamically control the client behavior after it has started running.  
    e.g. manage load balancing of client requests on the fly in response to changing system demands.

* The client-configuration set for the database level __overrides__ the [server-wide client-configuration](../../../../client-api/operations/server-wide/configuration/put-serverwide-client-configuration).

---

* In this page:
  * [What can be configured](../../../../client-api/operations/maintenance/configuration/put-client-configuration#what-can-be-configured)
  * [Put client-configuration (for database)](../../../../client-api/operations/maintenance/configuration/put-client-configuration#put-client-configuration-(for-database))
  * [Syntax](../../../../client-api/operations/maintenance/configuration/put-client-configuration#syntax)

{NOTE /}

---

{PANEL: What can be configured}

The following client-configuration options are available:  

* __Identity parts separator__:  
  Set the separator used with automatically generated document identity IDs (default separator is `/`).  
  Learn more about identity IDs creation [here](../../../../server/kb/document-identifier-generation#identity).

* __Maximum number of requests per session__:  
  Set this number to restrict the number of requests (Reads & Writes) per session in the client API.

* __Read balance behavior__:  
  Set the Read balance method the client will use when accessing a node with Read requests.  
  Learn more in [Balancing client requests - overview](../../../../client-api/configuration/load-balance/overview) and [Read balance behavior](../../../../client-api/configuration/load-balance/read-balance-behavior).
  
* __Load balance behavior__:  
  Set the Load balance method for Read & Write requests.  
  Learn more in [Load balance behavior](../../../../client-api/configuration/load-balance/load-balance-behavior).

{PANEL/}

{PANEL: Put client-configuration (for database)}

{CODE put_config_1@ClientApi\Operations\Maintenance\Configuration\PutClientConfig.cs /}

{CODE-TABS}
{CODE-TAB:csharp:Sync put_config_2@ClientApi\Operations\Maintenance\Configuration\PutClientConfig.cs /}
{CODE-TAB:csharp:Async put_config_3@ClientApi\Operations\Maintenance\Configuration\PutClientConfig.cs /}
{CODE-TABS/}

{PANEL/}

{PANEL: Syntax}

{CODE syntax_1@ClientApi\Operations\Maintenance\Configuration\PutClientConfig.cs /}

| Parameter         | Type                  | Description                                                            |
|-------------------|-----------------------|------------------------------------------------------------------------|
| __configuration__ | `ClientConfiguration` | Client configuration that will be set on the server (for the database) |

{CODE syntax_2@ClientApi\Operations\Maintenance\Configuration\PutClientConfig.cs /}

{PANEL/}

## Related Articles

### Studio

- [Client Configuration View](../../../../studio/database/settings/client-configuration-per-database)

### Operations

- [What are Operations](../../../../client-api/operations/what-are-operations)
- [Get client-configuration (for database)](../../../../client-api/operations/maintenance/configuration/get-client-configuration)
- [Get client-configuration (server-wide)](../../../../client-api/operations/server-wide/configuration/get-serverwide-client-configuration)
- [Put client-configuration (server-wide)](../../../../client-api/operations/server-wide/configuration/put-serverwide-client-configuration)


### Load balancing

- [Load balancing client requests](../../../../client-api/configuration/load-balance/overview)
