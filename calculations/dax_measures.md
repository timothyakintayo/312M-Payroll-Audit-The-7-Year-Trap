# Key DAX Measures
This document outlines the core DAX logic used in the **Total Rewards Equity Analysis** and dashboard. These measures handle the calendar (date), calculated columns for compa-ratio, tenure in years, gender pay gap, and total annual payroll expenses of the company, and security filtering.

## 1. Calendar (Date)
Created dynamically to ensure continuous dates for Time Intelligence functions.
#### Calendar Table
```dax
Dim_Calendar = CALENDAR(MIN(Fact_Tickets[Ticket_Date]), MAX(Fact_Tickets[Ticket_Date]))
```
#### Month
```dax
Month = FORMAT('Dim_Calendar'[Date], "mmm")
```
#### Month Number
```dax
Month Number = MONTH('Dim_Calendar'[Date])
```
#### Quarter

```dax
Quarter = FORMAT('Dim_Calendar'[Date], "\QQ")
```
#### Year
```dax
Year = FORMAT('Dim_Calendar'[Date], "yyyy")
```

## 2. Calculated Column: Compa-Ratio Grouping
A metric used to group employees based on their annual salary relative to the market midpoint.
```dax
Compa Ratio Group = 
VAR CR = DIVIDE('GE_TotalRewards_Mock_Data'[Annual_Salary], 'GE_TotalRewards_Mock_Data'[Market_Midpoint])
RETURN
SWITCH(TRUE(),
    CR < 0.80, "1. Underpaid (<80%)",
    CR >= 0.80 && CR < 0.90, "2. Low (80-90%)",
    CR >= 0.90 && CR < 1.10, "3. Market (90-110%)",
    CR >= 1.10 && CR < 1.20, "4. High (110-120%)",
    CR >= 1.20, "5. Overpaid (>120%)")
```

## 3. Calculated Column: Tenure in Years

To calculate the years spent by employees on average in the company, overall, the average tenure year was calculated using below formula for each rows by creating a calculated column.
```dax
Tenure (Years) = DIVIDE(DATEDIFF('GE_TotalRewards_Mock_Data'[Hire_Date], TODAY(), DAY), 365.25)
```

## 4. Headcount
This measure calculates the total number of employees in the organization
```dax
Headcount = COUNT(GE_TotalRewards_Mock_Data[EmployeeID])
```

## 5. Gender Pay Gap %
This measure calculates the gender pay gap unadjusted in the company to determine the unadjusted pay back in the organization in line with the incoming 5% compliance threshold, and the EU average of 13%.
```dax
Gender Pay Gap % = 
VAR Avg_Male_Pay = CALCULATE(AVERAGE('GE_TotalRewards_Mock_Data'[Annual_Salary]), 'GE_TotalRewards_Mock_Data'[Gender] = "Male")
VAR Avg_Female_Pay = CALCULATE(AVERAGE('GE_TotalRewards_Mock_Data'[Annual_Salary]), 'GE_TotalRewards_Mock_Data'[Gender] = "Female")
RETURN 
DIVIDE(Avg_Male_Pay - Avg_Female_Pay, Avg_Male_Pay)
```

## 6. Total Annual Payroll
Calculates the total annual amount spent on employee salary.
```dax
Total Annual Payroll = SUM(GE_TotalRewards_Mock_Data[Annual_Salary])
```

## 7. Average Compa-Ratio
This measure calculates the average compa-ratio and compares the value obtained to the market midpoint to see if the company is paying it's employees below or above the market.
```Avg Compa-Ratio = 
DIVIDE(
AVERAGE(GE_TotalRewards_Mock_Data[Annual_Salary]),
AVERAGE(GE_TotalRewards_Mock_Data[Market_Midpoint])
)
```

## 7. Average Performance Rating
This measure calculates the average performance of employees and benchmarked it against a target value. Similarly, it examined individual employees whose performance is below/above this calculated average to determine top performers, and least performers.
AVG Performance Rating = AVERAGE('GE_TotalRewards_Mock_Data'[Performance_Rating])
```

```
## 10. Row-Level Security (RLS) Filter
A specific role was created for the IT manager to be able to see the overall performance of his IT team only, enabling filtering for each managers to only be able to access reports that are germane to their team.
```text
// Filter expression applied to Dim_Technicians table
[Department] = "IT"
```