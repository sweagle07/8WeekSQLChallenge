Written with [StackEdit](https://stackedit.io/).
 1.  What is the total amount each customer spent at the restaurant?
	 SELECT 
	 SUM(m.price) total_spent, s.customer_id 
	 FROM menu m 
	 JOIN sales s ON m.product_id = s.product_id 
	 GROUP BY s.customer_id;

2.  How many days has each customer visited the restaurant?
	SELECT
	  COUNT(DISTINCT order_date),
	  s.customer_id
	FROM menu m
	  JOIN sales s ON m.product_id = s.product_id
	 GROUP BY
	  s.customer_id;
	  
3.  What was the first item from the menu purchased by each customer?
	SELECT 
	  customer_id,
	  product_id,
	  order_date,
	  DENSE_RANK() OVER(ORDER BY order_date) AS first_order
	FROM sales
	GROUP BY order_date, product_id, customer_id
	ORDER BY order_date;
4.  (a) What is the most purchased item on the menu and (b) how many times was it purchased by all customers?
	(a) SELECT 
	  COUNT(s.product_id) as total,
	  m.product_name
	FROM menu m
	JOIN sales s ON s.product_id = m.product_id
	GROUP BY m.product_name
	ORDER BY total DESC
	LIMIT 1;
	(b) SELECT 
	  COUNT(s.product_id) as total,
	  s.customer_id,
	  m.product_name
	FROM menu m
	JOIN sales s ON s.product_id = m.product_id
	WHERE m.product_name = 'ramen'
	GROUP BY m.product_name, s.customer_id
	ORDER BY total DESC
5.  Which item was the most popular for each customer?
	SELECT
	  customer_id,
	  product_id,
	  COUNT (product_id) as total
	FROM sales
	GROUP BY customer_id, product_id
	ORDER BY total DESC;
6.  Which item was purchased first by the customer after they became a member?
	SELECT 
	  s.customer_id,
	  mem.join_date,
	  s.order_date,
	  me.product_name
	FROM sales s
	JOIN members mem ON mem.customer_id = s.customer_id
	JOIN menu me ON me.product_id = s.product_id
	WHERE s.order_date > mem.join_date
	ORDER BY s.order_date ASC;
7.  Which item was purchased just before the customer became a member?
		Answer same as above, however switch the > to < and change ORDER BY 							sort to DESC 
8.  What is the total items and amount spent for each member before they became a member? **(QUESTION??** on solution - customer C is not a member, so doesn't show up in the result set. Is that a problem?
	SELECT 
	  sales.customer_id,
	  COUNT(sales.product_id),
	  SUM(menu.price)
	FROM sales
	JOIN menu ON menu.product_id = sales.product_id
	JOIN members ON members.customer_id = sales.customer_id
	WHERE sales.order_date < members.join_date
	GROUP BY sales.customer_id;
9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
	SELECT
	  sales.customer_id,
	  SUM (CASE WHEN menu.product_name = 'sushi' THEN menu.price * 20
	    ELSE menu.price * 10
	    END) AS points_multiplier,
	  SUM(menu.price)*10 AS points
	FROM sales
	JOIN menu ON menu.product_id = sales.product_id
	GROUP BY sales.customer_id;
10.  In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

BONUS Questions:
Re-create a reference table joining the separate tables:
	SELECT
	  sales.customer_id,
	  sales.order_date,
	--  members.join_date,
	  menu.product_name,
	  menu.price,
	  CASE WHEN (sales.customer_id = members.customer_id AND members.join_date <= sales.order_date) THEN 'Y'
	       ELSE 'N'
	       END AS member
	FROM sales
	JOIN menu ON menu.product_id = sales.product_id
	LEFT JOIN members ON members.customer_id = sales.customer_id
	ORDER BY sales.customer_id, sales.order_date;

Danny also requires further information about the  `ranking`  of customer products, but he purposely does not need the ranking for non-member purchases so he expects null  `ranking`  values for the records when customers are not yet part of the loyalty program.
	SELECT
	  sales.customer_id,
	  sales.order_date,
	--  members.join_date,
	  menu.product_name,
	  menu.price,
	  CASE WHEN (sales.customer_id = members.customer_id AND members.join_date <= sales.order_date) THEN 'Y'
	       ELSE 'N'
	       END AS member,
	  RANK() OVER() AS ranking -- **Need to practice adding in the rank with proper conditions**
	FROM sales
	JOIN menu ON menu.product_id = sales.product_id
	LEFT JOIN members ON members.customer_id = sales.customer_id
	ORDER BY sales.customer_id, sales.order_date;
