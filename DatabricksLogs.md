# DatabricksClusters

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

# DatabricksAccounts

From Databricks accounts, we can see the UserAgent that logged in 

# DatabricksJobs

From DatabricksJobs

for event runStart
    from RequestParams we can get 
- jobCluster, clusterId, idInJob, jobClusterType, jobId
- jobTaskType, jobTerminalState, jobTriggerType, multitaskParentRunId
- orgId, runId, taskDependencies, taskKey


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