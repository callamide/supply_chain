-- creating and importing data from my excel sheets -- 
create database supplychain;

USE supplychain;

-- test queries --
select Carrier, MAX(Weight)
from order_list;

select Carrier, TPT
from order_list
where TPT > 1 ;

-- Find the total number of orders per customer from the OrderList table.--
select count(`Order ID`), Customer
from Order_list
group by customer 
order by count(`Order ID`) desc;


-- List the supported warehouse-product combinations along with their corresponding capacities --
select p.`Plant Code`, p.`Product ID`,
		w.`Daily Capacity`
from product_percost p
join warehouse_capacities w on p.`Plant Code` = w.`Plant ID`;

-- List all warehouses that have not reached their full capacity--
select `Plant ID`, max(`Daily Capacity`)
from warehouse_capacities;

SELECT w.`Plant ID`, w.`Daily Capacity` - COALESCE(SUM(o.`Order ID`), 0) AS remaining_capacity
		FROM warehouse_Capacities w
LEFT JOIN Order_list o ON w.`Plant ID` = o.`Plant Code`
GROUP BY w.`Plant ID`, w.`Daily Capacity`
HAVING remaining_capacity > 0;

-- Identify which products are ordered and in demand the most from the OrderList table --
select `Product ID`, Count(`Order ID`) As Orders,
		sum(`Unit quantity`) as Demand
from Order_list
group by `Product ID`
Order by Orders Desc;

-- Determine the total storage cost for each warehouse. --
select w.`Plant ID`, sum(w.`Daily Capacity`*wc.`Cost/unit`) as storage_cost
from warehouse_capacities w
Join warehouse_costs wc on w.`Plant ID` = wc.WH
group by `Plant ID`
order by storage_cost desc;

-- Calculate the total freight cost for all orders based on their weights and destinations--
Select o.`Order ID`, sum(o.Weight * f.rate) AS total_freight_cost
FROM Order_list o
JOIN Freightrates f ON o.`Destination Port` = f.dest_port_cd
GROUP BY o.`Order ID`;

SELECT orig_port_cd, dest_port_cd, 
		SUM(tpt_day_cnt) AS total_shipping_time
FROM FreightRates
group by orig_port_cd
order by sum(tpt_day_cnt) asc;

 
-- Determine the total shipping time for all orders based on the transportation day count.
Select f.orig_port_cd, f.dest_port_cd, sum(o.Weight * f.rate) AS total_freight_cost
FROM Freightrates f 
JOIN Order_list o ON o.`Destination Port` = f.dest_port_cd
group by orig_port_cd
order by dest_port_cd
limit 5

-- Calculate the total cost of fulfilling all orders, considering both storage and freight rates --
SELECT o.`Order ID`, 
       SUM(o.`Unit quantity` * wc.`Cost/unit`) AS total_storage_cost, 
       SUM(o.Weight * f.rate) AS total_freight_cost,
       (SUM(o.`Unit quantity` * wc.`Cost/unit`) + SUM(o.Weight * f.rate)) AS total_cost
FROM Order_list o
JOIN freightrates f ON f.`dest_port_cd` = o.`Origin Port`
join Warehouse_Costs wc on wc.WH = o.`Plant Code`
GROUP BY o.`Order ID`;

-- Analyze trends in order fulfillment over time by finding the monthly order count and associated costs.--
SELECT date_format(o.`Order Date`, '%Y-%m-01') AS month, 
       COUNT(*) AS monthly_order_count,
       SUM(o.`Unit quantity`* wc.`Cost/unit`) AS monthly_storage_cost,
       SUM(o.Weight * f.rate) AS monthly_freight_cost
FROM Order_list o
JOIN Warehouse_Costs wc ON o.`Plant Code` = wc.WH
JOIN FreightRates f ON o.`Destination Port` = f.`dest_port_cd`
GROUP BY month
ORDER BY month;

-- catergorise port by types--
select Distinct(mode_dsc), orig_port_cd
from freightrates
group by mode_dsc
order by orig_port_cd;


