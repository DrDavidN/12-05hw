# 12.5 «Индексы» - Дрибноход Давид

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

```sql
select	ROUND(SUM(index_length)*100/SUM(data_length)) 'size index in %'
from	INFORMATION_SCHEMA.TABLES
```

### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

Оконнная функция содержит столбец f.title из таблицы которая присоединена без условий, что не правильно и не дает результата в выводе кроме увеличения времени обработки такого запроса
Если ограничится той информацией которую выводят в selecte то таблицу film и inventory необходимо исключить, получится следущий запрос, который вернет тоже количество строк что и первоначальный запрос, но за более короткое время 54ms против 52000ms в первоначальном запросе
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id)
from payment p, rental r, customer c
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id
```
однако, если в запросе хотели получить данные по клиентам в разрезе фильмов то запрос следовало бы доработать следующим образом, такой запрос бцдет обработан за 74ms
при этом мы добавляем условие выборки данных по таблице film и выводим в select дополнительный столбец f.title
```sql
select distinct concat(c.last_name, ' ', c.first_name), f.title, sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id and f.film_id = i.film_id 
```

### Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

Bitmap index, Partial index, Function based index
источник https://habr.com/ru/articles/102785/
