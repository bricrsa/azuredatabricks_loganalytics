# Monitoring Azure Databricks with Log Analytics


Azure Databricks has an in-built [monitoring capability](https://docs.microsoft.com/en-gb/azure/databricks/administration-guide/account-settings/azure-diagnostic-logs). 

For additional capabilities or for integrating with other logs, or across workspaces, then using the standard Log Analytics integration is very useful.
Add [Diagnostic settings](/media/DiagnosticSettings.png).

For further useful logging references, see [this list of useful reading](/useful_reading.md).

## *Contents*





## Insights into logs

The following logs are the core of what is being examined here.

DatabricksClusters
DatabricksJobs

# DatabricksClusters
For the Cluster logs, there is unique list of Actions

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
create, createResult, deleteResult and resizeResult include the cluster name via Response.????
delete events do not include cluster name

For jobs clusters, the provisioned resources or pool can be found in the create event
in RequestParams
    use node_type_id and num_workers for SQL Analytics
    for pooled users = use instance_pool_id and driver_instance_pool_id and num_workers
there is also a resize event that occurs
    with a target_workers inside autoscale
    or just num_workers
    both have a cause

the instance_pool_id is part of the cluster create event

--investigate cause of AUTORECOVERY events in resize



For interactive clusters, the following events exists (from ActionName):
start
startResult
resizeResult
deleteResult 

For interactive clusters,
startResult, resizeResult, deleteResult include cluster name via
start events do not include cluster name

From Databricks accounts, we can see the UserAgent that logged in 

//some useful info in resize  about number workers and cause for resize (rparams : num_workers and cause)
//some useful info in rparams: cluster_name in format job-{jobid}-run-{runid}

//

From DatabricksJobs

for event runStart
    from RequestParams
 we can get jobCluster, clusterId
 clusterId
idInJob
jobClusterType
jobId
jobTaskType
jobTerminalState
jobTriggerType
multitaskParentRunId
orgId
runId
taskDependencies
[]
taskKey


All events

with ClusterId
runStart    RunId = rparams.runId, JobId = rparams.jobId, ClusterID = rparams.clusterId
runSucceeded same as runStart
runFailed same as runStart

with RunID
getRunOutput    rparams.run_id
submitRun   response.result.run_id
runTriggered rparams.runId, rparams.jobId
cancel  rparams.run_id
runNow rparams.job_id, response.result.run_id
deleteRun rparams.run_id

with just JobId
update  rparams.job_id

with Nothing
changeJobAcl