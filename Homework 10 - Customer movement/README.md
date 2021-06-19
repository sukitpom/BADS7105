# Customer movment by BIG Query and SQL Command

![Chart](https://github.com/sukitpom/BADS7105/blob/master/Homework%2010%20-%20Customer%20movement/img/Chart.png)

## Learning Step by step to create sql command -> reference :[p_new](https://github.com/tanatiem/BADS7105-CRM-Analytics/tree/main/Homework%2010%20-%20Customer%20Movement%20Analysis)

  
### Paht 1. Select customer code and month for sub-query in next step

#### With-clause syntax
https://cloud.google.com/bigquery/docs/reference/standard-sql/query-syntax#with-clause

#### Convert date int to datetime type
    date_trunc(PARSE_DATE('%Y%m%d', CAST(shop_date AS STRING)), month)

#### Select customer code and shop month for sub query in next step
    WITH cust_month AS (
      select distinct cust_code, date_trunc(PARSE_DATE('%Y%m%d', CAST(shop_date AS STRING)), month) shop_month
      from `idyllic-creek-308203.BAD7105.supermarket`
      where cust_code is not null
    )

### Path 2. Create new columns for status of customer movement (New,Repeat,Reactived).

LAG and LEAD Function:https://cloud.google.com/bigquery/docs/reference/standard-sql/navigation_functions

#### Create Prev_month in partition table by LAG function (-1 month).
    SELECT cust_code, shop_month, LAG(shop_month, 1) OVER (PARTITION BY cust_code ORDER BY shop_month) AS prev_month
            FROM cust_month

#### Create status columns by Data_diff function between shop_month and prev_month.
    SELECT cust_code, shop_month AS report_month, prev_month
        ,CASE 
            WHEN DATE_DIFF(shop_month, prev_month, MONTH) IS NULL THEN 'New'
            WHEN DATE_DIFF(shop_month, prev_month, MONTH) = 1 THEN 'Repeat'
            WHEN DATE_DIFF(shop_month, prev_month, MONTH) > 1 THEN 'Reactivated'
        ELSE NULL END AS status
    FROM (
        SELECT cust_code, shop_month, LAG(shop_month, 1) OVER (PARTITION BY cust_code ORDER BY shop_month) AS prev_month
        FROM cust_month
        )
![status](https://github.com/sukitpom/BADS7105/blob/master/Homework%2010%20-%20Customer%20movement/img/subquery-1.png)


### Path 3. Create churn status from date_diff between current month and next one month.

#### Create date diff between current month and next month.
    SELECT cust_code, shop_month
        ,LEAD(shop_month, 1) OVER (PARTITION BY cust_code ORDER BY shop_month) AS next_trans_date
        ,DATE_DIFF(LEAD(shop_month, 1) OVER (PARTITION BY cust_code ORDER BY shop_month), shop_month, MONTH) AS n_months
    FROM cust_month
![Date_diff](https://github.com/sukitpom/BADS7105/blob/master/Homework%2010%20-%20Customer%20movement/img/subquery-2_check_date_diff.png)

#### Select churn status base on max(shop_month).
    SELECT cust_code, DATE_ADD(shop_month, INTERVAL 1 MONTH) report_month, shop_month as prev_month
        ,'Churn' AS status
    FROM (
        SELECT cust_code, shop_month
            ,LEAD(shop_month, 1) OVER (PARTITION BY cust_code ORDER BY shop_month) AS next_trans_date
            ,DATE_DIFF(LEAD(shop_month, 1) OVER (PARTITION BY cust_code ORDER BY shop_month), shop_month, MONTH) AS n_months
        FROM cust_month
    )

#### Filter date diff > 1 or null value and excluded last month.
    SELECT cust_code, DATE_ADD(shop_month, INTERVAL 1 MONTH) report_month, shop_month as prev_month
        ,'Churn' AS status
    FROM (
        SELECT cust_code, shop_month
            ,LEAD(shop_month, 1) OVER (PARTITION BY cust_code ORDER BY shop_month) AS next_trans_date
            ,DATE_DIFF(LEAD(shop_month, 1) OVER (PARTITION BY cust_code ORDER BY shop_month), shop_month, MONTH) AS n_months
        FROM cust_month
    ) WHERE 
        (n_months > 1 or n_months is null)
        AND shop_month < (SELECT MAX(shop_month) FROM cust_month)-- excluding last month
![Churn](https://github.com/sukitpom/BADS7105/blob/master/Homework%2010%20-%20Customer%20movement/img/subquery-2_churn_0.png)

### Part 4. Union all status and then count no. of customer by status
    WITH cust_month AS (
        select distinct cust_code, date_trunc(PARSE_DATE('%Y%m%d', CAST(shop_date AS STRING)), month) shop_month
        from `idyllic-creek-308203.BAD7105.supermarket`
        where cust_code is not null
    )
    SELECT report_month
        ,COUNT(DISTINCT CASE WHEN status = 'New' THEN cust_code ELSE NULL END) AS new_customers
        ,COUNT(DISTINCT CASE WHEN status = 'Repeat' THEN cust_code ELSE NULL END) AS repeat_customers
        ,COUNT(DISTINCT CASE WHEN status = 'Reactivated' THEN cust_code ELSE NULL END) AS reactivated_customers
        ,-COUNT(DISTINCT CASE WHEN status = 'Churn' THEN cust_code ELSE NULL END) AS churn_customers
    FROM (
        SELECT cust_code, shop_month AS report_month, prev_month
            ,CASE 
                WHEN DATE_DIFF(shop_month, prev_month, MONTH) IS NULL THEN 'New'
                WHEN DATE_DIFF(shop_month, prev_month, MONTH) = 1 THEN 'Repeat'
                WHEN DATE_DIFF(shop_month, prev_month, MONTH) > 1 THEN 'Reactivated'
            ELSE NULL END AS status
        FROM (
            SELECT cust_code, shop_month, LAG(shop_month, 1) OVER (PARTITION BY cust_code ORDER BY shop_month) AS prev_month
            FROM cust_month
        )
        UNION ALL
        SELECT cust_code, DATE_ADD(shop_month, INTERVAL 1 MONTH) report_month, shop_month as prev_month
            ,'Churn' AS status
        FROM (
            SELECT cust_code, shop_month
                ,LEAD(shop_month, 1) OVER (PARTITION BY cust_code ORDER BY shop_month) AS next_trans_date
                ,DATE_DIFF(LEAD(shop_month, 1) OVER (PARTITION BY cust_code ORDER BY shop_month), shop_month, MONTH) AS n_months
            FROM cust_month
        ) WHERE 
            (n_months > 1 or n_months is null)
            AND shop_month < (SELECT MAX(shop_month) FROM cust_month)-- excluding last month
    ) GROUP BY report_month
    order by report_month

#### Final result:
![result](https://github.com/sukitpom/BADS7105/blob/master/Homework%2010%20-%20Customer%20movement/img/data_result.png)
