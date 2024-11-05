Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**

SQL Queries:
**Commented out lines were put in to verify the total matched the total of Query number 2**
SELECT	
    CASE	
         WHEN city LIKE 'not available%'
         THEN '(not set)'
         ELSE city
    END AS city,
	country,
	SUM(totaltransactionrevenue / 1000000) AS total_revenue
--	,SUM(SUM(totaltransactionrevenue / 1000000)) 
--		OVER (ORDER BY SUM(totaltransactionrevenue / 1000000)DESC)
--		AS cum_total_revenue
FROM
	sessions
WHERE
	totaltransactionrevenue IS NOT NULL
GROUP BY
	1,2
ORDER BY
	3 DESC

SELECT	
	country,
	SUM(totaltransactionrevenue / 1000000) AS total_revenue
--	,SUM(SUM(totaltransactionrevenue / 1000000)) 
--		OVER (ORDER BY SUM(totaltransactionrevenue / 1000000)DESC)
--		AS cum_total_revenue
FROM
	sessions
WHERE
	totaltransactionrevenue IS NOT NULL
GROUP BY
	1
ORDER BY
	2 DESC



**This query was just used to confirm that the total_revenue of the above query matched the total_revenue for the table**
SELECT
	sum(totaltransactionrevenue) AS total_revenue
FROM
	sessions

Answer:

city	                            country	        total_revenue
(not set)	                        United States	6092.560000
San Francisco	                    United States	1564.320000
Sunnyvale	                        United States	992.230000
Atlanta	                            United States	854.440000
Palo Alto	                        United States	608.000000
Tel Aviv-Yafo	                    Israel	        602.000000
New York	                        United States	530.360000
Mountain View	                    United States	483.360000
Los Angeles	                        United States	479.480000
Chicago	                            United States	449.520000
Sydney	                            Australia	    358.000000
Seattle	                            United States	358.000000
San Jose	                        United States	262.380000
Austin	                            United States	157.780000
Nashville	                        United States	157.000000
San Bruno	                        United States	103.770000
Toronto	                            Canada	        82.160000
New York	                        Canada	        67.990000
Houston	                            United States	38.980000
Columbus	                        United States	21.990000
Zurich	                            Switzerland	    16.990000

country	            total_revenue
United States	    13154.170000
Israel	            602.000000
Australia	        358.000000
Canada	            150.150000
Switzerland	        16.990000




**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:
**I changed all cities that began with "not available" to be (not set) to match the formatting of the countries that were not set or not available in the data.**
**The commented out lines from the query are there to check that the avg_ordered_product column is correct.**
SELECT
    CASE	
         WHEN s.city LIKE 'not available%'
         THEN '(not set)'
         ELSE s.city
    END AS city,
	s.country,
--	SUM(sbs.total_ordered) AS total_ordered,
--	COUNT(*) AS total_orders,
--	SUM(sbs.total_ordered) / COUNT(*) AS avg_check,
	AVG(sbs.total_ordered) AS avg_ordered_product
FROM
	sessions s
JOIN
	sales_by_sku sbs USING(productsku) 
WHERE
	s.totaltransactionrevenue IS NOT NULL
GROUP BY
	1,2
ORDER BY
	1

SELECT
	s.country,
--	SUM(sbs.total_ordered) AS total_ordered,
--	COUNT(*) AS total_orders,
--	SUM(sbs.total_ordered) / COUNT(*) AS avg_check,
	AVG(sbs.total_ordered) AS avg_ordered_product
FROM
	sessions s
JOIN
	sales_by_sku sbs USING(productsku) 
WHERE
	s.totaltransactionrevenue IS NOT NULL
GROUP BY
	1
ORDER BY
	1


Answer:
CITIES:
![image](https://github.com/user-attachments/assets/2fec20e4-0502-46ed-a294-7f062aba028d)




COUNTRIES:
![image](https://github.com/user-attachments/assets/91b1911a-6118-4ead-8b47-8c0d76998cfd)




**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**
SQL Queries:
CITIES:
SELECT
	CASE
		WHEN s.city LIKE 'not available%'
		THEN '(not set)'
		ELSE s.city
	END AS city,
	s.country,
	CASE
		WHEN s.v2productcategory = '${escCatTitle}' 
		THEN '(not set)'		--Sets ${escCatTitle} as (not set) for consistency
		ELSE regexp_replace(
			regexp_replace(
				regexp_replace(v2productcategory, '^.*/([^/]+)/[^/]*$', '\1'), --Removes the last '/' and everything before and including the second last '/'
				'/$',''),		--removes the '/' at the end
			'''',''		--Removes every instance of the single quote
			)
	END AS product_category,
	COUNT(*) AS order_count
FROM
	sessions s
JOIN
	sales_by_sku sbs USING(productsku)
WHERE
	s.totaltransactionrevenue IS NOT NULL
GROUP BY
	3,1,2
ORDER BY
	1,3

COUNTRIES:
SELECT
	s.country,
	CASE
		WHEN s.v2productcategory = '${escCatTitle}' 
		THEN '(not set)'		--Sets ${escCatTitle} as (not set) for consistency
		ELSE regexp_replace(
			regexp_replace(
				regexp_replace(v2productcategory, '^.*/([^/]+)/[^/]*$', '\1'), --Removes the last '/' and everything before and including the second last '/'
				'/$',''),		--removes the '/' at the end
			'''',''		--Removes every instance of the single quote
			)
	END AS product_category,
	COUNT(*) AS order_count
FROM
	sessions s
JOIN
	sales_by_sku sbs USING(productsku)
WHERE
	s.totaltransactionrevenue IS NOT NULL
GROUP BY
	2,1
ORDER BY
	1

 
Answer:
CITIES:
![image](https://github.com/user-attachments/assets/8366b918-0121-4883-b5ef-ed334ccc3eb7)




COUNTRIES:
![image](https://github.com/user-attachments/assets/0e4d9d16-da83-4364-970b-cf2f21d330c6)





**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**
-Appears that some products may have either an order minimum or are ordered in a specific quantity (ie - 20 oz Stainless Steel Insulated Tumbler has been ordered 5 times for a quantity of 499 and Android Sticker Sheet Ultra Removable 3 times for 363)

SQL Queries:
CITIES:
SELECT
    city,
    country,
    name,
    total_ordered
FROM (
    SELECT
        CASE
            WHEN s.city LIKE 'not available%'
            THEN '(not set)'
            ELSE s.city
        END AS city,
        s.country,
        sr.name,
        SUM(sr.total_ordered) AS total_ordered,
        ROW_NUMBER() OVER (PARTITION BY s.city, s.country ORDER BY SUM(sr.total_ordered) DESC) AS rank
    FROM
        sessions s
    JOIN
        sales_report sr USING(productsku)
	WHERE
		s.totaltransactionrevenue IS NOT NULL
		AND sr.total_ordered > 0
    GROUP BY
        s.city, s.country, sr.name
) AS ranked_products
WHERE 
	rank = 1
ORDER BY
    1,2,3;

COUNTRIES:
SELECT
    country,
    name,
    total_ordered
FROM (
    SELECT
        s.country,
        sr.name,
        SUM(sr.total_ordered) AS total_ordered,
        ROW_NUMBER() OVER (PARTITION BY s.country ORDER BY SUM(sr.total_ordered) DESC) AS rank
    FROM
        sessions s
    JOIN
        sales_report sr USING(productsku)
	WHERE
		s.totaltransactionrevenue IS NOT NULL
		AND sr.total_ordered > 0
    GROUP BY
        s.country, sr.name
) AS ranked_products
WHERE 
	rank = 1
ORDER BY
    1;

Answer:
CITIES:
![image](https://github.com/user-attachments/assets/1a13b253-f435-4f7b-9fd4-f823f0e77ba8)



COUNTRIES:
![image](https://github.com/user-attachments/assets/3da88d54-64cc-411c-b92f-f7e78338d755)








**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:
CITIES:
SELECT
    CASE
        WHEN s.city LIKE 'not available%'
        THEN '(not set)'
        ELSE s.city
    END AS city,
    s.country,
    SUM(s.totaltransactionrevenue / 1000000) AS total_revenue,
    COUNT(*) AS transactions,
    AVG(s.totaltransactionrevenue / 1000000) AS avg_revenue,
    SUM(sbs.total_ordered) AS products_ordered,
    SUM(sbs.total_ordered) AS products_ordered / COUNT(*) AS transactions AS products_per_transaction
    AVG(sbs.total_ordered) AS avg_ordered_product
FROM
    sessions s
JOIN
    sales_by_sku sbs USING(productsku)
WHERE
    s.totaltransactionrevenue IS NOT NULL
GROUP BY
    city, s.country
ORDER BY
    total_revenue DESC;

COUNTRIES:
SELECT
    CASE
        WHEN s.city LIKE 'not available%'
        THEN '(not set)'
        ELSE s.city
    END AS city,
    s.country,
    SUM(s.totaltransactionrevenue / 1000000) AS total_revenue,
    COUNT(*) AS transactions,
    AVG(s.totaltransactionrevenue / 1000000) AS avg_revenue,
    SUM(sbs.total_ordered) AS products_ordered,
    SUM(sbs.total_ordered) AS products_ordered / COUNT(*) AS transactions AS products_per_transaction
    AVG(sbs.total_ordered) AS avg_ordered_product
FROM
    sessions s
JOIN
    sales_by_sku sbs USING(productsku)
WHERE
    s.totaltransactionrevenue IS NOT NULL
GROUP BY
    city, s.country
ORDER BY
    total_revenue DESC;
    
Answer:
CITIES:
![image](https://github.com/user-attachments/assets/f4da67c1-310a-4879-add9-d7a0ac816be4)

COUNTRIES:
![image](https://github.com/user-attachments/assets/36aa5d92-b283-4930-a0ca-4b53c4243015)




ERD:




