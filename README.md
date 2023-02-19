# PowerBI 

### Cumulative 
Cumulative  = Calculate(sum(revenue),filter(table, table[revenue]>=Earlier(table(revenue)))

### Ranking based on Revenue
Rank = Rankx(table,table(revenue))

### Create a New table from another table
e.g. selecting certain columns from stock to create a new table name is Turnover.

Turnover = Selectcolumns(stock,
                         "SKU-ID",stock[SKU-ID],
                         "Description",stock[Description],
                         "2021_Start_stock",stock[2021_start_stock])