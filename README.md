# 🛍️ Customer Behavior Analysis — Python, SQL & Power BI

An end-to-end **Customer Behavior Analysis** project that uses **Python for data cleaning and preprocessing, PostgreSQL for SQL-based analysis, and Power BI for interactive dashboarding**.

The project analyzes customer purchasing patterns, spending behavior, subscriptions, discounts, product performance, customer segments, and age-group trends.

---

## 📊 Dashboard Preview

<img width="1307" height="722" alt="image" src="https://github.com/user-attachments/assets/0e183c3c-3e80-46ac-875c-e1f91c6dad5d" />


The Power BI dashboard provides an interactive view of customer behavior with filters for:

* Subscription Status
* Gender
* Category
* Shipping Type

### Key KPIs

* **3.9K** Customers
* **$59.76** Average Purchase Amount
* **3.75** Average Review Rating
* **73%** Non-subscribed Customers
* **27%** Subscribed Customers

### Visualizations

* Customer distribution by subscription status
* Revenue by product category
* Sales by product category
* Revenue by age group
* Sales by age group
* Item purchases by size

---

## 🎯 Project Objectives

The main objectives of this project are to:

* Clean and preprocess raw customer shopping data
* Handle missing values and improve data quality
* Create meaningful derived features
* Store the processed dataset in PostgreSQL
* Perform business-oriented analysis using SQL
* Identify customer purchasing patterns
* Analyze subscription and discount behavior
* Segment customers based on previous purchases
* Build an interactive Power BI dashboard
* Present insights in a visually understandable format

---

## 🗂️ Dataset

The dataset contains **3,900 customer records** and initially includes **18 columns** describing customer demographics, purchases, product information, and purchasing behavior.

### Main Columns

| Column                   | Description                    |
| ------------------------ | ------------------------------ |
| `customer_id`            | Unique customer identifier     |
| `age`                    | Customer age                   |
| `gender`                 | Customer gender                |
| `item_purchased`         | Product purchased              |
| `category`               | Product category               |
| `purchase_amount`        | Purchase amount in USD         |
| `location`               | Customer location              |
| `size`                   | Product size                   |
| `color`                  | Product color                  |
| `season`                 | Purchase season                |
| `review_rating`          | Product review rating          |
| `subscription_status`    | Customer subscription status   |
| `shipping_type`          | Shipping method                |
| `discount_applied`       | Whether a discount was applied |
| `previous_purchases`     | Number of previous purchases   |
| `payment_method`         | Payment method                 |
| `frequency_of_purchases` | Customer purchase frequency    |

---

# 🐍 1. Data Cleaning & Preprocessing — Python

Python and Pandas were used to inspect, clean, transform, and prepare the dataset before loading it into PostgreSQL.

### Loading the Dataset

```python
import pandas as pd

df = pd.read_csv('customer_shopping_behavior.csv')
df.head()
```

### Dataset Inspection

The dataset was examined using:

```python
df.info()
df.describe(include='all')
```

The original dataset contained:

* **3,900 rows**
* **18 columns**
* Numerical and categorical features

### Missing Value Handling

The `Review Rating` column contained **37 missing values**.

Instead of removing those records, missing ratings were imputed using the **median rating of the corresponding product category**.

```python
df['Review Rating'] = df.groupby('Category')['Review Rating'].transform(
    lambda x: x.fillna(x.median())
)
```

This preserved all 3,900 customer records.

### Column Standardization

Column names were converted to lowercase and snake_case for easier SQL querying and documentation.

Example:

```text
Purchase Amount (USD)
```

was converted to:

```text
purchase_amount
```

### Feature Engineering

#### Age Group

Customers were divided into four age groups using quartile-based binning:

* Young Adult
* Adult
* Middle-aged
* Senior

```python
labels = ['Young Adult', 'Adult', 'Middle-aged', 'Senior']

df['age_group'] = pd.qcut(
    df['age'],
    q=4,
    labels=labels
)
```

#### Purchase Frequency in Days

Purchase frequency categories were converted into numerical day intervals.

```python
frequency_mapping = {
    'Fortnightly': 14,
    'Weekly': 7,
    'Monthly': 30,
    'Quarterly': 90,
    'Bi-Weekly': 14,
    'Annually': 365,
    'Every 3 Months': 90
}

df['purchase_frequency_days'] = (
    df['frequency_of_purchases'].map(frequency_mapping)
)
```

### Redundant Column Removal

`promo_code_used` was removed after verifying that its values were identical to `discount_applied`.

```python
(df['discount_applied'] == df['promo_code_used']).all()
```

Since the result was `True`, the redundant column was dropped.

---

# 🐘 2. PostgreSQL & SQL Analysis

The cleaned dataset was loaded into **PostgreSQL** using SQLAlchemy and `psycopg2`.

### Database Connection

```python
from sqlalchemy import create_engine

username = "postgres"
password = "your_password"
host = "localhost"
port = "5433"
database = "customer_behavior_analysis"

engine = create_engine(
    f"postgresql+psycopg2://{username}:{password}@{host}:{port}/{database}"
)
```

### Loading Data into PostgreSQL

```python
table_name = "customer"

df.to_sql(
    table_name,
    engine,
    if_exists="replace",
    index=False
)
```

The processed data was stored in a PostgreSQL table named:

```text
customer
```

---

## 🔎 SQL Business Analysis

The project uses SQL to answer **10 business questions**.

### 1. Revenue by Gender

Compared total revenue generated by male and female customers.

### 2. Discounted Purchases Above Average

Identified customers who used discounts but still spent at least the average purchase amount.

### 3. Top 5 Products by Average Rating

Used aggregation and sorting to identify the five highest-rated products.

### 4. Standard vs Express Shipping

Compared the average purchase amount between Standard and Express shipping customers.

### 5. Subscriber vs Non-Subscriber Spending

Compared:

* Number of customers
* Average spending
* Total revenue

between subscribers and non-subscribers.

### 6. Products with Highest Discount Rate

Calculated the percentage of purchases where discounts were applied.

### 7. Customer Segmentation

Customers were classified into:

* **New**
* **Returning**
* **Loyal**

based on their previous purchase count.

### 8. Top 3 Products Within Each Category

Used the `ROW_NUMBER()` window function to identify the three most purchased products within each category.

### 9. Repeat Buyers and Subscription

Analyzed whether customers with more than five previous purchases were also more likely to subscribe.

### 10. Revenue by Age Group

Compared total revenue generated by:

* Young Adults
* Adults
* Middle-aged customers
* Seniors

---

# 📈 3. Power BI Dashboard

The processed PostgreSQL data was connected to **Microsoft Power BI** to create an interactive dashboard.

### Dashboard Components

#### KPI Cards

* Number of Customers
* Average Purchase Amount
* Average Review Rating

#### Customer Analysis

* % of Customers by Subscription Status
* Revenue by Age Group
* Sales by Age Group

#### Product Analysis

* Revenue by Category
* Sales by Category
* % of Items Purchased by Size

#### Interactive Filters

Users can dynamically filter the dashboard by:

* Subscription Status
* Gender
* Category
* Shipping Type

This allows users to explore customer behavior across different demographic and purchasing segments.

---

# 🔄 Project Workflow

```text
Raw CSV Dataset
       ↓
Python / Pandas
       ↓
Data Cleaning
       ↓
Missing Value Treatment
       ↓
Feature Engineering
       ↓
PostgreSQL
       ↓
SQL Business Analysis
       ↓
Power BI
       ↓
Interactive Customer Behavior Dashboard
```

---

# 🛠️ Tech Stack

<p align="left">
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white"/>
  <img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white"/>
  <img src="https://img.shields.io/badge/SQL-4479A1?style=for-the-badge&logo=postgresql&logoColor=white"/>
  <img src="https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black"/>
  <img src="https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white"/>
  <img src="https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white"/>
</p>

---


# 💡 Key Skills Demonstrated

* Data Cleaning & Preprocessing
* Exploratory Data Analysis
* Feature Engineering
* Missing Value Imputation
* Python & Pandas
* PostgreSQL
* SQL Aggregations
* Subqueries
* CTEs
* Window Functions
* CASE Statements
* Business Analytics
* Data Visualization
* Power BI Dashboard Development
* Data Storytelling

---

## 🚀 Future Improvements

* Add customer lifetime value analysis
* Perform cohort analysis
* Analyze customer retention
* Add geographical sales analysis
* Build predictive models for customer purchase behavior
* Add advanced Power BI measures using DAX

---

## 👩‍💻 Author

**Niya Benny**

Data Science Student | Python | SQL | Power BI | Machine Learning

[GitHub](https://github.com/niya-benny)
