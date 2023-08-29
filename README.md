# Monitoring Azure Databricks with Log Analytics


Azure Databricks has an in-built [monitoring capability](https://docs.microsoft.com/en-gb/azure/databricks/administration-guide/account-settings/azure-diagnostic-logs). 

For additional capabilities or for integrating with other logs, or across workspaces, then using the standard Log Analytics integration is very useful.
Add [Diagnostic settings](/media/DiagnosticSettings.png).

For further useful logging references, see [this list of useful reading](/useful_reading.md).

## *Contents*

- [Introduction - Insights into logs](#introduction---insights-into-logs)
- [Specific scenario queries](#specific-queries-to-hightlight-key-information)
- [Understanding Cluster costs](#understanding-cluster-costs-from-cluster-information)
- [Understanding overall compute costs](#understanding-overall-compute-costs)
- [Understanding SQL Endpoint Usage](#understanding-sql-endpoint-usage)
- [Understanding the various logs](#understanding-the-various-logs)
  - [DatabricksClusters](#databricksclusters)
  - [DatabricksAccounts](#databricksaccounts)
  - [DatabricksJobs](#databricksjobs)
  - [DatabricksSQL](#sql-endpoint-logs)



## Introduction - Insights into logs

The following logs are the core of what is being examined here.

- DatabricksClusters
- DatabricksAccounts
- DatabricksJobs

These highlight some very basic information about Clusters, Jobs and Account activity.


## Specific queries to hightlight key information

- [Query DatabricksCluster to get clusterId](/loganalytics_queries/find_a_cluster_journey.kql)
  - detailed examination by each event in the cluster logs
- [Understanding Job Start and Ends](/loganalytics_queries/JobStart_Ends.kql)
  - Just using Jobs logs, get a list of all events
  - Using Jobs logs, join start and end to get a single row per job, with status
- [Link ADF to ADB and get Duration, Status and Cost](/loganalytics_queries/adf_linked_adb.kql)
  - Link ADF Activity Run logs to Databricks Cluster logs, to Databricks Jobs log, with Pool costs
  - Only Pools considered
  - For fuller discussion on costs, see [Cluster cost section](#understanding-cluster-costs-from-cluster-information)
- [Various cluster errors](/loganalytics_queries/cluster_error.kql)
- [Autorecovery records](/loganalytics_queries/cluster_resize_autorecovery.kql)

## Understanding Cluster costs from Cluster information

Databricks costs are broken into two main sections. Costs for Databricks Units (DBU) and costs for Azure resources.

The cost for DBU is relatively simple for PAYG use cases and has a relationship with the VM capability and the Databricks Tier of Service . If you have a reservation with Databricks for DBU, this is a little more complicated to work out, so will be handled seperately.

The cost for Azure resources largely related to VM, Storage, Networking etc, with the largest of these typically being the VM costs. 

You can use the Azure Pricing Calculator to get a rough guide on the cost of a particular cluster per hour. There are some variations that occur, such has how many disks get added to each node, but we will not consider that here. To populate the below table, choose your Databricks workload type and region, the VM family and type and add the per hour compute cost, DBU for that compute and the DBU per hour price into the table.

Using the Cluster logs and the create event, we can discover the pool id and number of workers for jobs clusters.

We can also create a definition for the costs of each pool, using information from the Azure Pricing Calculator.
```
let instPools =  datatable(InstancePoolID:string, PoolName:string
            , NodeType:string, DBUsage:int, DBUCost:decimal, VMCost:decimal, MinIdleInstances:int, Currency:string)
            ['0112-102712-own658-pool-9urlhql1', 'Pool0012', 'E16as v4', 4, 1.00, 1.01, 0, 'GBP',
             '0812-131731-homie204-pool-2U1OYTum', 'Pool0812', 'D13 v2', 2, 0.50, 0.36, 0, 'GBP',
             '1207-171037-steer985-pool-4hkjz8hd', 'Pool1207', 'L16s', 4, 1.00, 1.24, 0, 'GBP'];
```
Note: Some assumptions are at play here
  - Compute comes from pools
  - Pools are only used for one kind of workload - All Purpose Compute, Jobs Compute, Jobs Compute with Photon etc. This is because the DBU cost for each kind of workload is different
  - In the cost calculation we also assume that 
    - actual number of nodes is number of workers + 1, 
    - compute runs for at least 60 seconds longer than the actual job duration (this fails if you have to wait for a node to be provisioned to a pool, as it would likely be 200-400 seconds)

The Job cost calculation attempts to estimate the actual job cost, but does not include pool maintenance time. Review [pool best practices](https://learn.microsoft.com/en-us/azure/databricks/clusters/instance-pools/pool-best-practices), note however that if you set the pool idle instance auto termination to more than approx 5-10 minutes you are likely to increase actual execution cost. Test this out for yourself.

If we had the Log Analytics (Diagnostic Settings) integration turned on when we created or modified the pool, we should be able to capture pool information via this [query](/loganalytics_queries/pools.kql). The pool creation/modification time also needs be within your Log Analytics retention period.

By linking together cluster create information from the Clusters log, with Start and End information from the Jobs log, we can detect cluster duration and specification. An example query that generates a job cost can be found [here](/loganalytics_queries/job_costs.kql).

## Understanding overall compute costs

A common question is whether it is possible to save money by utilising an Azure Reservation (RI) for the VMs used in the clusters. Unless the compute is running for more than 50% of each day for a year, then this is generally not a useful option. You can estimate how many nodes and of what time are running by using the same telemetry as discussed above.

Example queries for counting the nodes and creating a chart are [here - count nodes](/loganalytics_queries/count_nodes_up_time_series.kql) and [here - last day only](/loganalytics_queries/count_nodes_time_series_lastday.kql). These are sensitive to the resolution of the time series, so ensure you do some validation to understand the detail.

Example of the output chart for  [all data](/media/Running%20Nodes.png) and [single day](/media/AllNodesDay.png)
![compute costs](/media/Running%20Nodes.png)

## Understanding SQL Endpoint Usage

It is possible to understand query activity by user, by examing DatabricksSQL logs.

Caution: SessionId has a different format from session_id
SessionId example: *eb07003524eb98e807a770b3b4c49f52f31848bd1706b1885381aaced84dbc67*

session_id example *01ee40f0-61a8-1961-bf8d-db827a1aef72*

Unfortunately, it does not appear possible to directly link executed queries with specific endpoints, but it is possible to derive general activity against specific endpoint ids.

If we need to know the cluster details for an endpoint id so that we can find out compute details from DatabricksCluster, this only exists on rows that have short form session_id. 
It would appear that query exectutions (xecuteAdhocQuery,	executeFastQuery) are sometimes accompanied by a listQueries event, so it may be possible to highlight endpoint activity by using linked session_ids.

Example Queries

[Queries executed - not linked to endpoint](/loganalytics_queries/sql_queries_executed.kql)

[Activity - linked to endpoints](/loganalytics_queries/sql_activity_by_endpoint.kql)
[To try and understand a minumum idle time](/loganalytics_queries/sql_time_between_queries_by_endpoint.kql)
![general activity by endpoint](/media/sql_endpoint_activity.png)
Activity times - attempt to measure time since last query - [to give idle time](/loganalytics_queries/sql_time_between_queries_by_endpoint.kql)

### *Areas for investigation*
- why don't the long form session ids link to anything?
- which activities actually highlight a run query (currently assuming ('executeAdhocQuery',	'executeFastQuery',	'executeSavedQuery') )
- how is query duration measured
- what does the datasource_id mean


## Understanding the various logs

Note that the details of time do not appear within each log row, rather the Time Generated is recorded. This is typically regarded as the actual time that the event occurred, rather than the Log Analytics ingest time, as recorded [here](https://docs.microsoft.com/en-us/azure/azure-monitor/logs/log-standard-columns#timegenerated).

### [DatabricksClusters](/understanding_logs.md#databricksclusters)

### [DatabricksAccounts](/understanding_logs.md#databricksaccounts)

### [DatabricksJobs](/understanding_logs.md#databricksjobs)

### [SQL Endpoint Logs](/understanding_logs.md#sql-endpoint-logs)




*[back to Contents](/README.md#contents)*

