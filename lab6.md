1. Найти долю продаж каждого продукта (цена продукта \* количество продукта), на каждый чек, в денежном выражении.

   ```sql
   select product_id, sales_order_id, (sum(order_qty) over(partition by product_id, sales_order_id) * Unit_Price)
   from Sales.Sales_Order_Detail AS SSOD 
   ```
2. Вывести на экран список продуктов, их стоимость, а также разницу между стоимостью этого продукта и стоимостью самого дешевого продукта в той же подкатегории, к которой относится продукт 53

   ```sql
   select product_id, list_price, product_subcategory_id,
   list_price - min(list_price) over (partition by product_subcategory_id) as diff
   from production.product
   where product_subcategory_id in (
   	select product_subcategory_id from production.product
   	where product_id = 53
   )
   ```
3. Вывести три колонки: номер покупателя, номер чека покупателя (отсортированный по возрастанию даты чека) и искусственно введенный порядковый номер текущего чека, начиная с 1, для каждого покупателя.

   ```sql
   select customer_id, sales_order_id, 
   row_number() over (partition by customer_id order by order_date asc)
   from sales.sales_order_header
   ```
4. Вывести номера продуктов, таких что их цена выше средней цены продукта в подкатегории, к которой относится продукт. Запрос реализовать двумя способами. В одном из решений допускается использование обобщенного табличного выражения.

   ```sql
   with t1 as(
   	select product_subcategory_id, product_id,list_price,
   	(avg(list_price) over (partition by product_subcategory_id)) as avgprice
   	from production.product
   )
   select product_id
   from t1
   where list_price > avgprice and product_subcategory_id is not null
   ```
5. Вывести на экран номер продукта, название продукта, а также информацию о среднем количестве этого продукта, приходящихся на три последних по дате чека, в которых был этот продукт.

   ```sql
   WITH last_three_sales AS (
       SELECT 
           sod.product_id,
           soh.sales_order_id,
           soh.order_date,
           sod.order_qty,
           ROW_NUMBER() OVER (PARTITION BY sod.product_id ORDER BY soh.order_date DESC) AS row_num
       FROM sales.sales_order_detail sod
       JOIN sales.sales_order_header soh
           ON sod.sales_order_id = soh.sales_order_id
   ),
   filtered_sales AS (
       SELECT 
           product_id,
           order_qty
       FROM last_three_sales
       WHERE row_num <= 3
   )
   SELECT 
       p.product_id,
       p.name AS product_name,
       ROUND(AVG(fs.order_qty), 2) AS avg_quantity_last_3_sales
   FROM production.product p
   LEFT JOIN filtered_sales fs
       ON p.product_id = fs.product_id
   GROUP BY p.product_id, p.name;
   ```
