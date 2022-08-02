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

[Query DatabricksCluster to get clusterId](/loganalytics_queries/find%20a%20cluster.kql)

# Understanding Cluster costs from Cluster information

Using the create event, we can discover the pool id and number of workers for jobs clusters.

We can also create a definition for the costs of each pool, using information from the Azure Pricing Calculator.
'''
let instPools =  datatable(InstancePoolID:string, PoolName:string
            , NodeType:string, DBUsage:int, DBUCost:decimal, VMCost:decimal, MinIdleInstances:int, Currency:string)
            ['0112-102712-own658-pool-9urlhql1', 'Pool0012', 'E16as v4', 4, 1.00, 1.01, 0, 'GBP',
             '0812-131731-homie204-pool-2U1OYTum', 'Pool0812', 'D13 v2', 2, 0.50, 0.36, 0, 'GBP',
             '1207-171037-steer985-pool-4hkjz8hd', 'Pool1207', 'L16s', 4, 1.00, 1.24, 0, 'GBP'];
'''

