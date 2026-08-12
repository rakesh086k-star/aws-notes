AzureMetrics
| where TimeGenerated >= ago(7d)
| summarize Count=count() by MetricName
| order by Count desc