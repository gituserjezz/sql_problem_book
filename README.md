# SQL Problem Book
## This repository contains my solutions to interesting tasks i had to face during SQL Server training (mostly from karpov.courses)
---
### Task 1 
**Problem:** Calculate the age of each user in the `users` table. Consider age relative to the last date in the `user_actions` table. Include columns with user id and age in the result. For those users who do not have a date of birth in the `users` table, enter the average age of all other users, rounded up to a whole number. Name the age column `age`. Sort the result in ascending order of user id.

**Solution:**
``` sql
WITH ag as (SELECT date_part('YEAR', age((SELECT max(time) :: date
                                                      FROM   user_actions), birth_date)) age
            FROM   users)
SELECT user_id,
       coalesce(date_part('YEAR', age((SELECT max(time) :: date
                                FROM   user_actions), birth_date)), round((SELECT avg(age)
                                           FROM   ag))) age
FROM   users
ORDER BY user_id
```
---
### Task 2 
**Problem:** From the `orders` table, print the id and content of orders that include at least one of the five most expensive items available in our service. Sort the result in ascending order id order.

**Solution:**
``` sql
WITH s1 as (SELECT product_id
            FROM   products
            ORDER BY price desc limit 5), s2 as (SELECT order_id,
                                            unnest(product_ids) as product_id
                                     FROM   orders), s3 as (SELECT order_id
                       FROM   (SELECT *
                               FROM   s2) as s2
                       WHERE  product_id in (SELECT *
                                             FROM   s1))
SELECT order_id,
       product_ids
FROM   orders
WHERE  order_id in (SELECT *
                    FROM   s3)
ORDER BY order_id
```
---
### Task 3
**Problem:** Determine the top 10 items in the `orders` table. The most popular will be those that were encountered in orders most often. If a product occurs several times in one order (that is, several units of the product were purchased), then this is also taken into account when calculating. Output the product id and how many times they occurred in orders. Name the new column with the number of product purchases `times_purchased`.

**Solution:** 
``` sql
SELECT unnest(product_ids) as product_id,
       COUNT(*) as times_purchased
FROM   orders
GROUP BY product_id
ORDER BY times_purchased DESC LIMIT 10
```
---
### Task 4 
**Problem:** Designate a 15% discount for products whose price is 50 rubles or higher than the average price for all products, as well as a 10% discount for products whose price is 50 rubles or lower than the average price. Leave the price of other products within the range (average - 50; average + 50) unchanged. When calculating the average price, round it up to two decimal places. Display information about all products with the old and new prices. Name the column with the new price `new_price`. Sort the result first in descending order of the previous price in the `price` column, then in ascending order of the product id.

**Solution:** 
``` sql
WITH avg_price as (SELECT round(avg(price), 2) as price
                   FROM   products)
SELECT product_id,
       name,
       price,
       CASE WHEN price >= (SELECT *
                    FROM   avg_price) + 50 THEN price*0.85 
            WHEN price <= (SELECT *
                    FROM   avg_price) - 50 THEN price*0.9 else price end as new_price
FROM   products
ORDER BY price desc, product_id
```
