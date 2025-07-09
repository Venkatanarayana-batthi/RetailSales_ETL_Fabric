
# 💼 Microsoft Fabric Project – Sales ETL Pipeline

This project demonstrates an **end-to-end ETL pipeline** using **Microsoft Fabric**, featuring:
- 💾 Data ingestion from CSVs
- 🔍 Lookup of product categories
- 🔁 Iterative transformation using **ForEach** and **Notebook**
- 🧪 Delta table output for downstream analytics

---

## 📁 Project Structure

```
Sales_ETL_Pipeline_Project/
├── data/
│   ├── products.csv
│   └── sales_data.csv
├── notebooks/
│   └── transform_sales_data.ipynb
├── pipeline/
│   └── Sales_ETL_Pipeline (JSON files exported from Fabric)
└── README.md
```

---

## 🧠 Key Components

### 1. 📄 Source Data

* **products.csv** – Contains product details and category info  
* **sales_data.csv** – Contains sales transactions  

These files are stored under:  
`RetailSalesLakehouse > Files > data/`

---

### 2. 📘 Notebook: `transform_sales_data`

This notebook is executed dynamically for each category. It:

* Accepts a **category name** from the pipeline
* Reads both CSVs
* Filters sales by category
* Joins product details
* Calculates revenue
* Writes output as Delta table to:  
  `Tables > curated > sales_by_category`

#### 🔧 Key code from notebook:

```python
category = spark.conf.get("category", "Electronics")
sales_df = spark.read.option("header", True).csv("Files/data/sales_data.csv")
products_df = spark.read.option("header", True).csv("Files/data/products.csv")
joined_df = sales_df.join(products_df, on="product_id", how="inner")
filtered_df = joined_df.filter(joined_df["category"] == category)
from pyspark.sql.functions import col
result_df = filtered_df.withColumn("revenue", col("quantity") * col("unit_price"))
result_df.write.mode("overwrite").format("delta").save("Tables/curated/sales_by_category")
```

---

### 3. 🔁 Pipeline Design

#### ✅ Lookup: `LookupCategories`

* Type: File  
* Path: `Files/data/products.csv`  
* Reads distinct product categories from CSV

#### 🔁 ForEach: `ForEachCategory`

* Sequential execution  
* Iterates over output from Lookup  
* Runs the notebook for each category

#### ▶ Activity: `RunNotebookForCategory`

* Executes notebook with parameter:

```json
{
  "category": "@item().category"
}
```

---

## 🖼 Screenshots

### 🔍 Lookup Activity Setup

![Lookup](screenshots/lookup-setup.png)

### 🔁 ForEach Setup

![ForEach](screenshots/foreach-setup.png)

### 📘 Notebook Code

![Notebook](screenshots/notebook-code.png)

---

## ✅ Output

Output is stored as **Delta table** under:  
`RetailSalesLakehouse > Tables > curated > sales_by_category`

This table is ready for **Power BI** or **Direct Lake queries**.

---

## 🚀 How to Run

1. Upload `products.csv` and `sales_data.csv` to:  
   `Files > data/`
2. Import the notebook and set up the connection to the Lakehouse.
3. Import and open the pipeline.
4. Validate → Run the pipeline.

---

## 📦 Deployment

To deploy this project:

* Clone the repo  
* Import pipeline JSON via Fabric UI  
* Update connections to your own Lakehouse  
* Run the pipeline

---

## 📌 Tools Used

* **Microsoft Fabric**
* **Data Factory Pipelines**
* **Notebooks (PySpark)**
* **Delta Lake Tables**

---

