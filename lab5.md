## Лаб5
1. Найти среднее количество покупок на чек для каждого покупателя (2 способа).

   ```sql
   with ordercount (sales_order_id, customer_id, pcount) as(
   	select sod.sales_order_id, customer_id, count(product_id) as pcount from sales.sales_order_detail as sod
   	join sales.sales_order_header as soh
   	on sod.sales_order_id=soh.sales_order_id
   	group by customer_id, sod.sales_order_id
   )
   select customer_id, avg(pcount) from ordercount
   group by customer_id
   order by customer_id asc
   ```
2. Найти для каждого продукта и каждого покупателя соотношение количества фактов покупки данного товара данным покупателем к общему количеству фактов покупки товаров данным покупателем

   ```sql
   with ordercount as (
   	select soh.customer_id, product_id, count(*) as product 
   	from sales.sales_order_detail as sod
   	join sales.sales_order_header as soh
   	on soh.sales_order_id = sod.sales_order_id
   	group by soh.customer_id, product_id
   ),
   totalord as (
   	select customer_id, sum(product) as sumprod  from ordercount
   	group by customer_id
   )
   select oc.customer_id, (oc.product::decimal/tot.sumprod) from ordercount as oc
   JOIN totalord AS tot
   ON oc.customer_id = tot.customer_id
   ORDER BY oc.customer_id
   ```
3. Вывести на экран следующую информацию: Название продукта, Общее количество фактов покупки этого продукта, Общее количество покупателей этого продукта

   ```sql
   with info as (
   	select p.name as productname, count(soh.customer_id) as customercnt, count(sod.sales_order_id) as sodcnt
   	from production.product as p
   	join sales.sales_order_detail as sod
   	on sod.product_id=p.product_id
   	join sales.sales_order_header as soh
   	on sod.sales_order_id = soh.sales_order_id
   	group by p.name
   )
   select productname,customercnt,sodcnt from info
   
   ```
4. Вывести для каждого покупателя информацию о максимальной и минимальной стоимости одной покупки, чеке, в виде таблицы: номер покупателя, максимальная сумма, минимальная сумма.

   ```sql
   with sums (customer_id, price) as (
   	select soh.customer_id, sod.unit_price from sales.sales_order_detail as sod
   	join sales.sales_order_header as soh
   	on sod.sales_order_id = soh.sales_order_id
   	group by customer_id, unit_price
   )
   select customer_id, max(price), min(price) from sums
   group by customer_id
   order by customer_id asc
   ```
5. Найти номера покупателей, у которых не было нет ни одной пары чеков с одинаковым количеством наименований товаров.

   ```sql
   WITH orders(Customer_ID, Sales_Order_ID, Product_ID)
   	as (SELECT Customer_ID, sod.Sales_Order_ID, Product_ID
   		FROM Sales.Sales_Order_Detail sod 
   		INNER JOIN Sales.Sales_Order_Header soh
   		ON sod.Sales_Order_ID = soh.Sales_Order_ID)
   SELECT Customer_ID 
   FROM Sales.Customer 
   WHERE Customer_ID NOT IN (
   	SELECT DISTINCT o1.Customer_ID 
   	FROM orders o1
   	INNER JOIN orders o2
   	ON o1.Customer_ID = o2.Customer_ID
   	AND o1.Sales_Order_ID != o2.Sales_Order_ID
   	AND o1.Product_ID = o2.Product_ID)
   ```
6. Найти номера покупателей, у которых все купленные ими товары были куплены как минимум дважды, т.е. на два разных чека.

   ```sql
   with cnt as (
   	select soh.customer_id, sod.product_id
   	from sales.sales_order_detail as sod
   	join sales.sales_order_header as soh
   	on soh.sales_order_id = sod.sales_order_id
   	group by soh.customer_id, sod.product_id
   	having count(*) =1
   )
   select distinct customer_id from sales.sales_order_header
   WHERE customer_id not in (
   	SELECT customer_id FROM cnt
   )
   ```
