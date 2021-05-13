# Домашнее задание к занятию "6.3. MySQL"

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/tree/master/additional/README.md).

## Задача 1

Используя docker поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-03-mysql/test_data) и 
восстановитесь из него.

Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h` получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из ее вывода версию сервера БД.

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

**Приведите в ответе** количество записей с `price` > 300.

В следующих заданиях мы будем продолжать работу с данным контейнером.

## Решение задачи 1
`
sudo docker run --name test_mysql -v /my/data:/var/lib/mysql -v /my/bckp:/var/backups -e MYSQL_ROOT_PASSWORD=123456 -d mysql:latest

sudo docker exec -it  test_mysql bash

mysql -u root -p123456

mysql> status
--------------
mysql  Ver 8.0.25 for Linux on x86_64 (MySQL Community Server - GPL)

Connection id:          21
Current database:       test_db
Current user:           root@localhost
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server version:         8.0.25 MySQL Community Server - GPL
Protocol version:       10
Connection:             Localhost via UNIX socket
Server characterset:    utf8mb4
Db     characterset:    utf8mb4
Client characterset:    latin1
Conn.  characterset:    latin1
UNIX socket:            /var/run/mysqld/mysqld.sock
Binary data as:         Hexadecimal
Uptime:                 16 min 42 sec

Threads: 2  Questions: 64  Slow queries: 0  Opens: 163  Flush tables: 3  Open tables: 81  Queries per second avg: 0.063
--------------


mysql> show tables;
+-------------------+
| Tables_in_test_db |
+-------------------+
| orders            |
+-------------------+
1 row in set (0.01 sec)


mysql> select * from orders where price>300;
+----+----------------+-------+
| id | title          | price |
+----+----------------+-------+
|  2 | My little pony |   500 |
+----+----------------+-------+
1 row in set (0.00 sec)
`
## Задача 2

Создайте пользователя test в БД c паролем test-pass, используя:
- плагин авторизации mysql_native_password
- срок истечения пароля - 180 дней 
- количество попыток авторизации - 3 
- максимальное количество запросов в час - 100
- аттрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James"

Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.
    
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES получите данные по пользователю `test` и 
**приведите в ответе к задаче**.

## Решение задачи 2
`
CREATE USER 'test'@'localhost' 
   ATTRIBUTE '{"fname": "Pretty", "lname": "James"}';
   IDENTIFIED WITH mysql_native_password BY 'test-pass'  
   PASSWORD EXPIRE INTERVAL 180 DAY   
   FAILED_LOGIN_ATTEMPTS 3 PASSWORD_LOCK_TIME 2  
   WITH MAX_QUERIES_PER_HOUR 100;

mysql> GRANT select ON test_db.orders TO 'test'@'localhost';
mysql> FLUSH PRIVILEGES;


mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES  WHERE USER = 'test' AND HOST = 'localhost';
+------+-----------+---------------------------------------+
| USER | HOST      | ATTRIBUTE                             |
+------+-----------+---------------------------------------+
| test | localhost | {"fname": "Pretty", "lname": "James"} |
+------+-----------+---------------------------------------+
1 row in set (0.00 sec)
`
## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`
- на `InnoDB`

## Решение задачи 3
`
mysql> SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES where TABLE_SCHEMA = 'test_db';
+------------+--------+
| TABLE_NAME | ENGINE |
+------------+--------+
| orders     | InnoDB |
+------------+--------+
1 row in set (0.00 sec)

mysql> ALTER TABLE orders ENGINE = MyISAM;
Query OK, 5 rows affected (0.11 sec)
Records: 5  Duplicates: 0  Warnings: 0


mysql> SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES where TABLE_SCHEMA = 'test_db';
+------------+--------+
| TABLE_NAME | ENGINE |
+------------+--------+
| orders     | MyISAM |
+------------+--------+
1 row in set (0.00 sec)

mysql> SHOW PROFILES;
+----------+------------+-----------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                                   |
+----------+------------+-----------------------------------------------------------------------------------------+
|        1 | 0.01641875 | show engines                                                                            |
|        2 | 0.00020875 | SHOW ENGINES                                                                            |
|        3 | 0.00062200 | SELECT orders, ENGINE FROM information_schema.TABLES where TABLE_SCHEMA = 'test_db'     |
|        4 | 0.00375000 | SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES where TABLE_SCHEMA = 'test_db' |
|        5 | 0.11336500 | ALTER TABLE orders ENGINE = MyISAM                                                      |
|        6 | 0.00092125 | SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES where TABLE_SCHEMA = 'test_db' |
+----------+------------+-----------------------------------------------------------------------------------------+
6 rows in set, 1 warning (0.00 sec)
`


## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):
- Скорость IO важнее сохранности данных
- Нужна компрессия таблиц для экономии места на диске

- Размер буффера с незакомиченными транзакциями 1 Мб
- Буффер кеширования 30% от ОЗУ
- Размер файла логов операций 100 Мб

Приведите в ответе измененный файл `my.cnf`.

## Решение задачи 4
`
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL

innodb_log_buffer_size=1M

innodb_log_file_size=100M

innodb_buffer_pool_size=333M

innodb_file_per_table=1

innodb_flush_log_at_trx_commit=2
`