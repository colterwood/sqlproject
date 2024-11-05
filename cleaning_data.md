What issues will you address by cleaning the data?
-Handling null rows
-Formatting field names to match (ie - Q2 the cities had 'not available in demo dataset', 'not available in dataset' and '(not set)' all used as city names.)
-Ranking products to narrow down top selling products
-Simplifying category names
-Merging category names that had slight character differences for no apparent reason (ie - Kid's and Kids)
-



Queries:
Below, provide the SQL queries you used to clean your data.
**For Q1 I removes rows with NULL totaltransaction revenue. I also added the piece to change cities that start with 'not available' into '(not set)' to be consistent with what I did for later questions.** 
**Also had to divide the totaltransactionrevenue by 1,000,000**

City:
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

 Country:
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

**For Q2 I used the case statement inside this query to convert all unset/unavailable city names to be '(not set)'.**

SELECT
    CASE	
         WHEN s.city LIKE 'not available%'
         THEN '(not set)'
         ELSE s.city
    END AS city,
	s.country,
--	SUM(p.orderedquantity) AS total_ordered,
--	COUNT(*) AS total_orders,
--	SUM(p.orderedquantity) / COUNT(*) AS avg_check,
	AVG(p.orderedquantity) AS avg_ordered_product
FROM
	sessions s
JOIN
	products p 
ON
	s.productsku = p.sku
WHERE
	s.totaltransactionrevenue IS NOT NULL
GROUP BY
	1,2
ORDER BY
	1

**For Q3 I needed to clean up city names like I did in previous questions. I also had to fix some discrepancies in category names:
	1. change ${escCatTitle} to (not set)
 	2. remove all the extra characters around the product categories, focusing on the words between the last set of '/'
  	3. remove the '/' at the end of a few categories
   	4. remove the apostrophe in a few categories, for example there was a 'kids' and a 'kid's' category. My dream state would have been to actually change 'kids' to 'kid's' but I could not get this figured out, 
    		even with help from the internet. Would love to get a walkthough on this.**
**Full transparency, I needed to use the internet to figure out a few things here, wasn't going to get there on my own**

SELECT
	CASE
		WHEN s.city LIKE 'not available%'  
		THEN '(not set)'	--Change cities that start with 'not available' to be '(not set)' for consistency
		ELSE s.city
	END AS city,
	s.country,
	CASE
		WHEN s.v2productcategory = '${escCatTitle}' 
		THEN '(not set)'		--Sets ${escCatTitle} as (not set) for consistency
		ELSE regexp_replace(
			regexp_replace(
				regexp_replace(v2productcategory, '^.*/([^/]+)/[^/]*$', '\1'), --Removes the last '/' and everything before and including the second last '/'
				'/$',''),		--Removes the '/' at the end
			'''',''		--Removes every instance of the single quote
			)
	END AS product_category,
	COUNT(*) AS order_count
FROM
	sessions s
GROUP BY
	3,1,2
ORDER BY
	1,3


**For Q4 I needed to do a subquery to rank the products by city, country. I also needed to remove the products that were showing up that had 0 ordered, as they were populating in my results.**

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
        p.name,
        SUM(p.orderedquantity) AS total_ordered,
        ROW_NUMBER() OVER (PARTITION BY s.city, s.country ORDER BY SUM(p.orderedquantity) DESC) AS rank
    FROM
        sessions s
    JOIN
        products p
    ON
        s.productsku = p.sku
    GROUP BY
        s.city, s.country, p.name
	HAVING
		SUM(p.orderedquantity) > 0
) AS ranked_products
WHERE 
	rank = 1
--	AND city = 'Jacksonville'
--	AND name = '20 oz Stainless Steel Insulated Tumbler'
ORDER BY
    1,2,3;

**For Q5 I created a number of different totals and average tabs to compare some base level analytics for city and country sales**
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


**I STARTED OUT BY JUMPING INTO CLEANING SOME DATAT AND ACTUALLY UPDATING TABLES. WHAT A ROOKIE MISTAKE THAT WAS... IF YOU WANT TO SEE EVERYTHING I DID THERE, KEEP READING. IF YOU'D LIKE TO SAVE YOURSELF FROM THE HORROR, FEEL FREE TO IGNORE. GREAT LEARNING OPPORTUNITY FOR ME!
Products Table:
-Confirm there are no duplicate sku numbers
-Create consistency in name formatting, spelling, etc
-Remove unnecessary spacing in name column (At the beginning of the name and double spacing in between words)
-




Queries:
Below, provide the SQL queries you used to clean your data.
**PRODUCTS TABLE**
**To check to see if there were any duplicate sku's:**
SELECT 
  sku, 
  name, 
  COUNT(*) AS count
FROM 
  products
GROUP BY 
  1,2
HAVING 
  COUNT(*) > 1;

**To confirm there are no negatives in the orderquantity, stocklevel, or restockingleadtime**
SELECT
	*
FROM
	products
WHERE
	orderedquantity < 0
	OR stocklevel < 0
	OR restockingleadtime < 0

 **Added extension to check for spelling mistakes**
CREATE EXTENSION IF NOT EXISTS pg_trgm;

**Checked for spelling mistakes** (This was a thing I found on the google machine that I thought we be fun to try)
SELECT 
  name
FROM 
  products
WHERE 
  name % 'correct_spelling';

**Update nulls from sentimentscore and sentimentmagnitude to 0.0**
UPDATE 
  products
SET 
  sentimentscore = 0.0
WHERE 
  sentimentscore IS NULL;

UPDATE 
  products
SET 
  sentimentmagnitude = 0.0
WHERE 
  sentimentmagnitude IS NULL;

**Check the rest of the table for nulls**
SELECT
  *
FROM
  products
WHERE
  sku IS NULL
  OR name IS NULL
  OR orderquantity IS NULL
  OR stocklevel IS NULL
  OR restockingleadtime IS NULL

**Trim the extra space at the beginning of some of the names**
UPDATE 
  products
SET 
  name = LTRIM(name, ' ')
WHERE 
  name LIKE ' %';

**Replace all instances in the name column where there are two spaces but should only be one**
UPDATE 
  products
SET 
  name = REPLACE(name, '  ', ' ')
WHERE 
  name LIKE '%  %';

**Some names had 17 oz and other were 17oz (for example), so I updated the date to match the first formatting** (Full transparency, ChatGPT helped me finesse this one)
UPDATE 
  products
SET 
  name = REGEXP_REPLACE(name, '([0-9]+)oz', '\1 oz', 'g')
WHERE 
  name ~ '[0-9]+oz';

**Most names that had men's or women's were formatted with the apostrophe, but a few were not**
UPDATE 
  products
SET 
  name = REPLACE(name, 'Womens', 'Women''s')
WHERE 
  name LIKE '%mens%';

**Set sku as primary key**
ALTER TABLE 
  products
ADD CONSTRAINT 
  products_pkey PRIMARY KEY (sku);
