# E-commerce Sales and Inventory Dashboard

## Overview
This project is an interactive Power BI dashboard designed to analyze e-commerce sales, inventory, and customer feedback for Blinkit (India’s Last Minute App). It provides insights into sales trends, top products, customer segments, and inventory performance, with a user-friendly interface designed in Figma.

## Features
- **Sales Analysis:** Tracks sales performance (YTD, last 6/9 months) with a growth rate of -2.79% in 2025 compared to 2024.
- **Inventory Management:** Monitors stock availability (81.14%) and damaged stock (16.66%).
- **Customer Insights:** Analyzes customer segments, top areas (e.g., Ghaziabad), and feedback ratings.
- **Data Visualizations:** Includes bar charts, line graphs, donut charts, and dynamic filters for interactive analysis.
- **SQL Data Integration:** Created a SQL View (`ecommerce_database`) to preprocess and integrate data from multiple tables.
- **UI/UX Design:** Designed the dashboard layout and navigation using Figma, ensuring a consistent and intuitive user experience.
- **DAX Calculations:** Utilizes DAX for creating measures and dynamic titles.

## Screenshots
### Dashboard Pages
![Sales Overview](https://github.com/Mahdy-Wagdy/Ecommerce-PowerBI-Dashboard/blob/main/Sales%20Overview.png) 
![Customer Feedback](screenshots/customer_feedback.png)  
![Inventory Analysis](screenshots/inventory.png)  

### Figma Design
![Figma Layout](screenshots/figma_design.png)

## Tools Used
- **Power BI:** For data visualization and dashboard creation.
- **SQL:** For data preprocessing and creating views.
- **Figma:** For designing the dashboard layout and UI.
- **DAX (Data Analysis Expressions):** For creating calculated measures.
- **Data Sources:** Excel/CSV (e-commerce dataset).

## SQL View
The following SQL View (`ecommerce_database`) was created to integrate data from multiple tables and calculate key metrics:

```sql
CREATE 
    ALGORITHM = UNDEFINED 
    DEFINER = `root`@`localhost` 
    SQL SECURITY DEFINER
VIEW `ecommerce_database` AS
    SELECT 
        orders.order_id,
        customer.customer_id,
        orders.delivery_partner_id,
        order_item.product_id,
        customer_feedbacks.feedback_id,
        orders.order_date AS order_datetime,
        customer.area,
        customer.customer_name,
        customer.customer_segment,
        products.product_name,
        products.category,
        products.price,
        order_item.quantity,
        ROUND(products.price * order_item.quantity, 2) AS value,
        orders.payment_method,
        delivery.promised_time,
        delivery.actual_time,
        delivery.delivery_time_minutes,
        COALESCE(delivery.reasons_if_delayed, 'No delay') AS reasons_if_delayed,
        CASE 
            WHEN delivery.actual_time > delivery.promised_time THEN 'Delayed'
            ELSE 'On Time'
        END AS delivery_status,
        customer_feedbacks.rating,
        customer_feedbacks.feedback_category,
        customer_feedbacks.feedback_text,
        customer_feedbacks.sentiment AS feedback_segment,
        CAST(orders.order_date AS DATE) AS date,
        rating.Emoji,
        rating.Star,
        category.Img,
        CASE 
            WHEN _Customer_Meaures.Period_Customer_PY IS NOT NULL 
                 AND _Customer_Meaures.Period_Customer_PY <> 0 
            THEN (_Customer_Meaures.Period_Customer_CY - _Customer_Meaures.Period_Customer_PY) / _Customer_Meaures.Period_Customer_PY
            ELSE NULL
        END AS Customer_Growth
    FROM orders
    JOIN customer ON orders.customer_id = customer.customer_id
    JOIN order_item ON order_item.order_id = orders.order_id
    JOIN products ON order_item.product_id = products.product_id
    JOIN customer_feedbacks ON customer_feedbacks.customer_id = orders.customer_id 
                             AND customer_feedbacks.order_id = orders.order_id
    JOIN delivery ON delivery.delivery_partner_id = orders.delivery_partner_id 
                  AND delivery.order_id = orders.order_id
    JOIN rating ON customer_feedbacks.rating = rating.Rating
    JOIN category ON products.category = category.category
    JOIN _Customer_Meaures ON _Customer_Meaures.customer_id = customer.customer_id
    WHERE orders.order_date >= '2024-01-01' AND orders.order_date <= '2025-12-31';
