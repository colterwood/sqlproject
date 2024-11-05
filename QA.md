What are your risk areas? Identify and describe them.
-The data is a mess and really tough to piece together
-Not all tables are easily linked together


QA Process:
Describe your QA process and include the SQL queries used to execute it.
-For question 1 I compared the total cumulative amounts for totaltransactionrevenue for each the cities and countries against the sum of totaltransactionrevenue, to ensure the numbers added up.
-For question 2 I added the SUM(sbs.total_ordered), COUNT(*), and SUM(sbs.total_ordered) / COUNT(*) columns to the query to confirm the avg ordered was giving me the same answer.
-For question 3 I used the following query to spot check the order totals at different cities and compared that to my query in Q2 where I was accessing the total_orders to make sure they matched, changing the city name in the WHERE statement to. I spot checked 6 different cities (Gonna note that I kinda hate this and am actively seeking feedback to make this better!):

SELECT
    city,
    country,
    product_category,
    order_count,
    ordered_quantity,
    SUM(order_count) OVER (ORDER BY row_num) AS cum_order_count
FROM (
    SELECT
        *,
        ROW_NUMBER() OVER (ORDER BY city, product_category) AS row_num
    FROM (
        SELECT
            CASE
                WHEN s.city LIKE 'not available%'
                THEN '(not set)'
                ELSE s.city
            END AS city,
            s.country,
            CASE
                WHEN s.v2productcategory = '${escCatTitle}' 
                THEN '(not set)'
                ELSE regexp_replace(
                    regexp_replace(
                        regexp_replace(v2productcategory, '^.*/([^/]+)/[^/]*$', '\1'), -- Removes the last '/' and everything before and including the second last '/'
                        '/$', ''                   -- Removes the '/' at the end
                    ),
                    '''', ''                      -- Removes every instance of the single quote
                )
            END AS product_category,
            COUNT(*) AS order_count,
            p.orderedquantity AS ordered_quantity
        FROM
            sessions s
        JOIN
            products p
        ON
            s.productsku = p.sku
        WHERE
            p.orderedquantity > 0
            AND s.city = 'Barcelona'
        GROUP BY
            3, 1, 2, 5
    ) AS inner_query
) AS subquery
ORDER BY
    1, 3;


Did the same for countries using this query:

SELECT
    city,
    country,
    product_category,
    order_count,
    ordered_quantity,
    SUM(order_count) OVER (ORDER BY row_num) AS cum_order_count
FROM (
    SELECT
        *,
        ROW_NUMBER() OVER (ORDER BY city, product_category) AS row_num
    FROM (
        SELECT
            CASE
                WHEN s.city LIKE 'not available%'
                THEN '(not set)'
                ELSE s.city
            END AS city,
            s.country,
            CASE
                WHEN s.v2productcategory = '${escCatTitle}' 
                THEN '(not set)'
                ELSE regexp_replace(
                    regexp_replace(
                        regexp_replace(v2productcategory, '^.*/([^/]+)/[^/]*$', '\1'), -- Removes the last '/' and everything before and including the second last '/'
                        '/$', ''                   -- Removes the '/' at the end
                    ),
                    '''', ''                      -- Removes every instance of the single quote
                )
            END AS product_category,
            COUNT(*) AS order_count,
            p.orderedquantity AS ordered_quantity
        FROM
            sessions s
        JOIN
            products p
        ON
            s.productsku = p.sku
        WHERE
            p.orderedquantity > 0
            AND s.city = 'Barcelona'
        GROUP BY
            3, 1, 2, 5
    ) AS inner_query
) AS subquery
ORDER BY
    1, 3;

-For question 4 and Q5 I was able to compare numbers against the previous queries to ensure consistency
