# Data Project
## Superstore Employee Performance Analysis

This project explores employee performance within a retail superstore to uncover key trends and metrics that directly impact sales and operational efficiency. By analyzing employee-related data, I identified trends such as peak sales periods by employee, sales contribution across departments, and customer satisfaction ratings tied to individual performance. Key metrics evaluated include employee sales revenue, task completion rates, and average handling time.

To address identified sales challenges, such as uneven workload distribution and low productivity during specific shifts, I provided actionable insights by:

Recommending optimized shift schedules to balance workload and enhance productivity.
Identifying top-performing employees to inform training and mentoring programs.
Highlighting sales patterns tied to employee performance for better-targeted sales strategies.
## Table of Contents  

- [Project Overview](#project-overview)  
- [Dataset Description](#dataset-description)  
- [Data Cleaning](#data-cleaning)  
  - [Removing Duplicates](#removing-duplicates)  
  - [Handling Missing Values](#handling-missing-values)  
  - [Removing Unnecessary Columns](#removing-unnecessary-columns)  
- [Analysis Conducted](#analysis-conducted)  
  - [Employee Demographics Analysis](#employee-demographics-analysis)  
  - [Performance Analysis](#performance-analysis)  
  - [Employee Growth and Promotions Analysis](#employee-growth-and-promotions-analysis)  
  - [Diversity and Inclusion Analysis](#diversity-and-inclusion-analysis)  
  - [Promotion Effectiveness Analysis](#promotion-effectiveness-analysis)
- [Advanced Analysis](#advanced-analysis)
  - [Key Performance Indicators](#key-performance-indicators)
  - [Areas Needing Improvement](#areas-needing-improvement)
  - [Diversity and Inclusion Analysis](#diversity-and-inclusion-analysis)
  - [Promotion Effectiveness Analysis](#promotion-effectiveness-analysis)
- [Key Findings and Recommendations](#key-findings-and-recommendations)
- [Insights and Visualizations](#insights-and-visualizations)  
  - [Employee Insights Dashboard](#employee-insights-dashboard)  
  - [Performance Trends Visualizations](#performance-trends-visualizations)  
- [Conclusion](#conclusion)  
- [How to Run the Project](#how-to-run-the-project)  

## Project Overview
This project evaluates employee performance in a superstore environment using SQL queries to process and analyze data. The insights generated aim to optimize workforce efficiency, reduce attrition, and promote diversity and inclusivity.

## Dataset Description
The data is housed within the hr_info table. The table contains columns such as employee demographics, performance ratings, years at the company, promotions, and attrition status.
You can get the dataset from the assets/docs folder of this very repository or Kaggle 

## Data Cleaning
Data cleaning ensures the accuracy, reliability, and usability of the dataset for meaningful analysis.

### Removing Duplicates
Duplicates in Employee_no were removed to maintain unique employee records.
```sql
DELETE FROM hr_database.hr_info
WHERE (`Employee_no`) IN (
    SELECT * FROM (
        SELECT `Employee_no`
        FROM hr_database.hr_info
        GROUP BY `Employee_no`
        HAVING COUNT(*) > 1
    ) AS duplicates
);
```
### Handling Missing Values
Count of missing Employee_no:
```sql
SELECT Employee_no, COUNT(*) - COUNT(Employee_no) AS null_count
FROM hr_database.hr_info
GROUP BY Employee_no
HAVING null_count > 0;
```
### Removal of rows with missing Employee_no:
```sql
DELETE FROM hr_database.hr_info
WHERE Employee_no IS NULL;
```
### Removing Unnecessary Columns
Columns unrelated to the analysis were dropped to simplify the dataset:
```sql
ALTER TABLE hr_database.hr_info
DROP COLUMN Over_18,
DROP COLUMN Employee_count;
```
## Analysis Conducted
### Employee Demographics Analysis
Count of Employees by Department, Gender, and Age Group
```sql
SELECT 
    Department,
    Gender,
    CF_age_band,
    COUNT(*) AS Employee_count
FROM hr_database.hr_info
GROUP BY Department, Gender, CF_age_band
ORDER BY Department, Gender, CF_age_band;
```

### Average Tenure and Age of Employees
```sql
SELECT 
    Department,
    AVG(Years_at_company) AS avg_tenure,
    AVG(Age) AS avg_age
FROM hr_database.hr_info
GROUP BY Department
ORDER BY avg_tenure DESC;
```
### Performance Analysis
Average Performance Rating by Department
```sql
SELECT 
    Department,
    AVG(Perfomance_rating) AS avg_performance_rating
FROM hr_database.hr_info
GROUP BY Department
ORDER BY avg_performance_rating DESC;
```
Top-Performing Employees
```sql
SELECT 
    Employee_no,
    Department,
    Job_Role,
    Perfomance_rating,
    Years_at_company
FROM hr_database.hr_info
WHERE Perfomance_rating >= 4 AND Years_at_company >= 15 
ORDER BY Years_at_company DESC;
```
### Employee Growth and Promotions Analysis
Number of Employees Promoted in the Last Year by Department
```sql
SELECT 
    Department,
    COUNT(*) AS promotions_last_year
FROM hr_database.hr_info
WHERE Years_since_promotion = 0
GROUP BY Department
ORDER BY promotions_last_year DESC;
```
## Advanced Analysis 
### Key Performance Indicators
Work-Life Balance vs. Performance Rating:
```sql
-- Analyzing the relationship between work-life balance and performance rating
SELECT 
    Relationship_satisfaction,
    AVG(Perfomance_rating) AS avg_performance_rating,
    COUNT(*) AS employee_count
FROM hr_database.hr_info
GROUP BY Relationship_satisfaction
ORDER BY Relationship_satisfaction;
```
Top Employees: This query will identify top-performing employees based on a combination of performance rating and years at the company.
```sql
WITH top_employees_cte AS (
    SELECT 
        Employee_no,
        Department,
        Job_Role,
        Perfomance_rating,
        Years_at_company,
        ROW_NUMBER() OVER (PARTITION BY Department ORDER BY Perfomance_rating DESC, Years_at_company DESC) AS Employee_rank
    FROM hr_database.hr_info
)
SELECT 
    Employee_no,
    Department,
    Job_Role,
    Perfomance_rating,
    Years_at_company
FROM top_employees_cte
WHERE Employee_rank <= 3
ORDER BY Department, Employee_rank;
```
Demand forecast 
```sql
-- Moving average of sales over the last 7 days
SELECT 
    Product_ID,
    Product_Name,
    Order_Date,
    AVG(Sales) OVER (PARTITION BY Product_ID ORDER BY Order_Date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moving_avg_sales
FROM inventory.inventory_data_1;
```

Inventory turnover 
```sql
SELECT 
    Product_ID,
    Product_Name,
    SUM(Sales * 2) / NULLIF(AVG(quantity), 0) AS turnover_rate  -- Turnover rate = sales * cost / quantity because the dataset had no cost field
FROM inventory.inventory_data_1
GROUP BY Product_ID, Product_Name;
### Areas Needing Improvement
Departments with the Highest Attrition Rate
```
Department Atrrition
```sql
WITH department_attrition_cte AS (
    SELECT 
        Department,
        COUNT(*) AS total_employees,
        SUM(CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END) AS attrition_count
    FROM hr_database.hr_info
    GROUP BY Department
)
SELECT 
    Department,
    attrition_count,
    total_employees,
    (attrition_count / total_employees) * 100 AS attrition_rate
FROM department_attrition_cte
ORDER BY attrition_rate DESC;
```
Re-Order Point analysis 
CTE to calculate the average daily sales for each product
```
WITH daily_sales AS (
    SELECT 
        Product_ID,
        AVG(sales) AS avg_daily_sales
    FROM inventory.inventory_data_1
    GROUP BY Product_ID
),
order_time AS (                        -- CTE to calculate the average lead time for each supplier
    SELECT 
        Order_ID,
        AVG(DATEDIFF(NOW(), Order_Date)) AS avg_order_time
    FROM inventory.inventory_data_1
    GROUP BY Order_ID
)                                       -- Calculate reorder point by joining the two CTEs
SELECT 
    i.Product_ID,
    i.Product_Name,
    ds.avg_daily_sales,
    lt.avg_order_time,
    (ds.avg_daily_sales * lt.avg_order_time) AS reorder_point
FROM daily_sales ds
JOIN inventory.inventory_data_1 i ON ds.Product_ID = i.Product_ID
JOIN order_time lt ON i.Order_ID = lt.Order_ID
GROUP BY 
    i.Product_ID, 
    i.Product_Name, 
    ds.avg_daily_sales, 
    lt.avg_order_time;
```
Shipping Performance (Supplier performance)
```sql
SELECT 
    Ship_Mode,
    AVG(DATEDIFF(NOW(), Ship_Date)) AS avg_lead_time,
    COUNT(CASE WHEN quantity = sales THEN 1 END) / COUNT(*) * 100 AS fulfillment_rate
FROM inventory.inventory_data_1
GROUP BY Ship_Mode;
```
Profitability Analysis 
```sql
SELECT 
    Product_ID,
    Product_Name,
    (SUM(Sales * price) - SUM(sales * cost)) / SUM(Sales * price) AS gross_margin
FROM inventory.inventory_data_1
GROUP BY Product_ID, Product_Name;
```
ABC Analysis 
```sql
WITH product_sales AS (
    SELECT 
        Product_ID, 
        Product_Name, 
        SUM(Sales * Quantity) AS total_sales
    FROM inventory.inventory_data_1
    GROUP BY Product_ID, Product_Name
),
cumulative_sales AS (
    SELECT 
        ps.Product_ID,
        ps.Product_Name,
        ps.total_sales,
        SUM(ps.total_sales) OVER (ORDER BY ps.total_sales DESC) / (SELECT SUM(total_sales) FROM product_sales) AS cumulative_share
    FROM product_sales ps
)
SELECT 
    Product_ID,
    Product_Name,
    CASE 
        WHEN cumulative_share <= 0.8 THEN 'A'
        WHEN cumulative_share <= 0.95 THEN 'B'
        ELSE 'C'
    END AS abc_classification
FROM cumulative_sales;
```
Stockout and Overstock Analysis
Query to identify products with consistent stockouts
```sql
WITH stock_status AS (
    SELECT 
        Product_ID, 
        Product_Name, 
        Quantity,
        SUM(Sales) OVER (PARTITION BY Product_ID ORDER BY Order_Date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS total_sales_last_7_days
    FROM inventory.inventory_data_1
),
stockout_products AS (
    SELECT 
        Product_ID, 
        Product_Name,
        COUNT(CASE WHEN Quantity = 0 AND total_sales_last_7_days > 0 THEN 1 END) AS stockout_count
    FROM stock_status
    GROUP BY Product_ID, Product_Name
)
SELECT 
    Product_ID,
    Product_Name,
    stockout_count
FROM stockout_products
WHERE stockout_count > 0
ORDER BY stockout_count DESC;
### Diversity and Inlusion Analysis 
Gender Diversity by Department
```sql
SELECT 
    Department,
    Gender,
    COUNT(*) AS employee_count,
    (COUNT(*) / (SELECT COUNT(*) FROM hr_database.hr_info)) * 100 AS gender_percentage
FROM hr_database.hr_info
GROUP BY Department, Gender
ORDER BY Department, gender_percentage DESC;
```
Age Diversity by Department
```sql
SELECT 
    Department,
    CF_age_band AS age_group,
    COUNT(*) AS employee_count,
    (COUNT(*) / (SELECT COUNT(*) FROM hr_database.hr_info)) * 100 AS age_group_percentage
FROM hr_database.hr_info
GROUP BY Department, CF_age_band
ORDER BY Department, age_group_percentage DESC;
```
Diversity by Education Field
```sql
SELECT 
    Department,
    Education_field,
    COUNT(*) AS employee_count,
    (COUNT(*) / (SELECT COUNT(*) FROM hr_database.hr_info)) * 100 AS education_field_percentage
FROM hr_database.hr_info
GROUP BY Department, Education_field
ORDER BY Department, education_field_percentage DESC;
```
### Promotion Effectiveness Analysis
Impact of Promotion on Performance Rating:
```sql
WITH pre_promotion AS (
    SELECT 
        Employee_no,
        Perfomance_rating AS pre_promotion_rating,
        Years_since_promotion
    FROM hr_database.hr_info
    WHERE Years_since_promotion > 0
), 
post_promotion AS (
    SELECT 
        Employee_no,
        Perfomance_rating AS post_promotion_rating
    FROM hr_database.hr_info
    WHERE Years_since_promotion = 0
)
SELECT 
    pre.Employee_no,
    pre.pre_promotion_rating,
    post.post_promotion_rating,
    (post.post_promotion_rating - pre.pre_promotion_rating) AS performance_change
FROM pre_promotion pre
JOIN post_promotion post
ON pre.Employee_no = post.Employee_no
ORDER BY performance_change DESC;
```
Attrition Rate Before and After Promotion
```sql
WITH promoted_employees AS (
    SELECT 
        Employee_no,
        Attrition,
        Years_since_promotion
    FROM hr_database.hr_info
)
SELECT 
    CASE 
        WHEN Years_since_promotion = 0 THEN 'Post-Promotion'
        ELSE 'Pre-Promotion'
    END AS promotion_status,
    COUNT(*) AS employee_count,
    SUM(CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END) AS attrition_count,
    (SUM(CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END) / COUNT(*)) * 100 AS attrition_rate
FROM promoted_employees
GROUP BY promotion_status;
```
