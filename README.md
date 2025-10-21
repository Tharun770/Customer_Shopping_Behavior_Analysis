# 🛍️ Customer Shopping Behavior Analysis

## 📘 Project Overview

This project analyzes **customer shopping behavior** using transactional data from **3,900 purchases** across various product categories.
The main objective is to uncover insights into **spending patterns**, **customer segments**, **product preferences**, and **subscription behavior** to guide **strategic business decisions**.

---

## 🎯 Objectives

1. **Data Preparation** – Clean and preprocess transactional data for analysis.
2. **Exploratory Data Analysis (EDA)** – Discover trends and customer behavior patterns.
3. **SQL Business Analysis** – Derive insights using SQL queries in PostgreSQL.
4. **Power BI Dashboard** – Visualize the insights interactively.
5. **Business Recommendations** – Suggest data-driven strategies for business growth.

---

## 🧩 Dataset Summary

| Attribute        | Description                                                                                                                                                                                                                                                               |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Rows**         | 3,900                                                                                                                                                                                                                                                                     |
| **Columns**      | 18                                                                                                                                                                                                                                                                        |
| **Key Features** | Customer demographics (Age, Gender, Location, Subscription Status), Purchase details (Item Purchased, Category, Purchase Amount, Season, Size, Color), Shopping behavior (Discount Applied, Promo Code Used, Previous Purchases, Frequency, Review Rating, Shipping Type) |
| **Missing Data** | 37 missing values in the `review_rating` column                                                                                                                                                                                                                           |

---

## 🐍 Data Cleaning & Preparation (Python)

```python
# Import libraries
import pandas as pd
import numpy as np

# Load dataset
df = pd.read_csv("customer_shopping_behavior.csv")

# Check structure and summary
df.info()
df.describe()

# Handle missing values in review_rating
df['review_rating'] = df.groupby('category')['review_rating'].transform(
    lambda x: x.fillna(x.median())
)

# Rename columns to snake_case
df.columns = df.columns.str.lower().str.replace(' ', '_')

# Feature engineering
bins = [0, 25, 40, 60, 100]
labels = ['Youth', 'Adult', 'Middle-aged', 'Senior']
df['age_group'] = pd.cut(df['age'], bins=bins, labels=labels)

# Create purchase frequency column
df['purchase_frequency_days'] = (df['last_purchase_date'] - df['first_purchase_date']).dt.days

# Drop redundant column
df.drop('promo_code_used', axis=1, inplace=True)

# Connect to PostgreSQL and upload cleaned data
from sqlalchemy import create_engine
engine = create_engine("postgresql://username:password@localhost:5432/customer_db")
df.to_sql("customer_shopping_behavior", engine, index=False, if_exists='replace')
```

---

## 🧮 SQL Business Analysis (PostgreSQL)

Below are key business queries used to derive insights:

### 1. Revenue by Gender

```sql
SELECT gender, SUM(purchase_amount) AS total_revenue
FROM customer_shopping_behavior
GROUP BY gender;
```

### 2. High-Spending Discount Users

```sql
SELECT customer_id, SUM(purchase_amount) AS total_spent
FROM customer_shopping_behavior
WHERE discount_applied = TRUE
GROUP BY customer_id
HAVING SUM(purchase_amount) > (SELECT AVG(purchase_amount) FROM customer_shopping_behavior);
```

### 3. Top 5 Products by Rating

```sql
SELECT item_purchased, ROUND(AVG(review_rating), 2) AS avg_rating
FROM customer_shopping_behavior
GROUP BY item_purchased
ORDER BY avg_rating DESC
LIMIT 5;
```

### 4. Shipping Type Comparison

```sql
SELECT shipping_type, ROUND(AVG(purchase_amount), 2) AS avg_purchase
FROM customer_shopping_behavior
GROUP BY shipping_type;
```

### 5. Subscribers vs Non-Subscribers

```sql
SELECT subscription_status, COUNT(*) AS total_orders, ROUND(AVG(purchase_amount), 2) AS avg_spend
FROM customer_shopping_behavior
GROUP BY subscription_status;
```

### 6. Discount-Dependent Products

```sql
SELECT item_purchased, 
       ROUND(100.0 * SUM(CASE WHEN discount_applied = TRUE THEN 1 ELSE 0 END) / COUNT(*), 2) AS discount_percentage
FROM customer_shopping_behavior
GROUP BY item_purchased
ORDER BY discount_percentage DESC
LIMIT 5;
```

### 7. Customer Segmentation

```sql
SELECT customer_id,
       CASE
           WHEN COUNT(*) = 1 THEN 'New'
           WHEN COUNT(*) BETWEEN 2 AND 5 THEN 'Returning'
           ELSE 'Loyal'
       END AS customer_segment
FROM customer_shopping_behavior
GROUP BY customer_id;
```

### 8. Top 3 Products per Category

```sql
SELECT category, item_purchased, COUNT(*) AS total_orders
FROM customer_shopping_behavior
GROUP BY category, item_purchased
QUALIFY RANK() OVER (PARTITION BY category ORDER BY COUNT(*) DESC) <= 3;
```

### 9. Repeat Buyers & Subscriptions

```sql
SELECT 
    CASE WHEN COUNT(*) > 5 THEN 'Frequent Buyer' ELSE 'Occasional Buyer' END AS buyer_type,
    SUM(CASE WHEN subscription_status = 'Subscribed' THEN 1 ELSE 0 END) AS subscribers
FROM customer_shopping_behavior
GROUP BY customer_id;
```

### 10. Revenue by Age Group

```sql
SELECT age_group, SUM(purchase_amount) AS total_revenue
FROM customer_shopping_behavior
GROUP BY age_group
ORDER BY total_revenue DESC;
```

---

## 📊 Power BI Dashboard

The insights were visualized in **Power BI** using the following visuals:

* **Gender-wise Revenue Distribution**
* **Top-Rated Products**
* **Discount vs Non-Discount Sales**
* **Customer Segmentation**
* **Revenue by Age Group**
* **Shipping Type Comparison**

---

## 💡 Key Findings

* **Female customers** contributed slightly higher total revenue.
* **High-spending customers** often used discounts strategically.
* **Top-rated products** drive repeat purchases.
* **Express shipping** correlates with higher average purchase values.
* **Subscribers** spent more on average than non-subscribers.

---

## 🚀 Business Recommendations

1. **Boost Subscriptions** — Offer exclusive benefits and loyalty points for subscribers.
2. **Customer Retention** — Reward returning buyers to convert them into loyal customers.
3. **Discount Policy Optimization** — Ensure discount usage aligns with profit margins.
4. **Product Positioning** — Promote high-rated and frequently purchased products.
5. **Targeted Marketing** — Focus campaigns on high-revenue age groups and express-shipping users.

---

## ⚙️ Tech Stack

* **Python** – Data cleaning, preprocessing, and feature engineering
* **PostgreSQL** – SQL-based data analysis
* **Power BI** – Dashboard and visual reporting

---

## 🧠 Conclusion

This project demonstrates the **end-to-end analytics workflow** — from cleaning and preparing customer data in Python, performing SQL-driven analysis in PostgreSQL, and visualizing insights through Power BI dashboards.
The results provide actionable business recommendations to **increase revenue, customer loyalty, and marketing efficiency**.

---

## 🗂️ How to Use

1. **Clone this repository**:

   ```bash
   git clone https://github.com/Tharun770/customer-shopping-behavior-analysis.git
   ```

2. **Set up the database**:
   Run the SQL scripts in the `database_setup.sql` file to create and populate your PostgreSQL database.

3. **Run the Python preprocessing script**:
   Execute `data_cleaning.py` to prepare your dataset.

4. **Execute SQL queries**:
   Use the `analysis_queries.sql` file to perform business analysis.

5. **Explore the Power BI dashboard**:
   Open the `.pbix` file to explore visual insights.

---

 

