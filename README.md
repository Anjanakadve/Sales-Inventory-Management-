# Sales-Inventory-Manegement
# Project Overview
Sales & inventory data for a fictitious toy store chain in Mexico called Maven Toys, including information about products, stores, daily sales transactions, and current inventory levels at each location.
# Sql Queries
#### 1. location with the highest concentration of stores.

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

-  **Store distribution** Downtown 29 (58%) | Commercial 12 (24%) | Residential 6 (12%) | Airport 3 (6%)

#### 2. Are stores in the "Downtown" area more profitable than those in the "Airport" or "Residential" or 'Commercial' areas
```sql
  select st.store_location, 
    sum(sa.units * (p.product_price - p.product_cost)) as total_profit
from stores st
join sales sa on st.store_id = sa.store_id
join products p on sa.product_id = p.product_id
group by st.store_location
order by total_profit desc;
```
| Store_Location | Total_Profit |
| :--- | :--- |
| **Downtown** | 493064.00 |
| **Commercial** | 206288.00 |
| **Residential** | 100542.00 |
| **Airport** | 84428.00 |

* **Revenue Concentration:** The Downtown sector acts as the primary financial engine, contributing approximately 56% of total profits across the entire store network.

#### 3. Product Category with their total units sold and total profit
```sql
select 
    p.product_category,
    sum(sa.units) as total_units_sold,
    sum(sa.units * (p.product_price - p.product_cost)) as total_profit
from sales sa
join products p on sa.product_id = p.product_id
join stores st on sa.store_id = st.store_id
group by p.product_category
order by total_profit desc;
```

| Product_Category | Total_Units_Sold | Total_Profit |
| :--- | :--- | :--- |
| **Electronics** | 41588 |	322252.00 |
| **Toys** |   63593 |	252094.00 |
| **Games** |   41864 |	136725.00 |
| **Sports & Outdoors** |  32626 |	99140.00 |
| **Art & Crafts** |  30221	| 74111.00  |                                           

 -  Electronics dominates profit margin at $7.74/unit generating $322,252 (35.1% of total profit) with only 41,588 units
 -  Art & Crafts has lowest margin ($2.45/unit) despite 30,221 units sold generating only $74,111 


#### 4.  Which product categories are driving the most profit,and how efficient is our pricing strategy for each
```sql
select  p.product_category,
        round(sum(ifnull(p.product_price - p.product_cost, 0) * s.units), 2) as total_profit,
        round(
            sum(ifnull(p.product_price - p.product_cost, 0) * s.units) / 
            nullif(sum(p.product_price * s.units), 0) * 100,  2
            ) as avg_margin_percentage
from products p
join sales s on p.product_id = s.product_id
group by p.product_category
order by total_profit desc;
```

| Product_Category | Total_Profit  | Avg_Margin_Percentage  |
| :--- | :--- | :--- |
| **Electronics** | 322,252.00 | **49.56** |
| **Toys** | 252,094.00 | 23.74 |
| **Games** | 136,725.00 | 34.83 |
| **Sports & Outdoors** | 99,140.00 | 21.35 |
| **Art & Crafts** | 74,111.00 | 28.69 |

* **Premium Performance:** Electronics makes 50 cents profit  of sales, so even though we sell fewer electronics items, they make the most money for the business.


#### 5. Top 5 Stores with Total Inventory values
```sql
select 
    s.store_name,
    sum(i.stock_on_hand * p.product_cost) as total_inventory_value
from inventory i
join products p on i.product_id = p.product_id
join stores s on i.store_id = s.store_id
group by s.store_name
order by total_inventory_value desc 
limit 5;
```

| Store_Name | Total_Inventory_Value  |
| :--- | :--- |
| **Maven Toys Ciudad de Mexico 2** | 8,917.85 |
| **Maven Toys Hermosillo 1** | 7,923.49 |
| **Maven Toys Hermosillo 3** | 7,666.44 |
| **Maven Toys Saltillo 2** | 7,578.93 |
| **Maven Toys Chihuahua 2** | 7,565.26 |

* **Inventory Insights:** Ciudad de Mexico 2 holds the highest inventory value at 8,917.85, while the remaining
 top 5 stores are closely grouped between 7,500–7,900, indicating consistent stock distribution across locations.


#### 6.which categories are taking up the most physical space in your stores with their total inventory values
```sql
select 
    p.product_category,
    sum(i.stock_on_hand) as total_items,
    round(sum(i.stock_on_hand * p.product_cost), 2) as total_inventory_value
from products p
join inventory i on p.product_id = i.product_id
group by p.product_category
order by total_inventory_value desc;
```

| Product_Category   | Total_Items | Total_Inventory_Value|
|--------------------|-------------|--------------------------|
| Toys               |   7553      |     	99861.47            |
|    Art & Crafts    |   	8635     |      65075.65            |
| Sports & Outdoors  |  	4981     |    	53077.19            |
|    Games	         |       6155  |	51489.45                |
|    Electronics     |     	2418   |	30705.82                |


* **Category Insights:** Toys dominate inventory value at $99,861 despite Art & Crafts holding the most physical stock (86,35 items),
suggesting Art & Crafts are low-cost but space-heavy, while Electronics hold the least items yet carry notable value per unit.

#### 7. Which cities in Mexico are generating the highest total profit
```sql
select 
    st.store_city,
    round(sum((p.product_price - p.product_cost) * sa.units), 2) as city_profit
from stores st
join sales sa on st.store_id = sa.store_id
join products p on sa.product_id = p.product_id
group by st.store_city
order by city_profit desc
limit 5;
```
| Store_City      | City_Profit |
|-----------------|-----------------|
| Ciudad de Mexico| 101,951.00      |
| Guadalajara     | 82,953.00       |
| Monterrey       | 79,639.00       |
| Hermosillo      | 57,724.00       |
| Guanajuato      | 48,920.00       |

* **City Profit Insights:** Ciudad de Mexico leads all cities with $101,951 in profit — nearly 23% more than Guadalajara ($82,953),
 while the top 3 major metro cities (CDMX, Guadalajara, Monterrey) together contribute over 70% of the total top-5 profit.


#### 8. Which categories are holding too much stock compared to how fast they actually sell
```sql
with category_sales as (
      select 
        product_id,
        sum(units) / count(distinct date) as daily_velocity
    from sales
    group by product_id
)
select 
    p.product_category,
    sum(i.stock_on_hand) as total_units_in_stock,
    round(sum(i.stock_on_hand * p.product_cost), 2) as inventory_value,
    round(sum(i.stock_on_hand) / sum(ifnull(cs.daily_velocity, 0.1)), 0) as days_of_stock_on_hand
from products p
join inventory i on p.product_id = i.product_id
left join category_sales cs on p.product_id = cs.product_id
group by p.product_category
order by inventory_value desc;
```

| Product_Category  | Total_Units_In_Stock | Inventory_Value     | Days_Of_Stock_On_Hand |
|-------------------|----------------------|---------------------|-----------------------|
|Toys               |          	7553       |      	99861.47     |        	0            |
| Art & Crafts      |         	8635	     |         65075.65	   |          1            |
| Sports & Outdoors |         	4981       |       	53077.19     |        	0            |
|   Games           |         	6155       |        	51489.45   |           	0          |
|  Electronics	    |           2418       |          	30705.82 |           0           |

 * **Overstock Insights:**  Art & Crafts holds the most stock (8635 units) but sells slowly, making it the most overstocked category.
  Toys lead in inventory value ($99861) with fast turnover


#### 9.  Top Selling Product Per Month
```sql
with monthly_product_sales as (
    select 
        monthname(s.date) as sales_month,
        p.product_name,
        sum(s.units) as product_units,
        sum(s.units * p.product_price) as product_revenue,
        rank() over (partition by monthname(s.date) order by sum(s.units) desc) as sales_rank
    from sales s
    join products p on s.product_id = p.product_id
    group by sales_month, p.product_name
)
select 
    sales_month,
    product_name as top_selling_product,
    product_units as units_sold,
    product_revenue as monthly_revenue
from monthly_product_sales
where sales_rank = 1
order by sales_month desc;
```
| Sales_Month | Top_Selling_Product | Units_Sold | Monthly_Revenue      |
|-------------|---------------------|------------|----------------------|
| January     | Colorbuds           | 8110      | 121568.90           |
| February    | Colorbuds           | 7627      | 114328.73           |
| March       | Colorbuds           | 7843      | 117566.57           |
| April       | Colorbuds           | 7267      | 108932.33           |
| May         | Colorbuds           | 6974      | 104540.26           |
| June        | Colorbuds           | 2834      | 242.17              |

 * **Monthly Sales Insights:** Colorbuds dominates as the top-selling product every single month.
 Sales peak in January ($121K) and steadily decline through June, suggesting possible seasonality or stock issues mid-year.

## Sales & Inventory Management - Key Recommendations

## 1. Optimize Art & Crafts Inventory
Art & Crafts has too much inventory (8,635 units) but sells slowly. Run sales or discounts to clear stock faster.

## 8. Investigate Games Category
Games have good margins (34.83%) but are underperforming compared to Electronics and Toys. Review pricing strategy or product selection to boost sales velocity.

## 3. Investigate June Sales Collapse
Colorbuds revenue dropped dramatically in June ($242.17 vs. $104K-$121K in prior months). Investigate inventory stockouts, supply chain issues, or demand shifts. This 75% revenue decline requires immediate attention to prevent further loss.

## 4. Leverage Electronics High-Margin Strategy
Electronics has the highest profit margin (49.56%) generating $322,252 profit on only 41,588 units. Increase Electronics shelf space, improve visibility, and upsell bundled packages. Even a 10% sales increase would add ~$32K in profit with minimal inventory investment.






































