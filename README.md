# Monitoring Azure Databricks with Log Analytics


Azure Databricks has an in-built [monitoring capability](https://docs.microsoft.com/en-gb/azure/databricks/administration-guide/account-settings/azure-diagnostic-logs). 

For additional capabilities or for integrating with other logs, or across workspaces, then using the standard Log Analytics integration is very useful.
Add [Diagnostic settings](/media/DiagnosticSettings.png).

For further useful logging references, see [this list of useful reading](/useful_reading.md).

## *Contents*

- [Introduction - Insights into logs](#introduction---insights-into-logs)
- [Specific scenario queries](#specific-queries-to-hightlight-key-information)
- [Understanding Cluster costs](#understanding-cluster-costs-from-cluster-information)
- [Understanding the various logs](#understanding-the-various-logs)
  - [DatabricksClusters](#databricksclusters)
  - [DatabricksAccounts](#databricksaccounts)
  - [DatabricksJobs](#databricksjobs)



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

You can use the Azure Pricing Calculator to get a rough guide on the cost of a particular cluster per hour. There are some variations that occur, such has how many disks get added to each node, but we will not consider that here.

Using the Cluster logs and the create event, we can discover the pool id and number of workers for jobs clusters.

We can also create a definition for the costs of each pool, using information from the Azure Pricing Calculator.
```
let instPools =  datatable(InstancePoolID:string, PoolName:string
            , NodeType:string, DBUsage:int, DBUCost:decimal, VMCost:decimal, MinIdleInstances:int, Currency:string)
            ['0112-102712-own658-pool-9urlhql1', 'Pool0012', 'E16as v4', 4, 1.00, 1.01, 0, 'GBP',
             '0812-131731-homie204-pool-2U1OYTum', 'Pool0812', 'D13 v2', 2, 0.50, 0.36, 0, 'GBP',
             '1207-171037-steer985-pool-4hkjz8hd', 'Pool1207', 'L16s', 4, 1.00, 1.24, 0, 'GBP'];
```

If we had the Log Analytics (Diagnostic Settings) integration turned on when we created or modified the pool, we should be able to capture pool information via this [query](/loganalytics_queries/pools.kql).

By linking together cluster create information from the Clusters log, with Start and End information from the Jobs log, we can detect cluster duration and specification. An example query that generates a job cost can be found [here](/loganalytics_queries/job_costs.kql).


## Understanding the various logs

Note that the details of time do not appear within each log row, rather the Time Generated is recorded. This is typically regarded as the actual time that the event occurred, rather than the Log Analytics injest time, as recorded [here](https://docs.microsoft.com/en-us/azure/azure-monitor/logs/log-standard-columns#timegenerated).

### **DatabricksClusters**

For the Cluster logs, there is [unique list of Actions](/loganalytics_queries/clusters_list_of_actions.kql)

*ActionName*
- start
- create
- startResult
- resize
- deleteResult
- resizeResult
- delete
- createResult

Custom Tags are available from RequestParams.custom_tags as list of key-value pairs.

For job clusters the following events exists (from ActionName):
+ create
+ delete
+ resize
+ deleteResult
+ createResult

For job clusters:
- *create* includes the cluster name via RequestParams.cluster_name
- *createResult*, *deleteResult* and *resizeResult* includes the cluster name via RequestParams.clusterName
- *delete* events do not include cluster name

For **jobs clusters**, the provisioned resources or pool can be found in the create event
- in RequestParams
    use node_type_id and num_workers for SQL Analytics
    for pooled users = use instance_pool_id and driver_instance_pool_id and num_workers
- there is also a resize event that occurs
    with a target_workers inside autoscale
    or just num_workers
    both have a cause
- the instance_pool_id is part of the cluster create event
- investigate cause of AUTORECOVERY events in [resize](/loganalytics_queries/cluster_resize_autorecovery.kql)

For **interactive clusters**, the following events exists (from ActionName):
- start
- startResult
- resizeResult
- deleteResult 

For interactive clusters,
- startResult, resizeResult, deleteResult include cluster name via
- start events do not include cluster name

### **DatabricksAccounts**

From Databricks accounts, we can see the UserAgent that logged in 

### **DatabricksJobs**

From DatabricksJobs

for event *runStart*
-    from RequestParams we can get 
```
    - jobCluster, clusterId, idInJob, jobClusterType, jobId
    - jobTaskType, jobTerminalState, jobTriggerType, multitaskParentRunId
    - orgId, runId, taskDependencies, taskKey
```

Event details:

*with ClusterId*
- runStart    RunId = rparams.runId, JobId = rparams.jobId, ClusterID = rparams.clusterId
- runSucceeded same as runStart
- runFailed same as runStart

*with RunID*
- getRunOutput    rparams.run_id
- submitRun   response.result.run_id
- runTriggered rparams.runId, rparams.jobId
- cancel  rparams.run_id
- runNow rparams.job_id, response.result.run_id
- deleteRun rparams.run_id

*with just JobId*
- update  rparams.job_id

*with Nothing*
- changeJobAcl



*[back to Contents](/README.md#contents)*

