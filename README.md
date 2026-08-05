search *
| summarize Count=count() by $table
| order by Count desc

AzureDiagnostics
| where ResourceProvider == "MICROSOFT.RECOVERYSERVICES"
| project TimeGenerated, Category, OperationName, Resource, ResultType, ResultDescription
| order by TimeGenerated desc



AzureDiagnostics
| where Category contains "Replication"
| project TimeGenerated,
          Resource,
          Category,
          OperationName,
          ResultType,
          ResultDescription
| order by TimeGenerated desc



