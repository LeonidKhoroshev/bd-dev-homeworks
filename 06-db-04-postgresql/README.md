# Домашнее задание к занятию 4. «PostgreSQL» - Хорошев Леонид

## Подготовка

Домашнее задание выполнено на виртуальной машине Centos7, развернутой в VirtualBox.

1.Создание директории для выполнения домашнего задания:
```
mkdir pgqsl_homework
cd pgqsl_homework
```

2. Установка docker
```
su root
yum update -y && sudo dnf upgrade -y
yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io -y
systemctl start docker.service
systemctl enable docker.service
systemctl status docker
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-04-postgresql/postgres/pgsql1.png)

3. Установка docker-compose
```
curl -L "https://github.com/docker/compose/releases/download/v2.22.0/docker-compose-linux-x86_64" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose -v
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-04-postgresql/postgres/pgsql2.png)

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.
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
    image: postgres:13
    container_name: psql
    ports:
      - "0.0.0.0:5432:5432"
    volumes:
      - data:/var/lib/postgresql/data
      - backup:/media/postgresql/backup
    environment:
      POSTGRES_USER: "admin"
      POSTGRES_PASSWORD: "admin"
```

2. Запуск docker-compose.yml
```
docker-compose up -d
docker ps -a
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-04-postgresql/postgres/pgsql3.png)

Подключитесь к БД PostgreSQL, используя `psql`.
```
docker exec -it 61b71ecf21d4 bash
psql -h 127.0.0.1 -U admin
```

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:

- вывода списка БД,
```sql
\l
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-04-postgresql/postgres/psql4.png)

- подключения к БД,
```sql
\c postgres
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-04-postgresql/postgres/psql5.png)

- вывода списка таблиц,
```
\dt
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-04-postgresql/postgres/psql6.png)
Поскольку в базе данных отсутствуют таблицы, вывод пустой.

- вывода описания содержимого таблиц,
```sql
show * from table_name;
```
вместо table_name указываем название интересующей таблицы.

- выхода из psql.
```sql
\q
```  

## Задача 2

Используя `psql`, создайте БД `test_database`.
```sql
create database test_database;
\q
```

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-04-postgresql/test_data).

1. Скачиваем дамп базы данных:
```
wget https://github.com/netology-code/virt-homeworks/blob/virt-11/06-db-04-postgresql/test_data/test_dump.sql
```

2. Копируем дамп базы данных в контейнер:
```
docker cp test_dump.sql psql://backup
```

Восстановите бэкап БД в `test_database`.
```
docker exec -it 61b71ecf21d4 bash
psql -U admin test_database < backup
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-04-postgresql/postgres/psql7.png)

Перейдите в управляющую консоль `psql` внутри контейнера.
```
psql -U admin -d test_database
\dt
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-04-postgresql/postgres/psql8.png)

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.
```sql
analyze verbose public.orders;
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-04-postgresql/postgres/psql9.png)

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.
```sql
select avg_width from pg_stats where tablename='orders';
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-04-postgresql/postgres/psql10.png)
По выводу команды получается, что наибольшее среднее значение размера элементов в байтах во втором столбце. Посмотрим, почему:

![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-04-postgresql/postgres/psql11.png)

Во втором столбце приведены название товара.



## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам как успешному выпускнику курсов DevOps в Нетологии предложили
провести разбиение таблицы на 2: шардировать на orders_1 - price>499 и orders_2 - price<=499.

1. Создаем 2 таблицы - orders_1 и orders_2:
```sql
create table orders_1 (
id bigint not null,
title text not null,
price bigint not null);
```

```
create table orders_2 (
id bigint not null,
title text not null,
price bigint not null);
```

2. Наполняем их данными, исходя из условий задания - поместим заказы с ценой более 499 в первую таблицу, а заказы сос тоимостью менее, либо равной 499 - во вторую:

```sql
insert into orders_1
select * from orders
where price > 499;
```

```sql
insert into orders_2
select * from orders
where price <= 499;
```

3. Проверяем, что разбиение прошло корректно:

```sql
select * from orders_1;
select * from orders_2;
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-04-postgresql/postgres/psql12.png)

Поскольку данных крайне мало, можно проверить, что наши данные нигде не потерялись:
```sql
select * from orders;
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-04-postgresql/postgres/psql13.png)

Все прошло успешно, но при наличии больших объемов данных такая проверка - излишне.

4. Удаляем таблицу orders, чтобы исключить дублирование данных:

```sql
drop table orders;
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-04-postgresql/postgres/psql14.png)



Можно ли было изначально исключить ручное разбиение при проектировании таблицы orders?

Ручное разбиение возможно исключить создав правила записи данных в таблицы orders_1 и orders_2, более того, с учетом удаления исходной таблицы orders, так сделать необходимо:

```sql
create rule orders_1 as on insert to orders_1 where ( price > 499 ) do instead insert into orders_1 values (new.*);
create rule orders_2 as on insert to orders_2 where ( price <= 499 ) do instead insert into orders_2 values (new.*);
```

## Задача 4

Используя утилиту `pg_dump`, создайте бекап БД `test_database`.

```
export PGPASSWORD=admin && pg_dump -U admin test_database > /tmp/test_database_backup.sql
```

Пароль и пользователь использованы те же, что и при создании контейнера.

Проверяем, что бекап создан успешно:
```
ls /tmp
cat /tmp/test_database_backup.sql
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-04-postgresql/postgres/psql15.png)

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

Для доработки базы данных в соответствии с заданием, на этапе создания таблиц orders_1 и orders_2 необходимо было добаить индекс unique:
```sql
create table orders_1 (
id bigint not null,
title text not null, unique,
price bigint not null);
```
```sql

create table orders_2 (
id bigint not null,
title text not null, unique,
price bigint not null);
```

Также можно добавить данный индекс в существующие таблицы:
```sql
alter table orders_1 add  constraint constraintname unique (title);
alter table orders_2 add  constraint constraintname unique (title);
```
