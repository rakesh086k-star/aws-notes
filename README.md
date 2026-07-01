print Metric="Active Users", Count=toscalar(WVDConnections | summarize dcount(UserName))
| union (
    print Metric="Disconnected Sessions", Count=toscalar(
        WVDCheckpoints
        | where Name contains "Disconnected"
        | summarize count())
)
| union (
    print Metric="Idle Sessions", Count=toscalar(
        WVDAgentHealthStatus
        | summarize dcount(SessionHostName))
)