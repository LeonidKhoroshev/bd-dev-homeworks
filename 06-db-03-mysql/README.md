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

```


Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h`, получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из её вывода версию сервера БД.

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

**Приведите в ответе** количество записей с `price` > 300.

В следующих заданиях мы будем продолжать работу с этим контейнером.

## Задача 2

Создайте пользователя test в БД c паролем test-pass, используя:

- плагин авторизации mysql_native_password
- срок истечения пароля — 180 дней 
- количество попыток авторизации — 3 
- максимальное количество запросов в час — 100
- аттрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James".

Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.
    
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES, получите данные по пользователю `test` и 
**приведите в ответе к задаче**.

## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`,
- на `InnoDB`.

## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):

- скорость IO важнее сохранности данных;
- нужна компрессия таблиц для экономии места на диске;
- размер буффера с незакомиченными транзакциями 1 Мб;
- буффер кеширования 30% от ОЗУ;
- размер файла логов операций 100 Мб.

Приведите в ответе изменённый файл `my.cnf`.

---

### Как оформить ДЗ

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

