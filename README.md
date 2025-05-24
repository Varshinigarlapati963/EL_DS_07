# EL_DS_07
# Sales Data Analysis with SQLite and Python in Google Colab

This project generates a sample sales dataset using SQLite, performs analysis using SQL queries and pandas, and visualizes the results with matplotlib. It is designed to run smoothly on Google Colab, saving the database to Google Drive to persist data between sessions.



## Features

- Create and populate an SQLite database (`sales_data.db`) with random sales data.
- Perform SQL queries to aggregate sales by product and by month.
- Visualize total quantity sold and revenue by product and by month using bar charts.
- Save and load the database from Google Drive to prevent data loss in Colab.


 How to Run
 1. Mount Google Drive in Colab

```python
from google.colab import drive
drive.mount('/content/drive')
```
2. Set database path
Save and load your SQLite database in a folder inside your Drive, for example:
```python
import os

# Create folder in Drive if it doesn't exist
os.makedirs('/content/drive/MyDrive/ColabData', exist_ok=True)

# Define the database file path in Drive
db_path = '/content/drive/MyDrive/ColabData/sales_data.db'
```
3. Create and populate database (run only once):
```python
import sqlite3
import random
import datetime

conn = sqlite3.connect(db_path)
cursor = conn.cursor()

cursor.execute("DROP TABLE IF EXISTS sales")
cursor.execute("""
CREATE TABLE sales (
    id INTEGER PRIMARY KEY,
    product TEXT,
    quantity INTEGER,
    price REAL,
    sale_date TEXT
)
""")

products = {
    'Apple': 0.5,
    'Banana': 0.2,
    'Orange': 0.3,
    'Mango': 1.0,
    'Grapes': 0.8
}

for _ in range(1000):
    product = random.choice(list(products.keys()))
    price = products[product]
    quantity = random.randint(1, 10)
    sale_date = datetime.date(2023, random.randint(1, 12), random.randint(1, 28)).isoformat()

    cursor.execute("INSERT INTO sales (product, quantity, price, sale_date) VALUES (?, ?, ?, ?)",
                   (product, quantity, price, sale_date))

conn.commit()
conn.close()
```
4. Load data and perform analysis:
```python
import pandas as pd
import sqlite3

conn = sqlite3.connect(db_path)

# Query sales summary by product
query = """
SELECT 
    product,
    SUM(quantity) AS total_qty,
    ROUND(SUM(quantity * price), 2) AS revenue
FROM sales
GROUP BY product
ORDER BY revenue DESC
"""
df = pd.read_sql_query(query, conn)
print(df)

# Query monthly sales summary
query_month = """
SELECT 
    strftime('%m', sale_date) AS month,
    SUM(quantity) AS total_qty,
    ROUND(SUM(quantity * price), 2) AS revenue
FROM sales
WHERE sale_date IS NOT NULL
GROUP BY month
ORDER BY month
"""
df_month = pd.read_sql_query(query_month, conn)
print(df_month)

conn.close()
```
5. Visualize the results
```python
import matplotlib.pyplot as plt

# Total quantity by product
plt.bar(df['product'], df['total_qty'])
plt.xlabel('Product')
plt.ylabel('Total Quantity')
plt.title('Total Quantity Sold by Product')
plt.show()

# Revenue by product
plt.bar(df['product'], df['revenue'])
plt.xlabel('Product')
plt.ylabel('Revenue')
plt.title('Total Revenue by Product')
plt.show()

# Total quantity by month
plt.bar(df_month['month'], df_month['total_qty'])
plt.xlabel('Month')
plt.ylabel('Total Quantity')
plt.title('Total Quantity Sold by Month')
plt.show()

# Revenue by month
plt.bar(df_month['month'], df_month['revenue'])
plt.xlabel('Month')
plt.ylabel('Revenue')
plt.title('Total Revenue by Month')
plt.show()
```
