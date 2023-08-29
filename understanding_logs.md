# Understanding the various logs

Note that the details of time do not appear within each log row, rather the Time Generated is recorded. This is typically regarded as the actual time that the event occurred, rather than the Log Analytics ingest time, as recorded [here](https://docs.microsoft.com/en-us/azure/azure-monitor/logs/log-standard-columns#timegenerated).

### **DatabricksClusters**

For the Cluster logs, there is [unique list of Actions](/loganalytics_queries/clusters_list_of_actions.kql).

*ActionName*
- start
- startResult
- create
- createResult
- resize
- resizeResult
- delete
- deleteResult


For a list of all cluster events by cluster type see [here](/loganalytics_queries/types%20of%20cluster.kql).

| Action Name | All Purpose Cluster | Jobs Cluster | SQL endpoint |
| :----------: | :----------: | :----------: | :----------: |
| create |  | X | X | 
| createResult |  | X | X | 
| delete | X | X | X | 
| deleteResult | X | X | X | 
| resizeResult | X | X | X | 
| restart | X |  |  | 
| restartResult | X |  |  | 
| start | X |  |  | 
| startResult | X |  |  | 

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

For **interactive/all purpose 
clusters**, the following events exists (from ActionName):
- start
- startResult
- resizeResult
- deleteResult 

For **interactive/all purpose,
- startResult, resizeResult, deleteResult include cluster name via
- start events do not include cluster name
- edit event shows node type for cluster (rare event so you may need to manually add this to some queries to get node info)

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

## SQL Endpoint Logs

From Databricks SQL
https://learn.microsoft.com/en-us/azure/databricks/administration-guide/account-settings/audit-logs#dbsql for high level reference

Caution: SessionId has a different format from session_id
SessionId example: *eb07003524eb98e807a770b3b4c49f52f31848bd1706b1885381aaced84dbc67*

session_id example *01ee40f0-61a8-1961-bf8d-db827a1aef72*

*Action Names*
 - cancelQueryExecution
 - createQueryDraft
 - downloadQueryResult
 - executeAdhocQuery
 - executeFastQuery
 - executeSavedQuery
 - executeWidgetQuery
 - getQueriesByLookupKeys
 - getQuery
 - listQueries
 - startEndpoint
 - updateQuery
 - updateQueryDraft

Log Details for ActionName


*with session id*
- getQueriesByLookupKeys
- listQueries
- downloadQueryResult

*with query id*
- getQueriesByLookupKeys
- listQueries
- getQuery

*with user id*
- getQueriesByLookupKeys
- listQueries

*with endpoint id*
- listQueries

Many rows have the long form SessionId.
Including 
*cancelQueryExecution,	createQuery,	createQueryDraft,	deleteQueryDraft,	downloadQueryResult,	executeAdhocQuery,	executeFastQuery,	executeSavedQuery,	executeWidgetQuery,	getQuery,	moveQueryToTrash,	requestPermissions,	updateQuery,	updateQueryDraft*
