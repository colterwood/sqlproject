What issues will you address by cleaning the data?
Products Table:
-Confirm there are no duplicate sku numbers
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
