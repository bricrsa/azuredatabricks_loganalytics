# Monitoring Azure Databricks with Log Analytics


Insights into logs

DatabricksClusters 
| distinct ActionName

ActionName
start
create
startResult
resize
deleteResult
resizeResult
delete
createResult


//find a specific job cluster
let c_id = DatabricksClusters 
| extend rparams = parse_json(RequestParams)
| extend response = parse_json(Response)
| extend StatusCode = response.statusCode
| extend ClusterId = extractjson("$.['cluster_id']", tostring(rparams))
| where isnotempty( ClusterId);
let cid = DatabricksClusters 
| extend rparams = parse_json(RequestParams)
| extend response = parse_json(Response)
| extend StatusCode = response.statusCode
| extend ClusterId = extractjson("$.['clusterId']", tostring(rparams))
| where isnotempty( ClusterId);
let c_idr = DatabricksClusters 
| extend rparams = parse_json(RequestParams)
| extend response = parse_json(Response)
| extend StatusCode = response.statusCode
| extend ClusterId = extractjson("$.['cluster_id']", tostring(response.result))
| where isnotempty( ClusterId);
c_id | union cid, c_idr
| order by TimeGenerated asc
| where ClusterId == "0721-080313-uztqenre"


//There appears to be no start event when clusters are job cluster, just 
create
delete
resize
deleteResult
createResult

//some useful info in resize  about number workers and cause for resize (rparams : num_workers and cause)
//some useful info in rparams: cluster_name in format job-{jobid}-run-{runid}

//