# Blinkit Sales, Delivery & Inventory Dashboard (Power BI)

A three-page Power BI report built as a hands-on learning project. It analyses a quick-commerce (Blinkit-style) dataset across **sales performance**, **delivery performance** and **inventory / product health**.

## Business questions

- How are orders and revenue trending month by month, and which categories and products drive revenue?
- How reliable is delivery? What share of orders arrive on time, and which areas see the most delays?
- Which categories lose the most stock to damage, and does shelf life play a part?

## Dashboard pages

| Page | What it shows |
|------|---------------|
| **Sales Performance** | Total Orders, Item Revenue, Average Order Value; orders by month (2023 vs 2024); revenue by category; orders by payment method; Top 10 products; slicers for customer segment and year |
| **Delivery Performance** | On-Time %, Average Delay (min); revenue by delivery status; On-Time % by area; count of delivery status by area; Key Influencers on delivery delays; delivery status slicer |
| **Inventory & Products** | Damage Rate %; damaged vs received stock by category; shelf life vs damage rate; damage rate by product; product table with min/max stock levels and a colour-coded stock status; category slicer |

## Data

- **Source:** [add dataset link and licence here]
- **Tables used:** `blinkit_orders`, `blinkit_order_items`, `blinkit_products`, `blinkit_customers`, `blinkit_inventory`
- **Not used yet:** `blinkit_marketing_performance`, `blinkit_customer_feedback`, `blinkit_delivery_performance`, `blinkit_inventoryNew` (see Next steps)

## Data model

Order items were merged into orders in Power Query, so `blinkit_orders` holds one row per order line. The model is a star-style layout:

```
blinkit_customers (1) ──► (*) blinkit_orders (*) ◄── (1) blinkit_products (1) ──► (*) blinkit_inventory
                                    ▲
                                    │ (*)
                               Date (1)
```

All relationships are one-to-many with a single filter direction.

## Data preparation (Power Query)

- Promoted headers and set data types on every table
- Merged `blinkit_order_items` into `blinkit_orders` on `order_id` and expanded `product_id`, `quantity` and `unit_price`
- Removed duplicate rows
- Built a Date table (2023-2024) with Year, Month and Month Number, sorted months by Month Number and marked it as the date table
- Disabled load on the source `blinkit_order_items` query to avoid a second, ambiguous path between tables

Because order items were merged into orders, `order_total` repeats on every item row. The measures below avoid double counting.

## Key DAX measures

```DAX
Total Orders = DISTINCTCOUNT(blinkit_orders[order_id])

Item Revenue = SUMX(blinkit_orders, blinkit_orders[quantity] * blinkit_orders[unit_price])

Avg Order Value (items) = DIVIDE([Item Revenue], [Total Orders])

On-Time % =
DIVIDE(
    CALCULATE(DISTINCTCOUNT(blinkit_orders[order_id]), blinkit_orders[delivery_status] = "On Time"),
    DISTINCTCOUNT(blinkit_orders[order_id])
)

Avg Delay (min) =
AVERAGEX(
    blinkit_orders,
    DATEDIFF(blinkit_orders[promised_delivery_time], blinkit_orders[actual_delivery_time], MINUTE)
)

Damage Rate % = DIVIDE(SUM(blinkit_inventory[damaged_stock]), SUM(blinkit_inventory[stock_received]))

Net Stock = SUM(blinkit_inventory[stock_received]) - SUM(blinkit_inventory[damaged_stock])

Stock Status =
VAR net = [Net Stock]
VAR minL = MAX(blinkit_products[min_stock_level])
VAR maxL = MAX(blinkit_products[max_stock_level])
RETURN
IF(ISBLANK(net), BLANK(),
   IF(net < minL, "Below min",
      IF(net > maxL, "Above max", "OK")))
```

## Key findings

- About 5K orders and roughly 5M in item revenue, with an average order value of about 994
- Dairy & Breakfast, Pharmacy and Fruits & Vegetables are the top categories by item revenue; Vitamins is the top product
- Orders are split almost evenly across Card, Cash, Wallet and UPI (about 25% each)
- About 69% of orders are delivered on time
- Average delivery delay: [fill in after confirming the measure]
- Key Influencers flagged specific areas (for example Kolhapur and Raipur) as more likely to see slight delays; treat this as exploratory, since each area has relatively few orders

## Limitations

- `Stock Status` compares total stock received in the selected period with each product's min/max level. It is an indicator, not true on-hand inventory, because the data has no opening stock or sales deduction.
- The monthly trend appears to start in March 2023 and end in early November 2024, so compare full months only.
- Key Influencers shows association, not cause.

## Tools and skills demonstrated

Power BI Desktop, Power Query (M), DAX, data modelling and relationships (cardinality, filter direction), date tables, Top N filters, conditional formatting, Key Influencers, slicers and tooltips.

## Next steps

- Add the delivery performance, customer feedback and marketing tables to the model
- Build a customer satisfaction and campaign performance page
- Rebuild the same dashboards in Tableau for comparison

## How to open

1. Download `blinkit_dashboard.pbix` and open it in Power BI Desktop (free).
2. To refresh with the files in `/data`, go to **Transform data > Data source settings** and point each source to your local copy.

## Author

**Okerio Davis**, Nairobi, Kenya
LinkedIn: [linkedin.com/in/okeriodavis](https://www.linkedin.com/in/okeriodavis)
