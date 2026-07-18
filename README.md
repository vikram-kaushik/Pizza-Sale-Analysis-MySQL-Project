# 🍕 Pizza Sales Analysis — MySQL Project

Analysis of a pizza restaurant's sales data using MySQL, covering order volume, revenue, category performance, and time-based trends. The project is structured into three levels — **Basic**, **Intermediate**, and **Advanced** — with each question solved using joins, subqueries, CTEs, and window functions.

**Tool:** MySQL Workbench
**Dataset:** Pizza Place Sales (`orders`, `orders_details`, `pizzas`, `pizza_types`)

---

## 📌 Basic

### 1. Retrieve the total number of orders placed.

```sql
-- Retrieve the total number of orders placed.
Select Count(Order_id) From orders;
```

**Result:** 21,350 total orders

---

### 2. Calculate the total revenue generated from pizza sales.

```sql
-- Calculate the total revenue generated from pizza sales.
SELECT 
    ROUND(SUM(orders_details.QUANTITY * pizzas.price), 2) AS Total_Revenue
FROM
    orders_details
        JOIN
    pizzas ON orders_details.PIZZA_ID = pizzas.pizza_id;
```

**Result:** $817,860.05

---

### 3. Identify the highest-priced pizza.

```sql
-- Identify the highest-priced pizza.
SELECT 
    py.name, p.price
FROM
    pizza_types py
        JOIN
    pizzas p ON py.pizza_type_id = p.pizza_type_id
ORDER BY p.price DESC
LIMIT 1;
```

**Result:** The Greek Pizza — $35.95

---

### 4. Identify the most common pizza size ordered.

```sql
-- Identify the most common pizza size ordered.
SELECT 
    pizzas.size,
    Sum(orders_details.QUANTITY) AS order_count
FROM
    pizzas
        JOIN
    orders_details ON pizzas.pizza_id = orders_details.PIZZA_ID
GROUP BY pizzas.size
ORDER BY order_count DESC;
```

**Result:** Large (L) is the most ordered size, with 18,956 units.

---

### 5. List the top 5 most ordered pizza types along with their quantities.

```sql
-- List the top 5 most ordered pizza types along with their quantities.
SELECT 
    pizza_types.name,
    SUM(orders_details.QUANTITY) AS order_count
FROM
    orders_details
        JOIN
    pizzas ON orders_details.PIZZA_ID = pizzas.pizza_id
        JOIN
    pizza_types ON pizzas.pizza_type_id = pizza_types.pizza_type_id
GROUP BY pizza_types.name
ORDER BY order_count DESC
LIMIT 5;
```

**Result:** The Classic Deluxe Pizza leads with 2,453 orders.

---

## 📌 Intermediate

### 6. Join the necessary tables to find the total quantity of each pizza category ordered.

```sql
-- Join the necessary tables to find the total quantity of each pizza category ordered.
Select pizza_types.category, sum(orders_details.QUANTITY) as quantity
from orders_details join pizzas on orders_details.PIZZA_ID = pizzas.pizza_id
join pizza_types on pizzas.pizza_type_id = pizza_types.pizza_type_id
group by pizza_types.category
order by quantity desc;
```

**Result:** Classic leads with 14,888 units, followed by Supreme, Veggie, and Chicken.

---

### 7. Determine the distribution of orders by hour of the day.

```sql
-- Determine the distribution of orders by hour of the day.
SELECT 
    HOUR(orders.order_time) AS dayhour,
    SUM(orders_details.QUANTITY) AS quantity
FROM
    orders_details
        JOIN
    orders ON orders_details.ORDER_ID = orders.ORDER_ID
GROUP BY dayhour
ORDER BY quantity DESC;
```

**Result:** Orders peak at 12 PM (6,776 pizzas), driven by the lunch and dinner rush hours.

---

### 8. Join relevant tables to find the category-wise distribution of pizzas.

```sql
-- Join relevant tables to find the category-wise distribution of pizzas.
Select category, count(name)
from pizza_types
group by category;
```

**Result:** Veggie and Supreme offer 9 pizza types each, Classic 8, and Chicken 6.

---

### 9. Group the orders by date and calculate the average number of pizzas ordered per day.

```sql
-- Group the orders by date and calculate the average number of pizzas ordered per day.
SELECT 
    ROUND(AVG(quan)) AS Avg_order_per_Day
FROM
    (SELECT 
        orders.ORDER_DATE, SUM(orders_details.QUANTITY) AS quan
    FROM
        orders_details
    JOIN orders ON orders.ORDER_ID = orders_details.ORDER_ID
    GROUP BY orders.ORDER_DATE) AS order_per_day;
```

**Result:** ~138 pizzas sold per day on average.

---

### 10. Determine the top 3 most ordered pizza types based on revenue.

```sql
-- Determine the top 3 most ordered pizza types based on revenue.
SELECT 
    pizza_types.name,
    ROUND(SUM(orders_details.QUANTITY * pizzas.price), 0) AS revenue
FROM
    pizzas
        JOIN
    orders_details ON pizzas.pizza_id = orders_details.PIZZA_ID
        JOIN
    pizza_types ON pizza_types.pizza_type_id = pizzas.pizza_type_id
GROUP BY pizza_types.name
ORDER BY revenue DESC
LIMIT 3;
```

**Result:** The Thai Chicken Pizza ($43,434), Barbecue Chicken ($42,768), and California Chicken ($41,410) top the list.

---

## 📌 Advanced

### 11. Calculate the percentage contribution of each pizza type to total revenue.

```sql
-- Calculate the percentage contribution of each pizza type to total revenue.
With total as ( 
    SELECT 
        SUM(orders_details.QUANTITY * pizzas.price) AS Total_Revenue
    FROM orders_details 
    JOIN pizzas ON orders_details.PIZZA_ID = pizzas.pizza_id
)
SELECT 
    pizza_types.category,
    Round((SUM(orders_details.QUANTITY * pizzas.price) / total.Total_Revenue) * 100, 2) AS revenue
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    orders_details ON orders_details.PIZZA_ID = pizzas.pizza_id
        CROSS JOIN
    total
GROUP BY pizza_types.category, total.Total_Revenue
ORDER BY revenue DESC;
```

**Result:** Classic contributes 26.91% of total revenue, followed by Supreme (25.46%), Chicken (23.96%), and Veggie (23.68%).

---

### 12. Analyze the cumulative revenue generated over time.

```sql
-- Analyze the cumulative revenue generated over time.
Select ORDER_DATE, Round(sum(revenue) Over(order by order_date), 2) as cum_revenue
From (
    Select orders.ORDER_DATE, 
        sum(orders_details.QUANTITY * pizzas.price) as revenue
    from pizzas 
    join orders_details on pizzas.pizza_id = orders_details.PIZZA_ID
    join orders on orders.ORDER_ID = orders_details.ORDER_ID
    group by orders.ORDER_DATE
) as daily_revenue;
```

**Result:** Revenue grows steadily day over day with no major dips, using a running total via `SUM() OVER (ORDER BY order_date)`.

---

### 13. Determine the top 3 most ordered pizza types based on revenue for each pizza category.

```sql
-- Determine the top 3 most ordered pizza types based on revenue for each pizza category.
WITH PizzaRevenue AS (
    SELECT 
        pizza_types.category,
        pizza_types.name,
        Round(SUM(orders_details.QUANTITY * pizzas.price)) AS Revenue
    FROM pizzas
    JOIN orders_details
        ON pizzas.pizza_id = orders_details.PIZZA_ID
    JOIN pizza_types
        ON pizza_types.pizza_type_id = pizzas.pizza_type_id
    GROUP BY pizza_types.category, pizza_types.name
),
RankedPizza AS (
    SELECT
        category,
        name,
        Revenue,
        DENSE_RANK() OVER (PARTITION BY category ORDER BY Revenue DESC) AS drnk
    FROM PizzaRevenue
)
SELECT
    category,
    name,
    Revenue,
    drnk
FROM RankedPizza
WHERE drnk <= 3;
```

**Result:** Ranks each pizza within its own category using `DENSE_RANK() OVER (PARTITION BY category ...)`, surfacing the top 3 earners per category — e.g. Thai Chicken leads Chicken, Classic Deluxe leads Classic.

---

## 🛠️ Concepts Used

- Joins (`INNER JOIN` across `orders`, `orders_details`, `pizzas`, `pizza_types`)
- Aggregate functions (`SUM`, `COUNT`, `AVG`, `ROUND`)
- Subqueries
- Common Table Expressions (`WITH`)
- Window functions (`SUM() OVER`, `DENSE_RANK() OVER PARTITION BY`)

## 👤 Author

**Vikram Kaushik**
[LinkedIn](https://linkedin.com/in/vikramkaushik007) · [GitHub](https://github.com/vikram-kaushik)
