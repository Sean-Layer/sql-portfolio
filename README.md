# sql-portfolio
Examples of my work using SQL

Data is from a ficticious bicycle company (Adventure Works) from datacamp. 

SQL Dialect is Standard SQL written using Google BigQuery.

There are three tables I'll be pulling from
* fact_internet_sales which contains sales data including revenues, costs, region key based on where it was sold, product key based on what item was sold, and more. 
* dim_product which contains product descriptions and product names in multiple languages.
* dim_sales_territory which contains the names of the countries we serve, their regions, and similar.
* These tables are stored in a dataset on BigQuery called Adventure Works

dim_product and fact_internet_sales join on product key

dim_sales_territory and fact_internet_sales join on region key

## Query 1
This query will use a common table expression and aggregate functions to calculate total sales, cost of goods sold, and profit for years 2011-2014.

```SQL
WITH t1 AS (
    SELECT 
        EXTRACT(year from OrderDate) AS year,
        ROUND(SUM(SalesAmount),2) AS total_sales,
        ROUND(SUM(TotalProductCost) + SUM(TaxAmt) + SUM(Freight),2) AS cogs
    FROM 
        AdventureWorks.fact_internet_sales
    GROUP BY 
        year)
SELECT 
    year,
    total_sales,
    cogs,
    ROUND(total_sales - cogs,2) AS total_profit
FROM t1
ORDER BY 
    year;
```
## Query 1 Output
![image not found](https://i.imgur.com/X5UzyZj.png)

## Query 2
This query selects the top 10 best selling products by sales in dollars. It uses a left join to achieve the desired output.

```SQL
SELECT 
    p.EnglishProductName AS product_name,
    ROUND(SUM(salesamount),2) AS total_sales
FROM 
    AdventureWorks.fact_internet_sales AS f   
LEFT JOIN AdventureWorks.dim_product as p
ON f.productkey = p.productkey
GROUP BY 
    p.EnglishProductName
ORDER BY 
    total_sales DESC
LIMIT 10;
```

## Query 2 output
![image not found](https://i.imgur.com/MK06VhH.png)

## Query 3
This final query finds the top selling product in each US Sales region by the number of items sold. To achieve this result I use a subquery in the from statement that uses a window function to get the top_n result

```SQL
SELECT 
    product_name,
    region,
    units_sold  
FROM 
    (SELECT 
        p.EnglishProductName as product_name,
        t.SalesTerritoryRegion as region,
        COUNT(*) as units_sold,
        ROW_NUMBER() OVER(PARTITION BY t.SalesTerritoryRegion ORDER BY COUNT(*) DESC) AS top_cat
    FROM 
        AdventureWorks.fact_internet_sales as f
    LEFT JOIN AdventureWorks.dim_product as p 
    ON f.productkey = p.productkey 
    LEFT JOIN AdventureWorks.dim_sales_territory as t
        ON f.salesterritorykey = t.salesterritorykey  
    WHERE 
        t.salesterritorycountry = 'USA'
    GROUP BY 
        product_name, region)  
WHERE top_cat = 1;
```

## Query 3 output
![image not found](https://i.imgur.com/O4czZIQ.png)
