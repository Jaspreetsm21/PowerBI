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
