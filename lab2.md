## Лаба 2
1. Найти и вывести на экран количество товаров каждого цвета, исключив из поиска товары, цена которых меньше 30.

```sql
select color, count (*) from production.product
where listprice>30 and color is not null
group by color
```

2\. Найти и вывести на экран список, состоящий из цветов товаров, таких, что минимальная цена товара данного цвета более 100.

```sql
select color from production.product
group by color
having min(listprice)>100
```

3\. Найти и вывести на экран номера подкатегорий товаров и количество товаров  в каждой подкатегории.

```sql
select productsubcategoryid, count(*) from production.product
where productsubcategoryid is not null
group by productsubcategoryid
```

4\. Найти и вывести на экран номера товаров и количество фактов продаж данного товара (используется таблица SalesORDERDetail).

```sql
select productid, count(*) from sales.Sales_ORDER_Detail
group by productid
```

5\. Найти и вывести на экран номера товаров, которые были куплены более пяти раз.

```sql
select productid, count(*) from sales.Sales_ORDER_Detail
group by productid
having count(*)>5;
```

6\. Найти и вывести на экран номера покупателей, CustomerID, у которых существует более одного чека, SalesORDERID, с одинаковой датой

```sql
select customerid, count(*) from sales.Sales_ORDER_header
group by customerid, orderdate
having count(salesorderid)>1;
```

7\. Найти и вывести на экран все номера чеков, на которые приходится более трех продуктов.

```sql
select salesorderid from sales.Sales_ORDER_detail
group by salesorderid
having count(productid)>3;
```

8\. Найти и вывести на экран все номера продуктов, которые были куплены более трех раз.

```sql
select productid from sales.Sales_ORDER_detail
group by productid
having count(*)>3
```


9\. Найти и вывести на экран все номера продуктов, которые были куплены или три или пять раз.

```sql
select productid from sales.Sales_ORDER_detail
group by productid
having count(*)=3 or count(*)=5;
```

10\. Найти и вывести на экран все номера подкатегорий, в которым относится более десяти товаров.

```sql
select productsubcategoryid from production.product
group by productsubcategoryid
having count(name)>10;
```

11\. Найти и вывести на экран номера товаров, которые всегда покупались в одном экземпляре за одну покупку.

```sql
select productid from sales.sales_order_detail
group by productid
having max(orderqty)=1;
```

12 Найти и вывести на экран номер чека, SalesORDERID, на который приходится с наибольшим разнообразием товаров купленных на этот чек. 

```sql
select salesorderid from sales.sales_order_detail
group by salesorderid
order by count(*) desc
limit 1
```

13\. Найти и вывести на экран номер чека, SalesORDERID с наибольшей суммой покупки, исходя из того, что цена товара – это UnitPrice, а количество конкретного товара в чеке – это ORDERQty.

```sql
select salesorderid from sales.sales_order_detail
group by salesorderid
order by sum(unitprice*orderqty) desc
limit 1
```

14\. Определить количество товаров в каждой подкатегории, исключая товары, для которых подкатегория не определена, и товары, у которых не определен цвет.

```sql
select productsubcategoryid, count(*) from production.product
where productsubcategoryid is not null and color is not null
group by productsubcategoryid
```

15\. Получить список цветов товаров в порядке убывания количества товаров данного цвета

```sql
select color, count(*) from production.product
group by color
order by count(*) desc
```

16\. Вывести на экран ProductID тех товаров, что всегда покупались в количестве более 1 единицы на один чек, при этом таких покупок было более двух.

```sql
select productid from sales.sales_order_detail
group by productid
having min(orderqty)>1 and count(salesorderid)>2;
```

17.  Разбить продукты по количеству символов в названии, для каждой группы определить количество продуктов.  

```sql
select length(name),count(*) from production.product
group by length(name)
```

  18.Найти номера первых трех подкатегорий (ProductSubcategoryID) с наибольшим количеством наименований товаров 

```sql
select productsubcategoryid,count(*) from production.product
where productsubcategoryid is not null
group by productsubcategoryid
order by count(*) desc
limit 3
```

19\. Проверить, есть ли продукты с одинаковым названием, если есть, то вывести эти названия  

```sql
select name,count(*) from production.product
group by productid,name
having count(*)>1;
```

20\.Найти и вывести на экран список товаров, которые были куплены в сумме более 1000 рублей за все время.  

```sql
select productid,count(*),orderqty*unitprice as p from sales.sales_order_detail
group by productid, p
having sum(orderqty*unitprice)>1000

```
