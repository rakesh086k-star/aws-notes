print Metric="Active Users", Count=
toscalar(
    WVDConnections
    | where TimeGenerated > ago(24h)
    | summarize dcount(UserName)
)
| union (
    print Metric="Disconnected Sessions", Count=
    toscalar(
        WVDCheckpoints
        | where TimeGenerated > ago(24h)
        | where Name contains "Disconnected"
        | summarize count()
    )
)
| union (
    print Metric="Idle Sessions", Count=
    toscalar(
        WVDAgentHealthStatus
        | where TimeGenerated > ago(24h)
        | summarize dcount(SessionHostName)
    )
)