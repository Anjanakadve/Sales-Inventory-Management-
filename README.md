# Sales-Inventory-Manegement
# Project Overview
Sales & inventory data for a fictitious toy store chain in Mexico called Maven Toys, including information about products, stores, daily sales transactions, and current inventory levels at each location.
# sql Queries
#### 1.location with the highest concentration of stores.

```sql
select count(store_id) as stores ,store_location  from stores
GROUP BY store_location having  count(store_id)
order by stores
```
| Stores | Store_Location |
| :----- | :------------- |
| 3      | Airport        |
| 6      | Residential    |
| 12     | Commercial     |
| 29     | Downtown       |

* **Downtime**

#### 2.Are stores in the "Downtown" area more profitable than those in the "Airport" or "Residential" or 'Commercial' areas
```sql
  SELECT st.Store_Location, 
    SUM(sa.Units * (p.Product_Price - p.Product_Cost)) AS Total_Profit
FROM Stores st
JOIN Sales sa ON st.Store_ID = sa.Store_ID
JOIN Products p ON sa.Product_ID = p.Product_ID
GROUP BY st.Store_Location
ORDER BY Total_Profit DESC;
```
| Store_Location | Total_Profit |
| :--- | :--- |
| **Downtown** | 493064.00 |
| **Commercial** | 206288.00 |
| **Residential** | 100542.00 |
| **Airport** | 84428.00 |

* **Revenue Concentration:** The Downtown sector acts as the primary financial engine, contributing approximately 56% of total profits across the entire store network.
* **Location Efficiency:** While Airport locations contribute the lowest total profit (84,428), they represent the highest profit-to-store ratio, indicating a more optimized and high-margin sales model per site.

#### 3.Product Category with their total units sold and total profit
```sql
SELECT 
    p.Product_Category,
    SUM(sa.Units) AS Total_Units_Sold,
    SUM(sa.Units * (p.Product_Price - p.Product_Cost)) AS Total_Profit
FROM Sales sa
JOIN Products p ON sa.Product_ID = p.Product_ID
JOIN Stores st ON sa.Store_ID = st.Store_ID
WHERE st.Store_City = 'Guadalajara' -- You can swap this for 'Monterrey', 'Campeche', etc.
GROUP BY p.Product_Category
ORDER BY Total_Profit DESC;
```

| Product_Category | Total_Units_Sold | Total_Profit |
| :--- | :--- | :--- |
| **Electronics** | 4104 | 31923.00 |
| **Toys** | 6555 | 25,878.00 |
| **Games** | 3165 | 10377.00 |
| **Sports & Outdoors** | 2503 | 7638.00 |
| **Art & Crafts** | 3160 | 7137.00 |

* **High-Margin Leadership:** Electronics is the premier profit driver (31,923), yielding the highest return despite moving lower volumes compared to the Toys category.
* **Volume vs. Value:** While Toys dominate in total units sold (6,555), the lower profit-per-unit suggests a strategy focused on high turnover and foot traffic rather than high individual margins.

#### 4.
```sql
SELECT 
    st.Store_City,
    SUM(CASE WHEN MONTH(sa.Date) = MONTH(CURDATE()) THEN sa.Units ELSE 0 END) AS Current_Month_Sales,
     SUM(CASE WHEN MONTH(sa.Date) = MONTH(DATE_SUB(CURDATE(), INTERVAL 1 MONTH)) THEN sa.Units ELSE 0 END) AS Last_Month_Sales,
   (SUM(CASE WHEN MONTH(sa.Date) = MONTH(CURDATE()) THEN sa.Units ELSE 0 END) - 
SUM(CASE WHEN MONTH(sa.Date) = MONTH(DATE_SUB(CURDATE(), INTERVAL 1 MONTH)) THEN sa.Units ELSE 0 END)) AS Unit_Difference
FROM Sales sa
JOIN Stores st ON sa.Store_ID = st.Store_ID
GROUP BY st.Store_City
ORDER BY Unit_Difference DESC;
```
| Store_City | Current_Month_Sales | Last_Month_Sales | Unit_Difference |
| :--- | :--- | :--- | :--- |
| **Tuxtla Gutierrez** | 923 | 655 | **+268** |
| **Mexicali** | 2,027 | 1,768 | **+259** |
| **Puebla** | 2,409 | 2,160 | **+249** |
| **Campeche** | 1,593 | 1,414 | **+179** |
| **San Luis Potosi** | 903 | 745 | **+158** |
| **Durango** | 578 | 429 | **+149** |
| **Cuidad de Mexico** | 4,478 | 4,339 | **+139** |
| **Chihuahua** | 1,611 | 1,480 | **+131** |
| **Saltillo** | 1,490 | 1,440 | **+50** |
| **Toluca** | 1,508 | 1,500 | **+8** |
| **Chilpancingo** | 555 | 574 | -19 |
| **Hermosillo** | 2,213 | 2,251 | -38 |
| **Zacatecas** | 352 | 400 | -48 |
| **Oaxaca** | 450 | 501 | -51 |
| **Villahermosa** | 762 | 819 | -57 |
| **Xalapa** | 1,305 | 1,370 | -65 |
| **Monterrey** | 2,974 | 3,059 | -85 |


* **Momentum Leaders:** Tuxtla Gutierrez and Mexicali are showing the strongest growth velocity, indicating a successful local sales surge or effective stock  in those regions.
* **Regional Contraction:** Larger markets like Monterrey and Xalapa are seeing a slight month-over-month decline, suggesting a cooling of demand or potential inventory gaps in major urban centers.


#### 5."Which product categories are driving the most profit,and how efficient is our pricing strategy for each?"
```sql
SELECT  p.Product_Category,
        ROUND(SUM(IFNULL(p.Product_Price - p.Product_Cost, 0) * s.Units), 2) AS Total_Profit,
    
     ROUND(
        AVG(IFNULL(p.Product_Price - p.Product_Cost, 0) / NULLIF(p.Product_Price, 0)) * 100, 
        2
    ) AS Avg_Margin_Percentage
FROM Products p
JOIN Sales s ON p.Product_ID = s.Product_ID
GROUP BY p.Product_Category
ORDER BY Total_Profit DESC;
```

| Product_Category | Total_Profit  | Avg_Margin_Percentage  |
| :--- | :--- | :--- |
| **Electronics** | 322,252.00 | **49.56** |
| **Toys** | 252,094.00 | 23.74 |
| **Games** | 136,725.00 | 34.83 |
| **Sports & Outdoors** | 99,140.00 | 21.35 |
| **Art & Crafts** | 74,111.00 | 28.69 |

* **Premium Performance:** Electronics is the most efficient category, commanding a near **50% margin**, making it the primary driver of bottom-line growth despite lower sales volume.
* **Volume vs. Efficiency:** Toys generate significant total profit through sheer volume, yet operate on a **thin 23.7% margin**, indicating a pricing strategy built on competitive, high-turnover sales.

#### 5. Top 5 Stores with Total Inventory values
```sql
SELECT 
    s.Store_Name,
    SUM(i.Stock_On_Hand * p.Product_Cost) AS Total_Inventory_Value
FROM Inventory i
JOIN Products p ON i.Product_ID = p.Product_ID
JOIN Stores s ON i.Store_ID = s.Store_ID
GROUP BY s.Store_Name
ORDER BY Total_Inventory_Value DESC limit 5 ;
```

| Store_Name | Total_Inventory_Value  |
| :--- | :--- |
| **Maven Toys Ciudad de Mexico 2** | 8,917.85 |
| **Maven Toys Hermosillo 1** | 7,923.49 |
| **Maven Toys Hermosillo 3** | 7,666.44 |
| **Maven Toys Saltillo 2** | 7,578.93 |
| **Maven Toys Chihuahua 2** | 7,565.26 |

> 💡 **Inventory Insights:** Ciudad de Mexico 2 holds the highest inventory value at 8,917.85, while the remaining
> top 5 stores are closely grouped between 7,500–7,900, indicating consistent stock distribution across locations.


#### 6.which categories are taking up the most physical space in your stores with their total inventory values
```sql
SELECT 
    p.Product_Category,
    SUM(i.Stock_On_Hand) AS Total_Items,
    ROUND(SUM(i.Stock_On_Hand * p.Product_Cost), 2) AS Total_Inventory_Value
FROM Products p
JOIN Inventory i ON p.Product_ID = i.Product_ID
GROUP BY p.Product_Category
ORDER BY Total_Inventory_Value DESC;
```

| Product_Category   | Total_Items | Total_Inventory_Value|
|--------------------|-------------|--------------------------|
| Toys               | 75,539      | 9,861.47                 |
| Art & Crafts       | 86,356      | 5,075.65                 |
| Sports & Outdoors  | 49,815      | 3,077.19                 |
| Games              | 61,555      | 1,489.45                 |
| Electronics        | 24,183      | 0,705.82                 |


> 💡 **Category Insights:** Toys dominate inventory value at $9,861 despite Art & Crafts holding the most physical stock (86,356 items),
> suggesting Art & Crafts are low-cost but space-heavy, while Electronics hold the least items yet carry notable value per unit.

#### 7. Which cities in Mexico are generating the highest total profit
```sql
SELECT 
    st.Store_City,
    ROUND(SUM((p.Product_Price - p.Product_Cost) * sa.Units), 2) AS City_Profit
FROM Stores st
JOIN Sales sa ON st.Store_ID = sa.Store_ID
JOIN Products p ON sa.Product_ID = p.Product_ID
GROUP BY st.Store_City
ORDER BY City_Profit DESC
LIMIT 5;
```
| Store_City      | City_Profit ($) |
|-----------------|-----------------|
| Ciudad de Mexico| 101,951.00      |
| Guadalajara     | 82,953.00       |
| Monterrey       | 79,639.00       |
| Hermosillo      | 57,724.00       |
| Guanajuato      | 48,920.00       |

> 💡 **City Profit Insights:** Ciudad de Mexico leads all cities with $101,951 in profit — nearly 23% more than Guadalajara ($82,953),
> while the top 3 major metro cities (CDMX, Guadalajara, Monterrey) together contribute over 70% of the total top-5 profit.


#### 8. Which categories are holding too much stock compared to how fast they actually sell
```sql
WITH CategorySales AS (
      SELECT 
        Product_ID,
        SUM(Units) / COUNT(DISTINCT Date) AS Daily_Velocity
    FROM Sales
    GROUP BY Product_ID
)
SELECT 
    p.Product_Category,
    SUM(i.Stock_On_Hand)                                                        AS Total_Units_In_Stock,
    ROUND(SUM(i.Stock_On_Hand * p.Product_Cost), 2)                             AS Inventory_Value,
    ROUND(SUM(i.Stock_On_Hand) / SUM(IFNULL(cs.Daily_Velocity, 0.1)), 0)        AS Days_Of_Stock_On_Hand
FROM Products p
JOIN Inventory i ON p.Product_ID = i.Product_ID
LEFT JOIN CategorySales cs ON p.Product_ID = cs.Product_ID
GROUP BY p.Product_Category
ORDER BY Inventory_Value DESC;
```

| Product_Category  | Total_Units_In_Stock | Inventory_Value     | Days_Of_Stock_On_Hand |
|-------------------|----------------------|---------------------|-----------------------|
| Toys              | 75,539               | 9,861.47            | 0                     |
| Art & Crafts      | 86,356               | 5,075.65            | 1                     |
| Sports & Outdoors | 49,815               | 3,077.19            | 0                     |
| Games             | 61,555               | 1,489.45            | 0                     |
| Electronics       | 24,183               | 0,705.82            | 0                     |

> 💡 **Overstock Insights:**  Art & Crafts holds the most stock (86,356 units) but sells slowly, making it the most overstocked category.
>  Toys lead in inventory value ($9,861) with fast turnover, while Electronics hold the least stock but maintain decent value.


#### 9.  Top Selling Product Per Month
```sql
WITH MonthlyProductSales AS (
    SELECT 
        MONTHNAME(s.Date) AS Sales_Month,
        p.Product_Name,
        SUM(s.Units) AS Product_Units,
        SUM(s.Units * p.Product_Price) AS Product_Revenue,
        RANK() OVER (PARTITION BY MONTHNAME(s.Date) ORDER BY SUM(s.Units) DESC) AS Sales_Rank
    FROM Sales s
    JOIN Products p ON s.Product_ID = p.Product_ID
    GROUP BY Sales_Month, p.Product_Name
)
SELECT 
    Sales_Month,
    Product_Name    AS Top_Selling_Product,
    Product_Units   AS Units_Sold,
    Product_Revenue AS Monthly_Revenue
FROM MonthlyProductSales
WHERE Sales_Rank = 1
ORDER BY Sales_Month DESC;
```
| Sales_Month | Top_Selling_Product | Units_Sold | Monthly_Revenue      |
|-------------|---------------------|------------|----------------------|
| January     | Colorbuds           | 8110      | 121568.90           |
| February    | Colorbuds           | 7627      | 114328.73           |
| March       | Colorbuds           | 7843      | 117566.57           |
| April       | Colorbuds           | 7267      | 108932.33           |
| May         | Colorbuds           | 6974      | 104540.26           |
| June        | Colorbuds           | 2834      | 242.17              |

> 💡 **Monthly Sales Insights:** Colorbuds dominates as the top-selling product every single month.
> Sales peak in January ($121K) and steadily decline through June, suggesting possible seasonality or stock issues mid-year.








































