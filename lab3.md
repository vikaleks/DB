## Лаба 3
1. Найти и вывести на экран название продуктов и название категорий товаров, к которым относится этот продукт, с учетом того, что в выборку попадут только товары с цветом Red и ценой не менее 100.

```sql
select p.name, c.name from production.product as p 
join production.product_subcategory as s 
on p.product_subcategory_id = s.product_subcategory_id
join production.product_category as c
on s.product_category_id = c.product_category_id
where p.color='Red' and p.list_price>=10
```

2. Вывести на экран названия подкатегорий с совпадающими именами.

```sql
select s1.name from production.product_subcategory as s1
inner join production.product_subcategory as s2
on s1.name = s2.name
and s1.product_subcategory_id != s2.product_subcategory_id
```

3. Вывести на экран название категорий и количество товаров в данной категории.

```sql
select c.name, count(*) from production.product as p
join production.product_subcategory as s
on p.product_subcategory_id = s.product_subcategory_id
join production.product_category as c
on s.product_category_id = c.product_category_id
group by c.name
```

 4. Вывести на экран название подкатегории, а также количество товаров в данной подкатегории с учетом ситуации, что могут существовать подкатегории с одинаковыми именами.

    ```sql
    select distinct s.product_subcategory_id, s.name, count(*) from production.product as p
    join production.product_subcategory as s
    on p.product_subcategory_id = s.product_subcategory_id
    group by s.name, s.product_subcategory_id
    ```
 5. Вывести на экран название первых трех подкатегорий с небольшим количеством товаров.

    ```sql
    select s.name, count(*) from production.product as p
    join production.product_subcategory as s
    on p.product_subcategory_id = s.product_subcategory_id
    group by s.name
    order by count asc
    limit 3
    ```

    ```sql
    select s.name,count(*) from production.product as p
    join production.product_subcategory as s
    on p.product_subcategory_id=s.product_subcategory_id
    group by s.name
    order by count(*) asc
    fetch next 3 rows with ties
    ```
 6. Вывести на экран название подкатегории и максимальную цену продукта с цветом Red в этой подкатегории.

    ```sql
    select s.name, max(p.list_price) from production.product as p
    join production.product_subcategory as s
    on p.product_subcategory_id = s.product_subcategory_id
    where color='Red'
    group by s.name
    ```
 7. Вывести на экран название поставщика и количество товаров, которые он поставляет.

    ```sql
    select v.name, count(*) from production.product as p
    join purchasing.product_vendor as pv
    on p.product_id = pv.product_id
    join purchasing.vendor as v
    on pv.business_entity_id = v.business_entity_id
    group by v.name
    ```
    
 8. Вывести на экран название товаров, которые поставляются более чем одним поставщиком.

    ```sql
    select p.name, count(v.name) from production.product as p
    join purchasing.product_vendor as pv
    on p.product_id = pv.product_id
    join purchasing.vendor as v
    on pv.business_entity_id = v.business_entity_id
    group by p.name
    having count(v.name)>1
    ```
 9. Вывести на экран название самого продаваемого товара.

    ```sql
    select p.name,count(*) from production.product as p
    join sales.sales_order_detail as soh
    on p.product_id = soh.product_id
    group by p.name
    order by count(*) desc
    limit 1
    ```
10. Вывести на экран название категории, товары из которой продаются наиболее активно.

    ```sql
    select c.name,count(*) from production.product_category as c
    join production.product_subcategory as s
    on c.product_category_id=s.product_category_id
    join production.product as p
    on s.product_subcategory_id=p.product_subcategory_id
    join sales.sales_order_detail as sod
    on sod.product_id = p.product_id
    group by c.name
    order by count(*) desc
    limit 1
    ```
11. Вывести на экран названия категорий, количество подкатегорий и количество товаров в них.

    ```sql
    select c.name, count(distinct p.product_id), count(distinct s.product_subcategory_id) from production.product_category as c
    join production.product_subcategory as s
    on c.product_category_id = s.product_category_id
    join production.product as p
    on s.product_subcategory_id = p.product_subcategory_id
    group by c.name
    ```
12. Вывести на экран номер кредитного рейтинга и количество товаров, поставляемых компаниями, имеющими этот кредитный рейтинг.

    ```sql
    select v.credit_rating, count(*) from purchasing.vendor as v
    join purchasing.product_vendor as pv
    on v.business_entity_id = pv.business_entity_id
    join production.product as p
    on p.product_id = pv.product_id
    group by v.credit_rating
    ```
13. Найти первые 10 процентов самых дорогих товаров, с учетом ситуации, когда цены у некоторых товаров могут совпадать.

    ```sql
    WITH ranked_products AS (
        SELECT
            name,
            list_price,
            ROW_NUMBER() OVER (ORDER BY list_price DESC) AS row_num,
            COUNT(*) OVER () AS total_rows
        FROM
            production.product
    )
    SELECT
        name,
        list_price
    FROM
        ranked_products
    WHERE
        row_num <= total_rows * 0.10
    ORDER BY
        list_price DESC;
    ```
14. Найти первых трех поставщиков, отсортированных по количеству поставляемых товаров, с учетом ситуации, что количество поставляемых товаров может совпадать для разных поставщиков.

    ```sql
    select v.name, pv.business_entity_id, count(*) from purchasing.vendor as v
    join purchasing.product_vendor as pv
    on v.business_entity_id = pv.business_entity_id
    join production.product as p
    on p.product_id=pv.product_id
    group by v.name, pv.business_entity_id
    order by count(*) desc
    fetch next 3 rows with ties
    ```
15. Найти для каждого поставщика количество подкатегорий продуктов, к которым относится продукты, поставляемые им, без учета ситуации, когда продукт не относится ни к какой подкатегории.

    ```sql
    select v.name, v.business_entity_id, count(distinct p.product_subcategory_id) from purchasing.vendor as v
    join purchasing.product_vendor as pv
    on v.business_entity_id=pv.business_entity_id
    join production.product as p
    on p.product_id=pv.product_id
    join production.product_subcategory as s
    on s.product_subcategory_id = p.product_subcategory_id
    where p.product_id is not null
    group by v.name, v.business_entity_id
    ```
16. Проверить, есть ли продукты с одинаковым названием, если есть, то вывести эти названия. (Решение через JOIN)

    ```sql
    SELECT P1.name, P2.name
    FROM production.product AS P1
    JOIN production.product AS P2
    ON P1.name = P2.name
    where p1.name=p2.name
    GROUP BY P1.name, P2.name
    order by P1.name ASC;
    ```

   
