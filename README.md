let SelectedUser = "";   
// सभी users के लिए blank रखें
// Specific user के लिए:
// let SelectedUser = "Anuradha218532@exlservice.com";

WVDConnectionNetworkData
| where TimeGenerated >= ago(7d)
| extend TimeSlot = bin(TimeGenerated, 30m)
| join kind=inner (
    WVDConnections
    | where TimeGenerated >= ago(7d)
    | where State == "Connected"
    | where UserName != ""
    | distinct CorrelationId, UserName
) on CorrelationId
| where isempty(SelectedUser) or UserName =~ SelectedUser
| summarize
    Avg_RTT = round(avg(EstRoundTripTimeInMs), 2),
    Max_RTT = round(max(EstRoundTripTimeInMs), 2),
    Avg_Bandwidth = round(avg(EstAvailableBandwidthKBps), 2),
    Max_Bandwidth = round(max(EstAvailableBandwidthKBps), 2)
    by UserName, TimeSlot
| order by TimeSlot asc