## README
# Overview
This script enriches an order dataset with historical stock context. Specifically, for each order in the order_details.csv file, it determines the most recent known stock level of the corresponding product from the product_stock_levels.csv file, at or before the time the order was placed.
The result is a new dataset enriched_orders.csv which retains the original order details and includes an additional column: stock_level_at_order_time.
This enriched view helps answer business-critical questions such as:
Was there enough stock when the order was placed?
Are there trends in stock shortages tied to order volumes?
Can fulfillment delays be attributed to insufficient inventory?

This type of historical enrichment supports root-cause analysis, backorder investigations, and more reliable forecasting.

# How to Set Up and Run the Script
Prerequisites:
Python 3.8+
The pandas library

Installation:
Create a virtual environment (optional but recommended), then install dependencies:
pip install -r requirements.txt

Input Files:
Ensure these two CSV files are in the same directory as the script:
order_details.csv — contains at least: order_id, product_id, quantity, and order_timestamp
product_stock_levels.csv — contains at least: product_id, stock_level, and timestamp

Run the script:
python enrich_orders.py

Output:
A file named enriched_orders.csv will be created, containing the original order records with one new column:
stock_level_at_order_time: the latest known stock level at or before the order time

# Assumptions and Design Choices
1. Handling Missing Quantities in order_details:
If the quantity column has missing values, the script fills them as follows:
First, it calculates the median quantity per product_id and uses that where available.
If a product has no valid quantity records, the global median is used instead.
This approach avoids arbitrary imputation and respects product-level behavior, while also avoiding dropping potentially valuable order records.

2. Stock Lookup Logic:
For each order, we fetch the latest available stock record for that product with a timestamp at or before the order time. This simulates what the inventory system would have recorded at the time of order placement. Future stock levels are not considered, to avoid contamination with data not yet available at order time.

3. Missing Stock Data:
If no stock data exists for a product prior to the order timestamp, the script assigns a null (None) value to stock_level_at_order_time. This flags that no valid stock history was available, which could reflect gaps in data tracking or new product additions.

4. Time Efficiency:
The script uses sorted data and row-wise application with efficient filtering to match order timestamps to stock records. While suitable for modest-sized datasets, further optimizations (e.g., indexed joins or interval trees) may be beneficial for large-scale datasets.
