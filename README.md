let
    WorkspaceId = "b9e2f1cd-9403-44c7-9f6f-813776f41c9a",

    KQL = "
arg("""").Resources
| where type =~ ""microsoft.compute/virtualmachines""
| where resourceGroup =~ ""RG-AVD-PH-EI-US""
| extend
    ComputerName = tolower(tostring(name)),
    PowerState = tostring(properties.extended.instanceView.powerState.code)
| extend
    VMStatus = case(
        PowerState =~ ""PowerState/running"", ""Online"",
        PowerState =~ ""PowerState/stopped"", ""Stopped"",
        PowerState =~ ""PowerState/deallocated"", ""Deallocated"",
        ""Unavailable""
    )
| project ComputerName, VMStatus
| join kind=leftouter hint.remote=right (
    WVDConnections
    | where TimeGenerated >= ago(24h)
    | extend
        ComputerName = tolower(tostring(split(SessionHostName, ""."")[0])),
        UserName = tostring(UserName)
    | summarize arg_max(TimeGenerated, State) by ComputerName, UserName
    | where State == ""Connected""
    | project ComputerName, UserName
) on ComputerName
| extend UserName = iff(
    isempty(UserName),
    ""No User"",
    UserName
)
| project
    ComputerName,
    UserName,
    VMStatus
| order by ComputerName asc, UserName asc
",

    Source =
        Json.Document(
            Web.Contents(
                "https://api.loganalytics.azure.com/v1/workspaces/" & WorkspaceId & "/query",
                [
                    Query = [
                        query = KQL
                    ],
                    Timeout = #duration(0, 0, 10, 0)
                ]
            )
        ),

    DataTable = Source[tables]{0},

    Columns = Table.FromRecords(DataTable[columns]),

    TypeMap = #table(
        {"AnalyticsTypes", "Type"},
        {
            {"string", Text.Type},
            {"int", Int32.Type},
            {"long", Int64.Type},
            {"real", Double.Type},
            {"timespan", Duration.Type},
            {"datetime", DateTimeZone.Type},
            {"bool", Logical.Type},
            {"guid", Text.Type},
            {"dynamic", Text.Type}
        }
    ),

    ColumnsWithType =
        Table.Join(
            Columns,
            {"type"},
            TypeMap,
            {"AnalyticsTypes"},
            JoinKind.LeftOuter
        ),

    Rows =
        Table.FromRecords(
            DataTable[rows],
            Columns[name]
        ),

    TableWithTypes =
        Table.TransformColumnTypes(
            Rows,
            Table.ToList(
                ColumnsWithType,
                (c) => {c{0}, c{3}}
            )
        )
in
    TableWithTypes