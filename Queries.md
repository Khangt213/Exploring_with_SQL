# Some queries to explore the Database

To make sure just to the the magist Database use:

```
use magist;
```

### How many orders are there in the dataset?

```
SELECT 
  COUNT(*) 
FROM 
  orders

```

### Are orders actually delivered?
```
SELECT 
  COUNT(*) 
FROM 
  orders 
WHERE 
  order_status = "delivered"

```

### Is the number of orders increasing?
```
SELECT 
  YEAR(order_purchase_timestamp) AS year, 
  MONTH(order_purchase_timestamp) AS month, 
  COUNT(*) AS total 
FROM 
  orders 
GROUP BY 
  year, 
  month 
ORDER BY 
  year, 
  month

```

### How many products are there in the products table?
```
SELECT 
  COUNT(
    DISTINCT(product_id)
  ) 
FROM 
  products

```

### Which are the categories with most products?
```
SELECT 
  product_category_name_translation.product_category_name_english, 
  COUNT(products.product_category_name) 
FROM 
  products 
  JOIN product_category_name_translation ON products.product_category_name = product_category_name_translation.product_category_name 
GROUP BY 
  product_category_name_english 
ORDER BY 
  COUNT(products.product_category_name) DESC
```

### What’s the price for the most expensive and cheapest products?
```
SELECT 
  MAX(price), 
  MIN(price) 
FROM 
  order_items;

```
# Answering some Business questions

### What’s the average time between the order being placed and the product being delivered?
```
SELECT 
  avg(
    TIMESTAMPDIFF(
      day, order_purchase_timestamp, order_delivered_customer_date
    )
  ) AS delivered_time 
FROM 
  orders;
 ``` 
 

### What's the average delay time for big products?
```
SELECT 
  order_items.product_id, 
  products.product_height_cm, 
  products.product_length_cm, 
  products.product_width_cm, 
  avg(
    timestampdiff(
      day, order_estimated_delivery_date, 
      order_delivered_customer_date
    )
  ) AS delay 
FROM 
  orders 
  JOIN order_items ON orders.order_id = order_items.order_id 
  JOIN products ON order_items.product_id = products.product_id 
WHERE 
  timestampdiff(
    day, order_estimated_delivery_date, 
    order_delivered_customer_date
  ) > 0 
  AND (
    product_height_cm * product_length_cm * product_width_cm
  ) > 27000 
  AND product_weight_g > 2000;
```
### What's the average price of high-end-tech products?
```
SELECT 
  count(order_items.product_id), 
  avg(price) 
FROM 
  products 
  JOIN order_items ON products.product_id = order_items.product_id 
  JOIN product_category_name_translation ON products.product_category_name = product_category_name_translation.product_category_name 
  JOIN orders ON order_items.order_id = orders.order_id 
  JOIN customers ON orders.customer_id = customers.customer_id 
  JOIN geo ON customers.customer_zip_code_prefix = geo.zip_code_prefix 
WHERE 
  product_category_name_translation.product_category_name_english in (
    'audio', 'cds_dvds_musicals', 'air_conditioning', 
    'consoles_games', 'cool_stuff', 
    'electronics', 'books_technical', 
    'music', 'pc_gamer', 'computers', 
    'telephony'
  ) 
  AND price > 1000
