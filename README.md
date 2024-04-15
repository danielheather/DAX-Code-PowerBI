# The use of DAX 

In conjunction with the PowerBI code, we utilized DAX code to create new measures and columns. This allowed us to better sculpt the visualisations.

The Power BI dashboard/reports can be viewed here https://github.com/danielheather/Power-BI-Sales-Dashboard

The DAX code is available as txt as notepad++ is not DAX compatible

## DAX Code Design

Examples of the Data Analysis Expression code is displayed below, grouped by its functionality. 

Average Calculations are used for ease of comparison between aspects of the data.
```
Avg_FC_Cost.1 = 
CALCULATE(
    AVERAGE(Orders[Shipping Cost]),
    FILTER(
        Orders,
        Orders[Ship Mode]= "First Class"))
```

The calculation of the average return rate across shipping rates is done to provide a measure to compare the categories against.
```
Average Return Rate = AVERAGEX(
    ALL('Orders'[Ship Mode]), 
    [Return Rate by Ship Mode]
)
```

Categorization of customer by their relative spend allows us gain insight into the different behavior based on amount spent.
```
Customer Segmented by Rank = 
VAR CustomerRank = [Sales Rank by Customer]
VAR TotalCustomers = DISTINCTCOUNT('Orders'[Customer ID])
VAR Percentile20 = TotalCustomers * 0.2
VAR Percentile50 = TotalCustomers * 0.5
RETURN
IF(
    CustomerRank <= Percentile20, "Big Spenders",
    IF(
        CustomerRank > Percentile20 && CustomerRank <= Percentile50, "Moderate Spenders",
        "Occasional Purchasers"
    )
)
```
Conditional Logic,
* Ship cost by mode used to determine cost by shipping mode for greater depth of insight.
* Profit Percentage is used to better compare the profit rate between categories, ship modes. Better guides pricing strategies and cost management.
```
First_Class = IF(Orders[Ship Mode] = "First Class", Orders[Shipping Cost])

Profit Percentage = SUM(Orders[Profit]) / SUM(Orders[Sales]) 
```
Calculates the return rate for each shipping mode while maintaining the context of that shipping mode.
```
Return Rate by Ship Mode = 
CALCULATE(
    [Return Rate],
    ALLEXCEPT('Orders', 'Orders'[Ship Mode])
)
```
Ranking and Status
* Sales Rank by Customer,  Assigns a rank to each customer based on total sales, using a dense ranking method.
* Shipment Status: Evaluates, if shipments are 'Late' or 'On Time' based on predefined criteria for each shipping mode.
```
Sales Rank by Customer = RANKX(
    ALL('Orders'[Customer ID]), 
    [Total Sales by Customer], 
    , 
    DESC, 
    DENSE
)

Shipment Status = 
SWITCH(TRUE(),
    Orders[Ship Mode] = "Same Day" && Orders[Shipping Time] > 1, "Late",
    Orders[Ship Mode] = "First Class" && Orders[Shipping Time] > 2, "Late",
    Orders[Ship Mode] = "Second Class" && Orders[Shipping Time] > 4, "Late",
    Orders[Ship Mode] = "Standard Class" && Orders[Shipping Time] > 6, "Late", 
    "On Time"
    )
```
Time Calculation, calculates the number of days between order and shipment date.
```
Shipping Time = DATEDIFF(Orders[Order Date], Orders[Ship Date], DAY)
```
Total Sales Computation, Calculates total sales for each customer, maintaining the context of customer ID across the Orders table.
```
Total Sales by Customer = CALCULATE(SUM('Orders'[Sales]), ALLEXCEPT('Orders', 'Orders'[Customer ID]))
```
Utility expressions, to isolate the shipping cost of one mode for comparison.
```
Standard_Class = IF(Orders[Ship Mode] = "Standard Class", Orders[Shipping Cost])
```
