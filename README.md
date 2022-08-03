# Monitoring Azure Databricks with Log Analytics


Azure Databricks has an in-built [monitoring capability](https://docs.microsoft.com/en-gb/azure/databricks/administration-guide/account-settings/azure-diagnostic-logs). 

For additional capabilities or for integrating with other logs, or across workspaces, then using the standard Log Analytics integration is very useful.
Add [Diagnostic settings](/media/DiagnosticSettings.png).

For further useful logging references, see [this list of useful reading](/useful_reading.md).

## *Contents*

- [Introduction - Insights into logs](#introduction---insights-into-logs)
- [Understanding the various logs](#understanding-the-various-logs)
   - DatabricksClusters [info](#databricksclusters)
  - [DatabricksAccounts](#databricksaccounts)
  - [DatabricksJobs](#databricksjobs)
- [Understanding all Cluster logs with ClusterId](#understanding-all-cluster-logs-with-clusterid)



## Introduction - Insights into logs

The following logs are the core of what is being examined here.

- DatabricksClusters
- DatabricksAccounts
- DatabricksJobs

## Understanding the various logs

[Query DatabricksCluster to get clusterId](/loganalytics_queries/find%20a%20cluster.kql)

### **DatabricksClusters**

[back to Contents](/README.md#contents)

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

For jobs clusters, the provisioned resources or pool can be found in the create event
- in RequestParams
    use node_type_id and num_workers for SQL Analytics
    for pooled users = use instance_pool_id and driver_instance_pool_id and num_workers
- there is also a resize event that occurs
    with a target_workers inside autoscale
    or just num_workers
    both have a cause
- the instance_pool_id is part of the cluster create event
- investigate cause of AUTORECOVERY events in [resize](/loganalytics_queries/cluster_resize_autorecovery.kql)

For interactive clusters, the following events exists (from ActionName):
- start
- startResult
- resizeResult
- deleteResult 

For interactive clusters,
- startResult, resizeResult, deleteResult include cluster name via
- start events do not include cluster name

[back to Contents](/README.md#contents)

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

[back to Contents](/README.md#contents)

## Understanding Cluster costs from Cluster information

Using the create event, we can discover the pool id and number of workers for jobs clusters.

We can also create a definition for the costs of each pool, using information from the Azure Pricing Calculator.
```
let instPools =  datatable(InstancePoolID:string, PoolName:string
            , NodeType:string, DBUsage:int, DBUCost:decimal, VMCost:decimal, MinIdleInstances:int, Currency:string)
            ['0112-102712-own658-pool-9urlhql1', 'Pool0012', 'E16as v4', 4, 1.00, 1.01, 0, 'GBP',
             '0812-131731-homie204-pool-2U1OYTum', 'Pool0812', 'D13 v2', 2, 0.50, 0.36, 0, 'GBP',
             '1207-171037-steer985-pool-4hkjz8hd', 'Pool1207', 'L16s', 4, 1.00, 1.24, 0, 'GBP'];
```

