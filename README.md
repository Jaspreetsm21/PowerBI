# PowerBI - Guide

## Dataflows

## Datasets

## Gateways

## Workspaces and App

## Functions

# Dates

### N Days
e.g. 7 Days of Sales
7 Days = 
  var Latest = max('Date Dimension'[Date])
  var startdate = latest-7 
Return
  CALCULATE([Customers], FILTER(ALL('Date Dimension'[Date]), 'Date Dimension'[Date] <= Latest &&  'Date Dimension'[Date] > startdate))
  
 Last 7 days of Sales
 
 7_sales = CALCULATE( SUM('table'[sales]),'table'[date_of_sales]>=TODAY()-14)
  

### Top N months

Allows you to filter out the last n months 

Top N Months = DATEDIFF('Date Dimension'[Date],TODAY(),MONTH)

## Rolling Avearge
RollingAvg = 

  Var NumDays = 14

  Var rollingsum_goal =   CALCULATE(SUM('table'[goals]), DATESINPERIOD('Dim - Date'[Date],LASTDATE('Dim - Date'[Date]),-NumDays,DAY))
  
  Var rollingsum_chances = CALCULATE(SUM('table'[chances]), DATESINPERIOD('Dim - Date'[Date],LASTDATE('Dim - Date'[Date]),-NumDays,DAY))

Return 
  
  DIVIDE(rollingsum_goal,rollingsum_chances)


### Current_months

## Previous Months 

Previous Months = CALCULATE([Sales],PREVIOUSMONTH('Dim - Date'[Date]))

### UserRelationshiop
Use this function when two tables currently dont have a relationship

Requests = calculate(DISTINCTCOUNT('Table[ID]),USERELATIONSHIP('Table'[Requested Date],'Dim - Date'[Date]))

Dax Calculate formuale = CALCULATE(SUM('Table'[Total]),FILTER('Table','Table'[ORDER_STATUS]="Live"))



### Row Total

row_total = DIVIDE([total],CALCULATE([total],ALL('Table'[Category])))

### Cumulative 
Cumulative  = Calculate(sum(revenue),filter(table, table[revenue]>=Earlier(table(revenue)))

### Ranking based on Revenue
Rank = Rankx(table,table(revenue))

![image](https://user-images.githubusercontent.com/60583082/219952480-f317b96e-0fe4-41b6-bff2-498d532de4fc.png)


### Create a New table from another table

### Switch 

type_Group = 
VAR MySelection =
SELECTEDVALUE ('Fees Flag'[Type] )
RETURN
SWITCH (
TRUE (),
MySelection = "Counts Non Discount", [Non_Disc_Cust],
MySelection = "Counts Discount", [Disc_Cust],
MySelection = "Avg Upfront Non Discount", [Avg_Annual_fees],
MySelection = "Avg Upfront Discount", [Avg_Disc_fees],
MySelection = "Avg Monthly Non Discount", [non_disc_avg_monthly],
MySelection = "Avg Monthly Discount", [avg_disc_monthly_fees],
0
)



## Organsation Level

# Path Length (number of People above them)

Path Length = PATHLENGTH('fact(Employees) Higher'[path])
path = PATH('fact(Employees) Higher'[BW ID],'fact(Employees) Higher'[Manger_BW])

Organization Level 1 = 
VAR EmployeeID = PATHITEM('fact(Employees) Higher'[path], 1 #(level 1,2,3,4), 1)
VAR EmployeeName = LOOKUPVALUE(
    'fact(Employees) Higher'[Employee_name],
    'fact(Employees) Higher'[BW ID], EmployeeID
)
VAR EmployeePosition = CALCULATE(
    FIRSTNONBLANK('fact(Employees) Higher'[Employee_Position], 1),
    FILTER(
        'fact(Employees) Higher',
        'fact(Employees) Higher'[Employee_name] = EmployeeName
    )
)
RETURN
IF(NOT(ISBLANK(EmployeeName)),IF(
    ISBLANK(EmployeePosition),
    EmployeeName,
    EmployeeName & " : " & EmployeePosition
),BLANK())

e.g. selecting certain columns from stock to create a new table name is Turnover.

Turnover = Selectcolumns(stock,
                         "SKU-ID",stock[SKU-ID],
                         "Description",stock[Description],
                         "2021_Start_stock",stock[2021_start_stock])


Organization Level 4 = 
Var EmployeeName = LOOKUPVALUE(
    'fact(Employees) Higher'[Employee_name],
    'fact(Employees) Higher'[BW ID],
                    PATHITEM(
                            'fact(Employees) Higher'[path],
                            4,
                            1))
RETURN
IF(NOT(ISBLANK(EmployeeName)), EmployeeName, BLANK())

# API in Power BI query

= (DT as text) as table=>
let
    Source = Json.Document(Web.Contents("URL="&DT&")),
    #"Converted to Table" = Table.FromRecords({Source}),
    #"Expanded result" = Table.ExpandListColumn(#"Converted to Table", "result"),
    #"Expanded result1" = Table.ExpandRecordColumn(#"Expanded result", "result", {"u_category", "number", "opened_at", "hr_service", "assignment_group", "u_subcategory", "opened_by", "work_end", "hr_profile.user.u_global_hr_id", "state", "topic_category", "approval_history"}, {"result.u_category", "result.number", "result.opened_at", "result.hr_service", "result.assignment_group", "result.u_subcategory", "result.opened_by", "result.work_end", "result.hr_profile.user.u_global_hr_id", "result.state", "result.topic_category", "result.approval_history"}),
    #"Changed Type" = Table.TransformColumnTypes(#"Expanded result1",{{"result.u_category", type text}, {"result.number", type text}, {"result.opened_at", type datetime}, {"result.hr_service", type text}, {"result.assignment_group", type text}, {"result.u_subcategory", type text}, {"result.opened_by", type text}, {"result.work_end", type datetime}, {"result.hr_profile.user.u_global_hr_id", Int64.Type}, {"result.state", type text}, {"result.topic_category", type text}}),
    #"Extracted Date" = Table.TransformColumns(#"Changed Type",{{"result.opened_at", DateTime.Date, type date}, {"result.work_end", DateTime.Date, type date}}),
    #"Sorted Rows" = Table.Sort(#"Extracted Date",{{"result.opened_at", Order.Descending}})
in
    #"Sorted Rows"


## Second table in query
let
    Source = Table.FromRows(Json.Document(Binary.Decompress(Binary.FromText("i45WMjIwMtY1MAQipVgdMNcElWsK48YCAA==", BinaryEncoding.Base64), Compression.Deflate)), let _t = ((type nullable text) meta [Serialized.Text = true]) in type table [#"API Date" = _t]),
    #"Added Custom" = Table.AddColumn(Source, "Custom", each #"ServiceNow Function"([API Date])),
    #"Expanded Custom" = Table.ExpandTableColumn(#"Added Custom", "Custom", {"result.u_category", "result.number", "result.opened_at", "result.hr_service", "result.assignment_group", "result.u_subcategory", "result.opened_by", "result.work_end", "result.hr_profile.user.u_global_hr_id", "result.state", "result.topic_category", "result.approval_history"}, {"Custom.result.u_category", "Custom.result.number", "Custom.result.opened_at", "Custom.result.hr_service", "Custom.result.assignment_group", "Custom.result.u_subcategory", "Custom.result.opened_by", "Custom.result.work_end", "Custom.result.hr_profile.user.u_global_hr_id", "Custom.result.state", "Custom.result.topic_category", "Custom.result.approval_history"}),
    #"Removed Columns" = Table.RemoveColumns(#"Expanded Custom",{"API Date"}),
    #"Grouped Rows" = Table.Group(#"Removed Columns", {"Custom.result.number", "Custom.result.u_category", "Custom.result.opened_at", "Custom.result.hr_service", "Custom.result.assignment_group", "Custom.result.u_subcategory", "Custom.result.opened_by", "Custom.result.work_end", "Custom.result.hr_profile.user.u_global_hr_id", "Custom.result.state", "Custom.result.topic_category", "Custom.result.approval_history"}, {{"Count", each Table.RowCount(_), Int64.Type}}),
    #"Renamed Columns" = Table.RenameColumns(#"Grouped Rows",{{"Custom.result.number", "result.number"}, {"Custom.result.u_category", "result.u_category"}, {"Custom.result.opened_at", "result.opened_at"}, {"Custom.result.hr_service", "result.hr_service"}, {"Custom.result.assignment_group", "result.assignment_group"}, {"Custom.result.u_subcategory", "result.u_subcategory"}, {"Custom.result.opened_by", "result.opened_by"}, {"Custom.result.work_end", "result.work_end"}, {"Custom.result.hr_profile.user.u_global_hr_id", "result.hr_profile.user.u_global_hr_id"}, {"Custom.result.state", "result.state"}, {"Custom.result.topic_category", "result.topic_category"}, {"Custom.result.approval_history", "result.approval_history"}}),
    #"Changed Type" = Table.TransformColumnTypes(#"Renamed Columns",{{"result.opened_at", type date}, {"result.work_end", type date}})
in
    #"Changed Type"
