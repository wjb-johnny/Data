
This file uses the dataset provided by Dillards. The dataset contains data about transactions from Aug 2004 to Aug 2005 and its stores and departments.   
The relational schema is shown as below:  

<img src="https://github.com/wjb-johnny/Data/blob/master/Database/Dillards_Teradata_SQL_graphs/Relational_Schema.png?raw=true" width="600">

Note: This dataset is based on Teradata SQL.

### 1. How many distinct dates are there in the saledate column of the transactiontable for each month/year combination in the database? 


```python
SELECT   
    COUNT (DISTINCT SALEDATE),  
    EXTRACT (YEAR from SALEDATE) AS year_num,  
    EXTRACT (MONTH from SALEDATE) AS month_num  
FROM TRNSACT  
GROUP BY year_num,month_num  
ORDER BY year_num ASC, month_num ASC  
```

**Result:**  
<img src="https://github.com/wjb-johnny/Data/blob/master/Database/Dillards_Teradata_SQL_graphs/1.png?raw=true" width="300">

### 2. Use a CASE statement within an aggregate function to determine which skuhad the greatest total sales during the combined summer months of June, July, and August. 



```python
SELECT SKU,SUM(AMT) AS sales
FROM TRNSACT
WHERE (EXTRACT (MONTH from SALEDATE) = 6
    OR EXTRACT (MONTH from SALEDATE) = 7
    OR EXTRACT (MONTH from SALEDATE) = 8)
    AND STYPE = 'P'
GROUP BY SKU
ORDER BY sales DESC
```

**Result:**  
<img src="https://github.com/wjb-johnny/Data/blob/master/Database/Dillards_Teradata_SQL_graphs/2.png?raw=true" width="250">

### 3. How many distinct dates are there in the saledate column of the transactiontable for each month/year/store combination in the database? Sort your results by the number of days per combination in ascending order. 


```python
SELECT 
	EXTRACT (YEAR from SALEDATE) AS year_num,
	EXTRACT (MONTH from SALEDATE) AS month_num,
        STORE,
        COUNT (DISTINCT SALEDATE) as day_num
FROM TRNSACT
GROUP BY year_num,month_num, STORE
ORDER BY day_num ASC
```

**Result:**  
<img src="https://github.com/wjb-johnny/Data/blob/master/Database/Dillards_Teradata_SQL_graphs/3.png?raw=true" width="400">

### 4. What is the average daily revenue for each store/month/year combination inthe database? Calculate this by dividing the total revenue for a group by the number of sales days available in the transaction table for that group.   
Remember to clean data: 
1. only examine purchases (not returns)
2. exclude all stores with less than 20 days of data
3. exclude all data from August, 2005


```python
SELECT 
        year_month,
        STORE,
        day_num,
        sales_sum,
        (sales_sum/day_num) as average_sales
FROM (
      SELECT 
    	     EXTRACT (YEAR from SALEDATE)||EXTRACT (MONTH from SALEDATE) AS year_month,
             STORE,
             COUNT (DISTINCT SALEDATE) as day_num,
             SUM(AMT) as sales_sum
      FROM TRNSACT
      WHERE STYPE = 'P' 
        AND SALEDATE IS NOT NULL
        AND SALEDATE < '2005-08-01'
      GROUP BY year_month, STORE
      HAVING  day_num >= 20
     ) AS cleaned_data
GROUP BY year_month, day_num, STORE, sales_sum
ORDER BY average_sales DESC
```

**Result:**  
<img src="https://github.com/wjb-johnny/Data/blob/master/Database/Dillards_Teradata_SQL_graphs/4.png?raw=true" width="600">

### 5. What is the average daily revenue brought in by Dillard’s stores in areas of high, medium, or low levels of high school education?   
Define areas of “low” education as those that have high school graduation rates between 50-60%, areas of “medium” education as those that have high school graduation rates between 60.01-70%, and areas of “high” education as those that have high school graduation rates of above 70%. 


```python
SELECT (CASE 
        WHEN msa.MSA_HIGH >= 50 AND msa.MSA_HIGH <= 60 THEN 'low'
        WHEN msa.MSA_HIGH > 60 AND msa.MSA_HIGH <= 70 THEN 'medium'
        WHEN msa.MSA_HIGH > 70 THEN 'high'
        END) AS high_school_quality,
        (SUM(sales_sum)/SUM(day_num)) as average_sales
FROM (
      SELECT 
    	     EXTRACT (YEAR from SALEDATE)||EXTRACT (MONTH from SALEDATE) AS year_month,
             STORE,
             COUNT (DISTINCT SALEDATE) as day_num,
             SUM(AMT) as sales_sum
      FROM TRNSACT
      WHERE STYPE = 'P' 
        AND SALEDATE IS NOT NULL
        AND SALEDATE < '2005-08-01'
      GROUP BY year_month, STORE
      HAVING  day_num >= 20
     ) AS cleaned_data 
     LEFT JOIN STORE_MSA msa
ON cleaned_data.STORE = msa.Store
GROUP BY high_school_quality
ORDER BY average_sales DESC
```

**Result:**  
<img src="https://github.com/wjb-johnny/Data/blob/master/Database/Dillards_Teradata_SQL_graphs/5.png?raw=true" width="250">

### 6. Compare the average daily revenues of the stores with the highest medianmsa_income and the lowest median msa_income. In what city and state were these stores, and which store had a higher average daily revenue?


```python
SELECT 
        cleaned_data.STORE,
        msa.city,
        msa.state,
        msa.msa_income,
        (SUM(sales_sum)/SUM(day_num)) as average_sales

FROM (
      SELECT 
    	     EXTRACT (YEAR from SALEDATE)||EXTRACT (MONTH from SALEDATE) AS year_month,
             STORE,
             COUNT (DISTINCT SALEDATE) as day_num,
             SUM(AMT) as sales_sum
      FROM TRNSACT
      WHERE STYPE = 'P' 
        AND SALEDATE IS NOT NULL
        AND SALEDATE < '2005-08-01'
      GROUP BY year_month, STORE
      HAVING  day_num >= 20
     ) AS cleaned_data
     LEFT JOIN store_msa msa
     ON cleaned_data.store = msa.store

WHERE msa.msa_income IN (SELECT MAX(msa_income) FROM store_msa)
    OR msa.msa_income IN (SELECT MIN(msa_income) FROM store_msa)

GROUP BY cleaned_data.STORE, msa.city, msa.state, msa.msa_income
```

**Result:**  
<img src="https://github.com/wjb-johnny/Data/blob/master/Database/Dillards_Teradata_SQL_graphs/6.png?raw=true" width="600">

### 7. What is the brand of the sku with the greatest standard deviation in sprice? Only examine skus that have been part of over 100 transactions. 


```python
SELECT TOP 1 info.brand, info.sku, cleaned_data.sd
FROM skuinfo info 
     LEFT JOIN(
                SELECT STDDEV_SAMP(sprice) AS sd,
                       sku
                FROM trnsact
                WHERE sku IS NOT NULL 
                  AND STYPE = 'P'
                GROUP BY sku
                HAVING COUNT(sprice) > 100
              ) AS cleaned_data
     ON info.sku = cleaned_data.sku
GROUP BY info.brand, info.sku, cleaned_data.sd
ORDER BY cleaned_data.sd DESC;
```

**Result:**  
<img src="https://github.com/wjb-johnny/Data/blob/master/Database/Dillards_Teradata_SQL_graphs/7.png?raw=true" width="300">

### 8. What was the average daily revenue Dillard’s brought in during each month of the year? 


```python
SELECT 
        month_num,
        (SUM(sales_sum)/SUM(day_num)) as average_sales
FROM (
      SELECT 
    	     EXTRACT(MONTH from SALEDATE) AS month_num,
             STORE,
             COUNT (DISTINCT SALEDATE) as day_num,
             SUM(AMT) as sales_sum
      FROM TRNSACT
      WHERE STYPE = 'P' 
        AND SALEDATE IS NOT NULL
        AND SALEDATE < '2005-08-01'
      GROUP BY month_num, STORE
      HAVING  day_num >= 20
     ) AS cleaned_data
GROUP BY month_num
ORDER BY average_sales DESC
```

**Result:**  
<img src="https://github.com/wjb-johnny/Data/blob/master/Database/Dillards_Teradata_SQL_graphs/8.png?raw=true" width="200">

### 9. Which department, in which city and state of what store, had the greatest % increase in average daily sales revenue from November to December? 


```python
SELECT 
        str.store, str.city, str.state, d.deptdesc,
        SUM (CASE WHEN EXTRACT(MONTH from saledate) = 12 THEN amt END) AS dece_sales_sum,
        SUM (CASE WHEN EXTRACT(MONTH from saledate) = 11 THEN amt END) AS nov_sales_sum,
        COUNT (DISTINCT CASE WHEN EXTRACT(MONTH from saledate) = 12 THEN saledate END) AS dece_day_sum,
        COUNT (DISTINCT CASE WHEN EXTRACT(MONTH from saledate) = 11 THEN saledate END) AS nov_day_sum,
        (dece_sales_sum/dece_day_sum) AS dece_average,
        (nov_sales_sum/nov_day_sum) AS nov_average,
        ((dece_average-nov_average)/nov_average) * 100 AS percentage_increase
FROM trnsact trn
LEFT JOIN strinfo str ON str.store=trn.store
LEFT JOIN skuinfo ski ON ski.sku=trn.sku
LEFT JOIN deptinfo d ON d.dept=ski.dept

WHERE stype = 'P' 
 AND saledate IS NOT NULL
 AND saledate < '2005-08-01'

GROUP BY str.store, str.city, str.state, d.deptdesc
HAVING dece_sales_sum >= 1000 AND nov_sales_sum >= 1000
ORDER BY percentage_increase DESC

```

**Result:**  
<img src="https://github.com/wjb-johnny/Data/blob/master/Database/Dillards_Teradata_SQL_graphs/9.png?raw=true" width="1000">

### 10. What is the city and state of the store that had the greatest decrease in average daily revenue from August to September? 


```python

SELECT 
        str.store, str.city, str.state,
        SUM (CASE WHEN EXTRACT(MONTH from saledate) = 8 THEN amt END) AS aug_sales_sum,
        SUM (CASE WHEN EXTRACT(MONTH from saledate) = 9 THEN amt END) AS sep_sales_sum,
        COUNT (DISTINCT CASE WHEN EXTRACT(MONTH from saledate) = 8 THEN saledate END) AS aug_day_sum,
        COUNT (DISTINCT CASE WHEN EXTRACT(MONTH from saledate) = 9 THEN saledate END) AS sep_day_sum,
        (aug_sales_sum/aug_day_sum) AS aug_average,
        (sep_sales_sum/sep_day_sum) AS sep_average,
        ((aug_average-sep_average)/sep_average) * 100 AS percentage_increase
FROM trnsact trn
LEFT JOIN strinfo str ON str.store=trn.store

WHERE stype = 'P' 
 AND saledate IS NOT NULL
 AND saledate < '2005-08-01'
 AND amt IS NOT NULL

GROUP BY str.store, str.city, str.state
HAVING aug_day_sum <> 0
 AND sep_day_sum <> 0
ORDER BY percentage_increase ASC;
```

**Result:**  
<img src="https://github.com/wjb-johnny/Data/blob/master/Database/Dillards_Teradata_SQL_graphs/10.png?raw=true" width="1000">

### 11. Determine the month of maximum total revenue for each store. Count the number of stores whose month of maximum total revenue was in each of the twelve months. Then determine the month of maximum average daily revenue. Count the number of stores whose month of maximum average daily revenue was in each of the twelve months. How do they compare? 


```python
SELECT rank_data.month_num,
       SUM(CASE WHEN average_sales_rank = 12 THEN 1 END) AS max_average_sales,
       SUM(CASE WHEN total_sales_rank = 12 THEN 1 END) AS max_total_sales
FROM
(
SELECT 
        month_num,
        store,
        day_num,
        sales_sum,
        (sales_sum/day_num) as average_sales,
        ROW_NUMBER() OVER (PARTITION BY store
                           ORDER BY average_sales DESC) AS average_sales_rank,
        ROW_NUMBER() OVER (PARTITION BY store
                           ORDER BY sales_sum DESC) AS total_sales_rank
FROM (
      SELECT 
    	     EXTRACT (MONTH from SALEDATE) AS month_num,
             store,
             COUNT (DISTINCT SALEDATE) as day_num,
             SUM(amt) as sales_sum
      FROM trnsact
      WHERE stype = 'P' 
        AND saledate IS NOT NULL
        AND saledate < '2005-08-01'
      GROUP BY month_num, store
      HAVING  day_num >= 20
     ) AS cleaned_data
GROUP BY month_num, store, sales_sum, day_num
) AS rank_data
GROUP BY rank_data.month_num
ORDER BY rank_data.month_num ASC
```

**Result:**  
<img src="https://github.com/wjb-johnny/Data/blob/master/Database/Dillards_Teradata_SQL_graphs/11.png?raw=true" width="400">
