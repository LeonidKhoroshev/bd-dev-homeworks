# Домашнее задание к занятию 3. «MySQL» - Леонид Хорошев

## Подготовка

Домашнее задание выполнено на виртуальной машине Centos7, развернутой в VirtualBox.

1.Создание директории для выполнения домашнего задания:
```
mkdir mysql_homework
cd mysql_homework
```

2. Установка docker:
```
su root
yum update -y && sudo dnf upgrade -y
yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io -y
systemctl start docker.service
systemctl enable docker.service
systemctl status docker
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-03-mysql/mysql/mysql1.png)

3. Установка docker-compose:
```
curl -L "https://github.com/docker/compose/releases/download/v2.22.0/docker-compose-linux-x86_64" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose -v
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-03-mysql/mysql/mysql2.png)



## Задача 1

Используя Docker, поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.
1. Создаем файл docker-compose.yml:
```
nano docker-compose.yml
```

```
version: '3.8'

volumes:
  data: {}
  backup: {}

services:
  mysql-server-80:
      image: mysql/mysql-server:8.0
      container_name: mysql
      ports:
        - "3308:3306"
      volumes:
        - ./docker-entrypoint-initdb.d/:/docker-entrypoint-initdb.d/
      environment:
        MYSQL_USER: "admin"
        MYSQL_PASSWORD: "admin"
```

2. Проверяем корректность конфигурации:
```
docker-compose -f docker-compose.yml config
```

3. Запускаем контейнер и проверяем результат:
```
docker-compose up -d
docker ps -a
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-03-mysql/mysql/mysql3.png)

4. Скачиваем бэкап базы данных:
```
 wget https://github.com/netology-code/virt-homeworks/blob/virt-11/06-db-03-mysql/test_data/test_dump.sql
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-03-mysql/mysql/mysql4.png)

5. Копируем бэкап `test_dump.sql` в созданный контейнер docker:
```
docker exec -it dd1f4a465695 bash
mkdir etc/mysql
exit
docker cp test_dump.sql mysql://etc/mysql/backup
```

6. Создаем базу данный в контейнере:
```
docker exec -it dd1f4a465695 bash
mysql -u root -p 
CREATE DATABASE test_db;
exit
```

7. Восстанавливаем дамп базы данных:
```
mysql -u root -p test_db < etc/mysql/backup
mysql -u root -p
```
```sql
show databases;
use test_db;
show tables;
```
Дамп восстановлен успешно:
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-03-mysql/mysql/mysql5.png)

8. Используя команду `\h`, получите список управляющих команд:
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-03-mysql/mysql/mysql6.png)

9. Найдите команду для выдачи статуса БД и **приведите в ответе** из её вывода версию сервера БД:
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-03-mysql/mysql/mysql7.png)

10. **Приведите в ответе** количество записей с `price` > 300:
```sql
select price from orders where price > 300;
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-03-mysql/mysql/mysql8.png)


## Задача 2

1. Создаем пользователя `test` в БД c паролем test-pass, используя:

- плагин авторизации mysql_native_password:
```sql
create user 'test'@'localhost' identified by 'test-pass':
```
- срок истечения пароля — 180 дней:
```sql
alter user 'test'@'localhost' password expire interval 180 day;
```
- количество попыток авторизации — 3 (при некорректном вводе пароля 3 раза подряд учетная запись пользователя `test` блокируется на 1 день):
```sql
alter user 'test'@'localhost' failed_login_attempts 3 password_lock_time 1;
```
- максимальное количество запросов в час — 100:
```sql
alter user 'test'@'localhost' with max_queries_per_hour 100;
```
- аттрибуты пользователя:
    - Фамилия "Pretty";
    - Имя "James".
```sql
alter user 'test'@'localhost' attribute '{"fname":"James", "lname":"Pretty"}';
```

2. Предоставляем привелегии пользователю `test` на операции SELECT базы `test_db`:

```sql
grant select on test_db.* to 'test'@'localhost';
show grants for 'test'@'localhost';
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-03-mysql/mysql/mysql9.png)

3. Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES, получаем данные по пользователю `test`:
```sql
select * from information_schema.user_attributes where user = 'test';
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-03-mysql/mysql/mysql10.png)

## Задача 3

1. Устанавливаем профилирование `SET profiling = 1` (имеется ввиду, что для управления профилированием должна использоваться переменная сеанса profiling, значение которой по умолчанию равно 0 (OFF), для включения профилирования необходимо установить значение переменной profiling  1 (ON):
```sql
set profiling = 1
```
2. Изучаем вывод профилирования команд `SHOW PROFILES;`:
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-03-mysql/mysql/mysql11.png)
В представленной выше таблице содержится информация о скорости выполнения различных запросов, так проанализируем самые простые и популярные запросы относительно их быстродействия:
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-03-mysql/mysql/mysql12.png)
Поскольку наша база данных практически пуста, все запросы выполняются почти мгновенно. Почему-то замый долгий (относительно конечно) был вывод всех таблиц БД `show tables`, а вот вывод данных из таблицы `select * from orders` занал практически в 1,5 раза меньшек времени. 

3. Исследуем, какой `engine` используется в таблице БД `test_db`:
```sql
show table status;
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-03-mysql/mysql/mysql13.png)
Поскольку у нас одна таблица, то вариантов формирования запроса не так много. Из представленной информации фидно, что в нашем случае используется InnoDB.

Изменим `engine` и **приведем время выполнения и запрос на изменения из профайлера в ответе** (видимо имеется ввиду запрос `select price from orders where price > 300`):
- на `MyISAM`:
```sql
alter table orders engine = MyISAM;
select price from orders where price > 300;
show profiles;
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-03-mysql/mysql/mysql13.png)
Время выполнения запроса - 0.0008 секунды.

- на `InnoDB`;
```sql
alter table orders engine = InnoDB;
select price from orders where price > 300;
show profiles;
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-03-mysql/mysql/mysql14.png)
Время выполнения запроса - 0.0006 секунды.

## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.
```sql
find / -name my.cnf
nano  /var/lib/docker/overlay2/08e0d1fa085b9a8cc9f3431715204190292b92560c8bf37a2cf393e527f6e61e/diff/etc/my.cnf
```
Измените его согласно ТЗ (движок InnoDB):
- скорость IO важнее сохранности данных (параметр innodb_flush_log_at_trx_commit определяет, как MySQL  будет писать в лог на диске данные о транзакциях и имеет 3 значения: 0, 1, 2, при выборе значения 0 лог сбрасывается на диск один раз в секунду, вне зависимости от транзакций. Скорость записи возрастает, но так же растёт риск потери данных);
```
innodb_flush_log_at_trx_commit = 0
```

- нужна компрессия таблиц для экономии места на диске (включаем параметр innodb_file_per_table, он позваляет сохранять вновь созданные таблицы в отдельные файлы);
```
innodb_file_per_table = 1
```
  
- размер буффера с незакомиченными транзакциями 1 Мб (в параметре Innodb_log_buffer_size необходимо задать требуемое значение размера буфера транзакций);
```
innodb_log_buffer_size = 1M
```

- буффер кеширования 30% от ОЗУ (чтбоы правильно настроить параметр innodb_buffer_pool_size, требуется узнать одщий объем ОЗУ, выходим из режима редактирования файла и смотрим вывод команды `free -h`);
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-03-mysql/mysql/mysql15.png)
```
innodb_buffer_pool_size = 1,4G
```
- размер файла логов операций 100 Мб (innodb_log_file_size устанавливает размер лога операций. Операции сначала записываются в лог, а потом применяются к данным на диске. Чем больше этот лог, тем быстрее будут работать записи).
```
innodb_log_file_size = 100M
```
Приведите в ответе изменённый файл `my.cnf`.
```
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/8.0/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M

# Remove leading # to revert to previous value for default_authentication_plugin,
# this will increase compatibility with older clients. For background, see:
# https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_default_authentication_plugin
# default-authentication-plugin=mysql_native_password
skip-host-cache
skip-name-resolve
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
secure-file-priv=/var/lib/mysql-files
user=mysql

pid-file=/var/run/mysqld/mysqld.pid
innodb_flush_log_at_trx_commit = 0
innodb_file_per_table = 1
innodb_log_buffer_size = 1M
innodb_buffer_pool_size = 1,4G
innodb_log_file_size = 100M
```
