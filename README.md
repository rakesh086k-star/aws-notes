WVDConnections
| where TimeGenerated >= ago(30d)
| summarize LastLogin=max(TimeGenerated) by UserName
| extend DaysSinceLastLogin = datetime_diff("day", now(), LastLogin)
| project UserName, LastLogin, DaysSinceLastLogin
| order by DaysSinceLastLogin 








WVDConnections
| where TimeGenerated >= ago(90d)
| summarize LastLogin = max(TimeGenerated) by UserName
| extend DaysSinceLastLogin = datetime_diff("day", now(), LastLogin)

| extend LoginStatus = case(
    LastLogin >= startofday(now()), "🟢 Today",
    LastLogin >= startofday(ago(1d)) and LastLogin < startofday(now()), "🟡 Yesterday",
    DaysSinceLastLogin <= 7, strcat("🟠 Last ", DaysSinceLastLogin, " Days"),
    DaysSinceLastLogin <= 30, strcat("🔵 ", DaysSinceLastLogin, " Days Ago"),
    strcat("🔴 Inactive - ", DaysSinceLastLogin, " Days")
)

| extend Attention = case(
    DaysSinceLastLogin == 0, "✅ Active",
    DaysSinceLastLogin <= 7, "🟢 Normal",
    DaysSinceLastLogin <= 30, "🟡 Review",
    "🔴 Follow-up Required"
)

| extend UserActivity = case(
    DaysSinceLastLogin == 0, "🟢 Frequently Using AVD",
    DaysSinceLastLogin <= 7, "🟡 Regular User",
    DaysSinceLastLogin <= 30, "🟠 Less Active",
    "🔴 Inactive User"
)

| extend Recommendation = case(
    DaysSinceLastLogin == 0, "✅ No Action Required",
    DaysSinceLastLogin <= 7, "👍 Continue Monitoring",
    DaysSinceLastLogin <= 30, "📞 Verify if user still requires AVD access",
    "🚨 Review account and confirm if AVD access is still required"
)

| project
    UserName,
    LastLogin,
    DaysSinceLastLogin,
    LoginStatus,
    Attention,
    UserActivity,
    Recommendation

| order by LastLogin desc