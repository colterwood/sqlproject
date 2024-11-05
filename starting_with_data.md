Question 1: What are the top selling products by total transaction revenue and total quantity ordered?

SQL Queries:
REVENUE:
SELECT
	sr.name,
	sr.total_ordered,
	s.totaltransactionrevenue / 1000000 AS total_revenue
FROM
	sales_report sr
JOIN
	sessions s USING(productsku)
WHERE
	s.totaltransactionrevenue IS NOT NULL
ORDER BY
	total_revenue DESC

QUANTITY ORDERED:
SELECT
	sr.name,
	sr.total_ordered,
	s.totaltransactionrevenue / 1000000 AS total_revenue
FROM
	sales_report sr
JOIN
	sessions s USING(productsku)
WHERE
	s.totaltransactionrevenue IS NOT NULL
ORDER BY
	2 DESC

Answer: 
REVENUE:
![image](https://github.com/user-attachments/assets/82fb0d32-9581-4249-b66a-42679c563149)


QUANTITY:
![image](https://github.com/user-attachments/assets/3ec6a30f-806a-4f61-b58e-35fe5d11eca5)



Question 2: Which are the Top 10 cities and countries for the most page views?

SQL Queries:
CITIES:
SELECT
	CASE
		WHEN city LIKE 'not available%'
		THEN '(not set)'
		ELSE city
	END AS city,
	country,
	SUM(pageviews) AS total_pageviews
FROM
	sessions
WHERE
	(CASE
		WHEN city LIKE 'not available%'
		THEN '(not set)'
		ELSE city
	END)	!= '(not set)'
GROUP BY
	1,2
ORDER BY
	3 DESC LIMIT 10

 COUNTRIES:
 SELECT
	country,
	SUM(pageviews) AS total_pageviews
FROM
	sessions
WHERE
	country != '(not set)'
GROUP BY
	1
ORDER BY
	2 DESC LIMIT 10
 
Answer:
CITIES:
![image](https://github.com/user-attachments/assets/89709ebf-a5d7-496b-94e0-a0d029ba3598)

COUNTRIES:
![image](https://github.com/user-attachments/assets/05f07eb2-b18f-49b8-934c-5fe20bab54df)


Question 3: Which channel groupings generate the most pageviews? Which ones generate the most sales? List the Top 5

SQL Queries:
PAGE VIEWS:
SELECT
	channelgrouping,
	SUM(pageviews)
FROM
	sessions
GROUP BY
	1
ORDER BY
	2 DESC LIMIT 5

SALES:
SELECT
	s.channelgrouping,
	SUM(sr.total_ordered) AS products_sold,
	SUM(s.totaltransactionrevenue / 1000000) AS total_sales
FROM
	sessions s
JOIN
	sales_report sr USING (productsku)
WHERE
	s.totaltransactionrevenue IS NOT NULL
GROUP BY
	1
ORDER BY
	3 DESC LIMIT 5

Answer:
PAGE VIEWS:
![image](https://github.com/user-attachments/assets/41bf4b4a-412b-4437-8492-55a16599fab2)

SALES:
![image](https://github.com/user-attachments/assets/c948bd86-ea63-4110-8ff6-d8b30a277462)



Question 4: What percentage of visitors return to the site?

SQL Queries:

WITH visitor_count AS (
		SELECT
			fullvisitorid,
			COUNT(*) AS visits
		FROM
			sessions
		GROUP BY
			1
	),
	return_visitors AS (
		SELECT
			COUNT(*) AS return_user_count
		FROM
			visitor_count
		WHERE
			visits > 1
	),
	total_visitors AS (
		SELECT
			COUNT(*) AS total_user_count
		FROM
			visitor_count
	)
SELECT
	(SELECT return_user_count FROM return_visitors) AS returning_users,
	(SELECT total_user_count FROM total_visitors) AS total_users,
	100.0 * (SELECT return_user_count FROM return_visitors)::decimal / 
		(SELECT total_user_count FROM total_visitors) AS return_user_percentage
  
Answer:
![image](https://github.com/user-attachments/assets/3bf290b1-089f-4af9-a934-168eafe27ba8)



Question 5: Which users have ordered more than once?

SQL Queries:
SELECT
	fullvisitorid,
	COUNT(*)
FROM
	sessions
WHERE	
	totaltransactionrevenue IS NOT NULL
GROUP BY
	1
HAVING
	COUNT(*) > 1
ORDER BY
	2 DESC


Answer:
![image](https://github.com/user-attachments/assets/3b8425af-3a70-470c-89c0-6fe3e280c3f2)
