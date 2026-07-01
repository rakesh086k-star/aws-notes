search *
| summarize Count = count() by $table
| order by Count desc



union withsource=TableName *
| summarize Count=count() by TableName
| order by Count desc