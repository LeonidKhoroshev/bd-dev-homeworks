# Домашнее задание к занятию 2. «SQL» - Леонид Хорошев

## Подготовка к выполнению домашнего задания

Домашнее задание выполнено на виртуальной машине, развернутой в YandexCloud под управлением ОС AlmaLinux9.
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql.png)

1. Установка docker:
```
su root
dnf update -y && sudo dnf upgrade -y
dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
dnf install docker-ce docker-ce-cli containerd.io -y
systemctl start docker.service
systemctl enable docker.service
systemctl status docker
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql0.png)
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql1.png)

2. Установка docker-compose:
```
curl -L "https://github.com/docker/compose/releases/download/v2.22.0/docker-compose-linux-x86_64" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql2.png)

3. Создадим директорию для выполнения домашнего задания и перейдем в нее:
```
mkdir homework_sql
cd homework_sql
```

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.
```
nano docker-compose.yml
```

1. Docker-compose-манифест.
```
version: '3.8'

volumes:
  data: {}
  backup: {}

services:
  postgres:
    image: postgres:12
    container_name: psql
    ports:
      - "0.0.0.0:5432:5432"
    volumes:
      - data:/var/lib/postgresql/data
      - backup:/media/postgresql/backup
    environment:
      POSTGRES_USER: "admin"
      POSTGRES_PASSWORD: "admin"
      POSTGRES_DB: "test_db"
```
2. Запуск docker-compose.yml
```
docker-compose.yml
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql3.png)

3. Проверка статуса запущенного контейнера.
```
docker ps
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql4.png)

4. Вход в базу данных.
```
docker exec -it c4289e3dc75d bash
psql -h 127.0.0.1 -U admin -d test_db
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql5.png)

## Задача 2

В БД из задачи 1: 

1. Создание пользователя test-admin-user и БД test_db.
```sql
CREATE USER "test-admin-user";
```

2. Создание таблицы orders.
```sql
CREATE TABLE orders (
    id SERIAL,
    наименование VARCHAR, 
    цена INTEGER,
    PRIMARY KEY (id)
);
```sql

3. Создание таблицы clients.
```sql
CREATE TABLE clients (
    id SERIAL,
    фамилия VARCHAR,
    "страна проживания" VARCHAR, 
    заказ INTEGER,
    PRIMARY KEY (id)
);
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql6.png)

4. Предоставление привилегий на все операции пользователю test-admin-user на таблицы БД test_db;
```sql
GRANT ALL PRIVILEGES ON TABLE orders, clients to "test-admin-user";
```

5. Создание пользователя test-simple-user;
```sql
CREATE USER "test-simple-user";
```

6. Предоставление пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE таблиц БД test_db.
```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLE orders, clients to "test-simple-user";
```

7. Приведите:

- итоговый список БД после выполнения пунктов выше;
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql7.png)
- описание таблиц (describe);
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql8.png)
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db;
```sql
SELECT
    grantee, table_name, privilege_type
FROM
    information_schema.table_privileges
WHERE
    grantee in ('test-admin-user','test-simple-user')
    and table_name in ('clients','orders');
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql9.png)
- список пользователей с правами над таблицами test_db.
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql10.png)

## Задача 3

Используя SQL-синтаксис, наполните таблицы следующими тестовыми данными:

1. Таблица orders

|Наименование|цена|
|------------|----|
|Шоколад| 10 |
|Принтер| 3000 |
|Книга| 500 |
|Монитор| 7000|
|Гитара| 4000|
```sql
INSERT INTO orders VALUES (1, 'Шоколад', 10), (2, 'Принтер', 3000), (3, 'Книга', 500), (4, 'Монитор', 7000), (5, 'Гитара', 4000);
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql11.png)

Таблица clients

|ФИО|Страна проживания|
|------------|----|
|Иванов Иван Иванович| USA |
|Петров Петр Петрович| Canada |
|Иоганн Себастьян Бах| Japan |
|Ронни Джеймс Дио| Russia|
|Ritchie Blackmore| Russia|

```sql
INSERT INTO clients VALUES (1, 'Иванов Иван Иванович', 'USA'), (2, 'Петров Петр Петрович', 'Canada'), (3, 'Иоганн Себастьян Бах', 'Japan'), (4, 'Ронни Джеймс Дио', 'Russia'), (5, 'Ritchie Blackmore', 'Russia');
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql12.png)

Используя SQL-синтаксис:
- вычислите количество записей для каждой таблицы.

```sql
SELECT count(1) FROM orders;
SELECT count(1) FROM clients;
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql13.png)

## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys, свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения этих операций.
```sql
update clients
set заказ=(select id from orders where наименование='Книга')
where фамилия='Иванов Иван Иванович';
```

```sql
update clients
set заказ=(select id from orders where наименование='Монитор')
where фамилия='Петров Петр Петрович';
```

```sql
update clients
set заказ=(select id from orders where наименование='Гитара')
where фамилия='Иоганн Себастьян Бах';
```

![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql14.png)

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод этого запроса.

```sql
select фамилия, заказ
from clients
where заказ is not null;
```

![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql15.png)

Модернизируем вывод, чтобы увидеть не id заказа, а наименование заказанного товара:

```sql
SELECT фамилия, наименование
FROM clients
JOIN orders ON clients.заказ = orders.id;
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql16.png)


## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).
1. SQL-запрос для выдачи всех пользователей, которые совершили заказ с указанием id заказа

```sql
explain
select фамилия, заказ
from clients
where заказ is not null;
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql17.png)

2. SQL-запрос для выдачи всех пользователей, которые совершили заказ с указанием наименования заказанного товара.

```sql
explain
select фамилия, наименование
from clients
join orders ON clients.заказ = orders.id;
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql18.png)

Из анализа полученных значений видно, что оба запроса используют один и тот же метод последовательного чтения данных из таблицы clients (Seq Scan), но второй запрос "тяжелее" первого, он помимо Seq Scan использует еще и Hash (видимо при операции join).

Также "сложность" второго запроса выражается в большем количестве ожидаемого числв строк (rows) и большем их ожидаемом среднем размере (width).

В обоих случаях каждая запись сравнивается с условием "заказ" IS NOT NULL. Если условие выполняется, запись вводится в результат, если нет, то отбрасывается.

Запрос для выдачи всех пользователей, которые совершили заказ с указанием id заказа выглядит оптимальнее второго в части нагрузки на систему.


## Задача 6

1. Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).
```
pg_dump -U admin test_db > /media/postgresql/backup/test_db.backup
```

2. Остановите контейнер с PostgreSQL, но не удаляйте volumes.
```
docker ps -a
docker stop c4289e3dc75d
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql19.png)

3. Поднимите новый пустой контейнер с PostgreSQL.
Для этого не будем пользоваться compose файлом из задания 1, а выполним 
```
docker pull postgres
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql20.png)

```
docker run --name my-pg -e POSTGRES_PASSWORD=admin -d postgres
```

![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql21.png)


4. Восстановите БД test_db в новом контейнере.
Для этого мы коопируем наш бэкап из старого контейнера (psql) в новый (my-pg)
```
docker cp psql://media/postgresql/backup/test_db.backup backup
docker cp backup my-pg:/home/
```

Далее входим в новый контейнер и в нашу базу данных через бэкап.
```
docker exec -it f51cd6609d85 bash
psql -U postgres -f /home/backup/
```
Выполняем проверку:
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql22.png)

База данных с названием test_db отсутствует, нот есть template0.

Проверяем, что это:
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql23.png)

Название таблиц совпадает с test_db.

Проверяем данные таблиц:
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-02-sql/sql/sql24.png)

Бэкап прошел успешно.


