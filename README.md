### Hi there 👋

8 Week SQL Challenge - Case Study #1: Danny's Diner

Objective:
Use data to answer questions about customers, such as money spent, popular products, and frequency of visits.


Database Relationship Diagram:


Case Study Questions:
1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
What is the most purchased item on the menu and how many times was it purchased by all customers?
Which item was the most popular for each customer?
Which item was purchased first by the customer after they became a member?
Which item was purchased just before the customer became a member?
What is the total items and amount spent for each member before they became a member?
If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?


## Solutions

### 1. What is the total amount each customer spent at the restaurant?
```sql
SELECT sales.customer_id, SUM(price) AS "TotalSales"
FROM dannys_diner.sales
JOIN dannys_diner.menu ON sales.product_id=menu.product_id
GROUP BY customer_id
ORDER BY customer_id;
```
#### Reasoning
-  Find total amount spent using the **SUM** function
-  Use **JOIN** function to combine two seperate tables that contain the information necessary to complete the query (```product_id``` and ```price```)
-  To sort by each customer, utilize the **GROUP BY** function
-  The **ORDER BY** function is used to further sort the query by ```customer_id``` 

#### Answer:
| customer_id | TotalSales  |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

From the query above we see that customer A spent $76,  customer B spent $74, and customer C spent $36.

***

### 2. How many days has each customer visited the restaurant?

```sql
SELECT sales.customer_id, COUNT(DISTINCT(order_date)) as "TimesVisited"
FROM dannys_diner.sales
GROUP BY customer_id;
```

#### Reasoning
- All information required to answer the question is in the Sales table (```customer_id``` and ```order-date```)
- To figure out number of days visited, I used the **DISTINCT** and **COUNT** functions. This filters out multiple orders on the same day, and converts each order date into a numerical  value that represents times visited
- Filter the results by ```customer_id``` to see the number of times each customer visited

#### Answer:
| customer_id | TimesVisited |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

From the query above we see that customer A visited 4 times, customer B visited 6 times, and customer C visited 2 times.

***

### 3. What was the first item from the menu purchased by each customer?

````sql
WITH order_ranked AS
(
	SELECT customer_id, order_date, product_name
	DENSE_RANK() OVER(PARTITION BY sales.customer_id order by sales.order_date)
	as rank
	FROM dannys_diner.sales
	JOIN dannys_diner.menu
  	ON sales.product_id=menu.product_id
)

SELECT customer_id, product_name
FROM order_ranked
WHERE rank = 1
GROUP BY customer_id, product_name;
````

#### Reasoning
- Using a **MIN** function on ```order_date``` combined with the ```product_name``` would not work as it would provide me the the earliest time each customer ordered each menu item, rather than the first item they had ordered
- Creating a CTE with a **DENSE_RANK** function, a **PARTITION BY** function, and a **ORDER_BY** function provides each product order by the customer to be assigned a rank
- **DENSE_RANK** is used instead of **RANK** as the data does not indicate which product was ordered earlier on the same date
- Using the CTE, I then can filter the first product ordered by each customer based on the rank provided

#### Answer:
| customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |

From the above query, customer A's first orders are curry and sushi, customer B's first order is curry, and customer C's first order is ramen.

***

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

````sql
SELECT TOP 1 (COUNT(s.product_id)) AS most_purchased, product_name
FROM dbo.sales AS s
JOIN dbo.menu AS m
   ON s.product_id = m.product_id
GROUP BY s.product_id, product_name
ORDER BY most_purchased DESC;
````

#### Steps:
- **COUNT** number of ```product_id``` and **ORDER BY** ```most_purchased``` by descending order. 
- Then, use **TOP 1** to filter highest number of purchased item.

#### Answer:
| most_purchased | product_name | 
| ----------- | ----------- |
| 8       | ramen |


- Most purchased item on the menu is ramen which is 8 times. Yummy!

***

### 5. Which item was the most popular for each customer?

````sql
WITH fav_item_cte AS
(
   SELECT s.customer_id, m.product_name, COUNT(m.product_id) AS order_count,
      DENSE_RANK() OVER(PARTITION BY s.customer_id
      ORDER BY COUNT(s.customer_id) DESC) AS rank
   FROM dbo.menu AS m
   JOIN dbo.sales AS s
      ON m.product_id = s.product_id
   GROUP BY s.customer_id, m.product_name
)

SELECT customer_id, product_name, order_count
FROM fav_item_cte 
WHERE rank = 1;
````

#### Steps:
- Create a ```fav_item_cte``` and use **DENSE_RANK** to ```rank``` the ```order_count``` for each product by descending order for each customer.
- Generate results where product ```rank = 1``` only as the most popular product for each customer.

#### Answer:
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

- Customer A and C's favourite item is ramen.
- Customer B enjoys all items on the menu. He/she is a true foodie, sounds like me!

***

### 6. Which item was purchased first by the customer after they became a member?

````sql
WITH member_sales_cte AS 
(
   SELECT s.customer_id, m.join_date, s.order_date, s.product_id,
      DENSE_RANK() OVER(PARTITION BY s.customer_id
      ORDER BY s.order_date) AS rank
   FROM sales AS s
   JOIN members AS m
      ON s.customer_id = m.customer_id
   WHERE s.order_date >= m.join_date
)

SELECT s.customer_id, s.order_date, m2.product_name 
FROM member_sales_cte AS s
JOIN menu AS m2
   ON s.product_id = m2.product_id
WHERE rank = 1;
````

#### Steps:
- Create ```member_sales_cte``` by using **windows function** and partitioning ```customer_id``` by ascending ```order_date```. Then, filter ```order_date``` to be on or after ```join_date```.
- Then, filter table by ```rank = 1``` to show 1st item purchased by each customer.

#### Answer:
| customer_id | order_date  | product_name |
| ----------- | ---------- |----------  |
| A           | 2021-01-07 | curry        |
| B           | 2021-01-11 | sushi        |

- Customer A's first order as member is curry.
- Customer B's first order as member is sushi.

***

### 7. Which item was purchased just before the customer became a member?

````sql
WITH prior_member_purchased_cte AS 
(
   SELECT s.customer_id, m.join_date, s.order_date, s.product_id,
         DENSE_RANK() OVER(PARTITION BY s.customer_id
         ORDER BY s.order_date DESC) AS rank
   FROM sales AS s
   JOIN members AS m
      ON s.customer_id = m.customer_id
   WHERE s.order_date < m.join_date
)

SELECT s.customer_id, s.order_date, m2.product_name 
FROM prior_member_purchased_cte AS s
JOIN menu AS m2
   ON s.product_id = m2.product_id
WHERE rank = 1;
````

#### Steps:
- Create a ```prior_member_purchased_cte``` to create new column ```rank``` by using **Windows function** and partitioning ```customer_id``` by descending ```order_date``` to find out the last ```order_date``` before customer becomes a member.
- Filter ```order_date``` before ```join_date```.

#### Answer:
| customer_id | order_date  | product_name |
| ----------- | ---------- |----------  |
| A           | 2021-01-01 |  sushi        |
| A           | 2021-01-01 |  curry        |
| B           | 2021-01-04 |  sushi        |

- Customer A’s last order before becoming a member is sushi and curry.
- Whereas for Customer B, it's sushi. That must have been a real good sushi!

***

### 8. What is the total items and amount spent for each member before they became a member?

````sql
SELECT s.customer_id, COUNT(DISTINCT s.product_id) AS unique_menu_item, 
   SUM(mm.price) AS total_sales
FROM sales AS s
JOIN members AS m
   ON s.customer_id = m.customer_id
JOIN menu AS mm
   ON s.product_id = mm.product_id
WHERE s.order_date < m.join_date
GROUP BY s.customer_id;

````

#### Steps:
- Filter ```order_date``` before ```join_date``` and perform a **COUNT** **DISTINCT** on ```product_id``` and **SUM** the ```total spent``` before becoming member.

#### Answer:
| customer_id | unique_menu_item | total_sales |
| ----------- | ---------- |----------  |
| A           | 2 |  25       |
| B           | 2 |  40       |

Before becoming members,
- Customer A spent $ 25 on 2 items.
- Customer B spent $40 on 2 items.

***

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?

````sql
WITH price_points AS
(
   SELECT *, 
      CASE
         WHEN product_id = 1 THEN price * 20
         ELSE price * 10
      END AS points
   FROM menu
)

SELECT s.customer_id, SUM(p.points) AS total_points
FROM price_points_cte AS p
JOIN sales AS s
   ON p.product_id = s.product_id
GROUP BY s.customer_id
````

#### Steps:
Let’s breakdown the question.
- Each $1 spent = 10 points.
- But, sushi (product_id 1) gets 2x points, meaning each $1 spent = 20 points
So, we use CASE WHEN to create conditional statements
- If product_id = 1, then every $1 price multiply by 20 points
- All other product_id that is not 1, multiply $1 by 10 points
Using ```price_points```, **SUM** the ```points```.

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

- Total points for Customer A is 860.
- Total points for Customer B is 940.
- Total points for Customer C is 360.

***

### 10. 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?

````sql
WITH dates_cte AS 
(
   SELECT *, 
      DATEADD(DAY, 6, join_date) AS valid_date, 
      EOMONTH('2021-01-31') AS last_date
   FROM members AS m
)

SELECT d.customer_id, s.order_date, d.join_date, d.valid_date, d.last_date, m.product_name, m.price,
   SUM(CASE
      WHEN m.product_name = 'sushi' THEN 2 * 10 * m.price
      WHEN s.order_date BETWEEN d.join_date AND d.valid_date THEN 2 * 10 * m.price
      ELSE 10 * m.price
      END) AS points
FROM dates_cte AS d
JOIN sales AS s
   ON d.customer_id = s.customer_id
JOIN menu AS m
   ON s.product_id = m.product_id
WHERE s.order_date < d.last_date
GROUP BY d.customer_id, s.order_date, d.join_date, d.valid_date, d.last_date, m.product_name, m.price
````

#### Steps:
- In ```dates_cte```, find out customer’s ```valid_date``` (which is 6 days after ```join_date``` and inclusive of ```join_date```) and ```last_day``` of Jan 2021 (which is ‘2021–01–31’).

Our assumptions are:
- On Day -X to Day 1 (customer becomes member on Day 1 ```join_date```), each $1 spent is 10 points and for sushi, each $1 spent is 20 points.
- On Day 1 ```join_date``` to Day 7 ```valid_date```, each $1 spent for all items is 20 points.
- On Day 8 to ```last_day``` of Jan 2021, each $1 spent is 10 points and sushi is 2x points.

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 1370 |
| B           | 820 |

- Total points for Customer A is 1,370.
- Total points for Customer B is 820.

***

## BONUS QUESTIONS

### Join All The Things - Recreate the table with: customer_id, order_date, product_name, price, member (Y/N)

````sql
SELECT s.customer_id, s.order_date, m.product_name, m.price,
   CASE
      WHEN mm.join_date > s.order_date THEN 'N'
      WHEN mm.join_date <= s.order_date THEN 'Y'
      ELSE 'N'
      END AS member
FROM sales AS s
LEFT JOIN menu AS m
   ON s.product_id = m.product_id
LEFT JOIN members AS mm
   ON s.customer_id = mm.customer_id;
 ````
 
#### Answer: 
| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | -------------| ----- | ------ |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

***

### Rank All The Things - Danny also requires further information about the ```ranking``` of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ```ranking``` values for the records when customers are not yet part of the loyalty program.

````sql
WITH summary_cte AS 
(
   SELECT s.customer_id, s.order_date, m.product_name, m.price,
      CASE
      WHEN mm.join_date > s.order_date THEN 'N'
      WHEN mm.join_date <= s.order_date THEN 'Y'
      ELSE 'N' END AS member
   FROM sales AS s
   LEFT JOIN menu AS m
      ON s.product_id = m.product_id
   LEFT JOIN members AS mm
      ON s.customer_id = mm.customer_id
)

SELECT *, CASE
   WHEN member = 'N' then NULL
   ELSE
      RANK () OVER(PARTITION BY customer_id, member
      ORDER BY order_date) END AS ranking
FROM summary_cte;
````

#### Answer: 
| customer_id | order_date | product_name | price | member | ranking | 
| ----------- | ---------- | -------------| ----- | ------ |-------- |
| A           | 2021-01-01 | sushi        | 10    | N      | NULL
| A           | 2021-01-01 | curry        | 15    | N      | NULL
| A           | 2021-01-07 | curry        | 15    | Y      | 1
| A           | 2021-01-10 | ramen        | 12    | Y      | 2
| A           | 2021-01-11 | ramen        | 12    | Y      | 3
| A           | 2021-01-11 | ramen        | 12    | Y      | 3
| B           | 2021-01-01 | curry        | 15    | N      | NULL
| B           | 2021-01-02 | curry        | 15    | N      | NULL
| B           | 2021-01-04 | sushi        | 10    | N      | NULL
| B           | 2021-01-11 | sushi        | 10    | Y      | 1
| B           | 2021-01-16 | ramen        | 12    | Y      | 2
| B           | 2021-02-01 | ramen        | 12    | Y      | 3
| C           | 2021-01-01 | ramen        | 12    | N      | NULL
| C           | 2021-01-01 | ramen        | 12    | N      | NULL
| C           | 2021-01-07 | ramen        | 12    | N      | NULL


***



