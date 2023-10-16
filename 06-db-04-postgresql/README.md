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

Предложите SQL-транзакцию для проведения этой операции.

Можно ли было изначально исключить ручное разбиение при проектировании таблицы orders?

## Задача 4

Используя утилиту `pg_dump`, создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

