## Лаба 4
1. Найти название самого продаваемого продукта.

```sql
select name from production.product
where product_id = (
	select product_id from sales.sales_order_detail
	group by product_id
	order by count(*) desc
	limit 1
)
```

2\.Найти покупателя, совершившего покупку на самую большую сумму, считая сумму покупки исходя из цены товара без скидки (unit_price).

```sql
select customer_id from sales.sales_order_header
where sales_order_id in (
	select sales_order_id from sales.sales_order_detail
	group by sales_order_id
	having sum(unit_price*order_qty) = (
		select max(tp) from (
			select sum(unit_price*order_qty) as tp from sales.sales_order_detail
			group by sales_order_id
		)
	)
)
```

```sql
select customer_id from sales.sales_order_header
where sales_order_id = (
	select sales_order_id from sales.sales_order_detail
	group by sales_order_id
	order by sum(order_qty*unit_price) desc
	limit 1
)
```

3\.Найти такие продукты, которые покупал только один покупатель.

```sql
-- найти такие продукты которые покупал только один покупатель --
select product_id from sales.sales_order_detail
where sales_order_id in (
	select sales_order_id from sales.sales_order_header
	where customer_id in (
		select customer_id from sales.sales_order_header
		group by customer_id 
		having count(*) = 1
	) 
)
group by product_id
having count(sales_order_id)=1
```

4\.Вывести список продуктов, цена которых выше средней цены товаров в подкатегории, к которой относится товар

```sql
-- Вывести список продуктов, цена которых выше средней цены товаров в подкатегории, к которой относится товар --
select product_id from production.product as p
where list_price > all (
	select avg(list_price) from production.product as p1
	where p.product_subcategory_id = p1.product_subcategory_id
)
```

5\.Найти такие товары, которые были куплены более чем одним покупателем, при этом все покупатели этих товаров покупали товары только одного цвета и товары не входят в список покупок покупателей, купивших товары только двух цветов.

```sql
```

6\.Найти такие товары, которые были куплены такими покупателями, у которых  они присутствовали в каждой их покупке.

```sql
select product_id from sales.sales_order_detail
where product_id in (
	select product_id from sales.sales_order_detail as sod
	where not exists (
		select 1 from sales.sales_order_header as soh
		where customer_id = (
			select customer_id from sales.sales_order_header 
			where sales_order_id=sod.sales_order_id
		)
		and not exists (
			select 1 from sales.sales_order_detail as sod2
			where sod2.sales_order_id = soh.sales_order_id
			and sod2.product_id = sod.product_id
		)
	)
)
group by product_id
```

7\.Найти покупателей, у которых есть товар, присутствующий в каждой покупке/чеке.

```sql
select customer_id from sales.customer
where not exists (
	select 1 from sales.sales_order_header as soh
	where soh.customer_id=sales.customer.customer_id
	and not exists (
		select 1 from sales.sales_order_detail as sod
		where sod.sales_order_id=soh.sales_order_id
		and sod.product_id in (
			select product_id from sales.sales_order_detail
			where sales_order_id=soh.sales_order_id
		)
	)
)
```

```sql
select customer_id from sales.customer as c
where not exists (
	select 1 from sales.sales_order_header as soh
	where soh.customer_id = c.customer_id
	and not exists (
		select 1 from sales.sales_order_detail as sod
		where product_id in (
			select product_id from sales.sales_order_detail 
			where sod.product_id = product_id
		)
	)
)
--найти покупателей у которых есть товар присутсвующий в каждой покупке/чеке--
```

8\.Найти такой товар или товары, которые были куплены не более чем тремя различными покупателями.

```sql
select product_id from sales.sales_order_detail as sod
group by product_id
having count( distinct(
	select customer_id from sales.sales_order_header as soh
	where sod.sales_order_id=soh.sales_order_id
))<=3
--найти товар который был куплен не более чем тремя различными покупателями--
```

9\.Найти все товары, такие что их покупали всегда с товаром, цена которого максимальна в своей категории.

```sql
```

10\.Найти номера тех покупателей, у которых есть как минимум два чека, и каждый из этих чеков содержит как минимум три товара, каждый из которых как минимум был куплен другими покупателями три раза.

```sql
--найти номера покупателей, у которых есть минимум 3 продукта в чеке, 2 чека, был куплен 3 раза другими
select customer_id from sales.sales_order_header
where customer_id in (
	select customer_id from sales.sales_order_header
	where sales_order_id in (
		select sales_order_id from sales.sales_order_detail
		where product_id in (
			select product_id from sales.sales_order_detail
			group by product_id
			having count(*)>=3
		)
		group by sales_order_id
		having count(product_id)>=3
	)
	group by customer_id
	having count(sales_order_id)>=2
)
```

11\.Найти все чеки, в которых каждый товар был куплен дважды этим же покупателем.

```sql
select sales_order_id from sales.sales_order_detail
where sales_order_id in (
	select sales_order_id from sales.sales_order_detail
	group by sales_order_id, product_id
	having sum (2*order_qty) = 2* count(distinct product_id)
)
group by sales_order_id
having count(distinct product_id) = (
	select count(distinct product_id) from sales.sales_order_detail as sod
	where sod.sales_order_id=sales.sales_order_detail.sales_order_id
	
) --найти чеки в которыйх каждый товар был куплен дважды
```

12\.Найти товары, которые были куплены минимум три раза различными покупателями.

```sql
--Найти товары, которые были куплены минимум три раза различными покупателями.
select product_id from sales.sales_order_detail
where sales_order_id in (
	select sales_order_id from sales.sales_order_header
	where customer_id in (
		select customer_id from sales.sales_order_header
		group by customer_id
		having count(customer_id) >= 3
	)
)
group by product_id
having count(*) >= 3
```

13\.Найти такую подкатегорию или подкатегории товаров, которые содержат более трех товаров, купленных более трех раз.

```sql
--найти подкатегорию товаров которые содержат более трех товаров и куплены более трех раз
select product_subcategory_id from production.product
where product_id in(
	select product_id from sales.sales_order_detail
	where sales_order_id in (
		select sales_order_id from sales.sales_order_detail
		group by sales_order_id 
		having count(*) >3
	)
)
group by product_subcategory_id
having count(*)>3
```

```sql
select name from production.product_subcategory
where product_subcategory_id in (
	select product_subcategory_id from production.product
	where product_id in (
		select product_id from sales.sales_order_detail
		where sales_order_id in (
			select sales_order_id from sales.sales_order_detail
			group by sales_order_id
			having count(*)>3
		)
	)
	group by product_subcategory_id
	having count(*)>3
)
```

14\.Найти те товары, которые не были куплены более трех раз, и как минимум дважды одним и тем же покупателем.

```sql
select product_id from sales.sales_order_detail
where sales_order_id in (
	select sales_order_id from sales.sales_order_header 
	where customer_id in (
		select customer_id from sales.sales_order_header
		group by customer_id
		having count(*)>=2
	)
)
group by product_id
having count(*)<=3
--найти товары которые были куплены менее 3 раз и как минимум дважды одним и тем же покупателем
```
