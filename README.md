let LastLogin =
WVDConnections
| summarize LastLogin = max(TimeGenerated) by UserName;

LastLogin
| extend DaysSinceLastLogin = datetime_diff("day", LastLogin, now())

| extend LoginStatus = case(
    DaysSinceLastLogin == 0, "🟢 Today",
    DaysSinceLastLogin == 1, "🟡 Yesterday",
    DaysSinceLastLogin == 2, "🟠 2 Days Ago",
    DaysSinceLastLogin == 3, "🟠 3 Days Ago",
    DaysSinceLastLogin == 4, "🟠 4 Days Ago",
    DaysSinceLastLogin == 5, "🟠 5 Days Ago",
    DaysSinceLastLogin == 6, "🟠 6 Days Ago",
    DaysSinceLastLogin == 7, "🟠 7 Days Ago",
    DaysSinceLastLogin <= 14, strcat("🟣 ", tostring(DaysSinceLastLogin), " Days Ago"),
    DaysSinceLastLogin <= 30, strcat("🔴 ", tostring(DaysSinceLastLogin), " Days Ago"),
    strcat("⚫ ", tostring(DaysSinceLastLogin), " Days Ago")
)

| extend UserActivity = case(
    DaysSinceLastLogin == 0, "✅ Active Today",
    DaysSinceLastLogin == 1, "👍 Daily User",
    DaysSinceLastLogin >= 2 and DaysSinceLastLogin <= 3, "🚀 Frequently Using AVD",
    DaysSinceLastLogin >= 4 and DaysSinceLastLogin <= 7, "📅 Regular User",
    DaysSinceLastLogin >= 8 and DaysSinceLastLogin <= 14, "⚠️ Less Frequent User",
    DaysSinceLastLogin >= 15 and DaysSinceLastLogin <= 30, "⏳ Rarely Using AVD",
    "❌ Inactive User"
)

| project
    UserName,
    LastLogin,
    DaysSinceLastLogin,
    LoginStatus,
    UserActivity

| order by LastLogin desc