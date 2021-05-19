# Домашнее задание к занятию "6.4. PostgreSQL"

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL используя `psql`.

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:
- вывода списка БД
- подключения к БД
- вывода списка таблиц
- вывода описания содержимого таблиц
- выхода из psql

## Решение Задачи 1

1 вывод списка БД  `\l`

2 подключение к БД `\c`

3 вывод списка таблиц `\dt`

4 вывод описания содержимого таблиц`\d`

5 выход из psql `\q`


```
sudo docker run --name my_postgres1 -e POSTGRES_PASSWORD=123456 -d -p 5432:5432 -v /postgres/data:/var/lib/postgresql/data -v /postgres/bckp:/var/backups postgres

sudo docker exec -it my_postgres1 bash

root@a9d88a586ff8:/# psql -U postgres
```

## Задача 2

Используя `psql` создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления и полученный результат.


## Решение задачи 2

```
postgres=# CREATE DATABASE test_database;

root@a9d88a586ff8:/# psql test_database < test_dump.sql -U postgres

postgres=# \c test_database
You are now connected to database "test_database" as user "postgres".
test_database=# \l
                                   List of databases
     Name      |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
---------------+----------+----------+------------+------------+-----------------------
 postgres      | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0     | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
               |          |          |            |            | postgres=CTc/postgres
 template1     | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
               |          |          |            |            | postgres=CTc/postgres
 test_database | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 test_db       | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =Tc/postgres         +
               |          |          |            |            | postgres=CTc/postgres
(5 rows)

test_database=# ANALYZE VERBOSE orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE
```
выборка по столбцу 

SELECT * FROM pg_stats where tablename = 'orders';

столбец attname "title" avg_width 16 байт


## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили
провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

Предложите SQL-транзакцию для проведения данной операции.

Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?

## Решение задачи 3

```
BEGIN TRANSACTION
CREATE TABLE orders_new ( id      SERIAL NOT NULL,    title   varchar (80),    price   int4) PARTITION BY RANGE (price);
CREATE TABLE orders_1    partition of orders_new   for values from (499) to (MAXVALUE);
CREATE INDEX ON orders_1 (price);
CREATE TABLE  orders_2    partition of orders_new   for values from (0) to (499);
CREATE INDEX ON orders_2 (price);
INSERT INTO orders_new (select * FROM orders);  
DROP TABLE orders;
ALTER TABLE  orders_new RENAME TO orders;
COMMIT;
```

## Задача 4

Используя утилиту `pg_dump` создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

## Решение задачи 4

1. pg_dump test_database > test1.sql -U postgres

2. нужно прописать ALTER TABLE orders ADD UNIQUE (title);


