# Monitoring Azure Databricks with Log Analytics


Azure Databricks has an in-built [monitoring capability](https://docs.microsoft.com/en-gb/azure/databricks/administration-guide/account-settings/azure-diagnostic-logs). 

For additional capabilities or for integrating with other logs, or across workspaces, then using the standard Log Analytics integration is very useful.
Add [Diagnostic settings](/media/DiagnosticSettings.png).

For further useful logging references, see [this list of useful reading](/useful_reading.md).

## *Contents*

- [Introduction - Insights into logs](/DatabricksLogs.md/#introduction---insights-into-logs)
- [DatabricksClusters](/DatabricksLogs.md#databricksclusters)
- [DatabricksAccounts](/DatabricksLogs.md#databricksaccounts)
- [DatabricksJobs](/DatabricksLogs.md#databricksjobs)
- [Understanding all Cluster logs with ClusterId](#understanding-all-cluster-logs-with-clusterid)



## Introduction - Insights into logs

The following logs are the core of what is being examined here.

DatabricksClusters
DatabricksAccounts
DatabricksJobs

# Understanding all Cluster logs with ClusterId