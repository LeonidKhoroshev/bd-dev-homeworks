# Домашнее задание к занятию 5. «Elasticsearch» - Леонид Хорошев

## Подготовка к выполнению домашнего задания

Домашнее задание выполнено на виртуальной машине Centos7, развернутой в VirtualBox. На данной виртуальной машине уже установлен [docker, docker-compose](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-04-postgresql/README.md) и выполнен ряд домашних заданий.

1. Создаем директорию для выполнения домашнего задания:
```
mkdir elastic_homework
cd elastic_homework
```

2. Регистрируемся на [dockerhub](https://hub.docker.com/) 
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-05-elasticsearch/elk/elk1.png)

3. Авторизуемся под созданной на dockerhub учетной записью.
```
docker login --username leonid1984
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-05-elasticsearch/elk/elk2.png)

## Задача 1

Используя Docker-образ [centos:7](https://hub.docker.com/_/centos) как базовый и 
[документацию по установке и запуску Elastcisearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html):

- составьте Dockerfile-манифест для Elasticsearch;
```
nano Dockerfile
```
При составлении Dockerfile-манифеста возник ряд сложностей, связанных с невозможностью использования официального репозитория Elasticsearch, поиск сторонних репозиториев также успехов не принес, либо там была нерабочая версия, либо я некорректно настроил конфигурацию, на всякий случай ссылку на репозиторий оставлю [тут](https://sourceforge.net/projects/elasticsearch.mirror/files/v8.10.4/).

Использование яндекс зеркала, счел "неспортивным", так как оно располагает только deb версией (по условиям задание требуется rpm), хотя судя по комментария м к домашнему заданию такой вариант решения также принимается к проверке.

В результате предприняты следующие действия:
- на хостовой ОС Windows 10 поднят [OpenVPN](https://teletype.in/@vpntype/for-windows)
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-05-elasticsearch/elk/elk6.png)
- c официального репозитория скачан [Elasticsearch](https://www.elastic.co/downloads/enterprise-search) версии 8.10.4 под rpm линейку
- скачанный архив залит на гугл [диск](https://drive.google.com/u/0/uc?id=1IgmcX3iJ-6JzAv5losWl3h_GmA1VkLKw&export=download&confirm=t&uuid=16e1a298-f7d3-4191-b6bc-e3eb4c0440a0&at=AB6BwCA57MoT-65WZGx5-F5TKEtc:1697961303399), ссылку тоже оставляю здесь, доступ к архиву открыл, если у других студентов будут вопросы, можно смело делиться. 

В результате получился следующий Dockerfile-манифест
```
FROM centos:7
EXPOSE 9200
LABEL maintainer = khoroshevlv@gmail.com
ENV TZ=Europe/Moscow
ENV ES_HOME="/var/lib/elasticsearch"
WORKDIR ${ES_HOME}
USER 0
RUN export ES_HOME="/var/lib/elasticsearch"
RUN yum install wget perl-Digest-SHA -y
RUN wget -O elasticsearch-8.10.4 "https://drive.google.com/u/0/uc?id=1IgmcX3iJ-6JzAv5losWl3h_GmA1VkLKw&export=download&confirm=t&uuid=16e1a298-f7d3-4191-b6bc-e3eb4c0440a0&at=AB6BwCA57MoT-6$
RUN tar -xzf elasticsearch-8.10.4
RUN useradd -m -u 1000 elasticsearch
RUN chown elasticsearch:elasticsearch -R ${ES_HOME}
RUN yum -y remove wget
RUN yum clean all
COPY elasticsearch.yml /var/lib/elasticsearch/elasticsearch-8.10.4/config/elasticsearch.yml

CMD ["sh", "-c", "/var/lib/elasticsearch/elasticsearch-8.10.4/bin/elasticsearch"]
USER 1000
```
Создаем elasticsearch.yml
```
nano elasticsearch.yml
```
```
network.host: 0.0.0.0
path.data: /var/lib/elasticsearch
http.port: 9200
discovery.type: single-node
node.name: netology_test
xpack.security.enabled: false
xpack.security.enrollment.enabled: false
```

- соберите Docker-образ и сделайте `push` в ваш docker.io-репозиторий,
```
docker build -t leonid1984/elasticsearch8 .
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-05-elasticsearch/elk/elk3.png)

```
docker push leonid1984/elasticsearch8
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-05-elasticsearch/elk/elk4.png)
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-05-elasticsearch/elk/elk5.png)

- запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины.
```
docker run -d -p 9200:9200 leonid1984/elasticsearch8
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-05-elasticsearch/elk/elk7.png)

Небольшой комментарий - сборка и запуск контейнера прошла далеко не с первого раза, поэтому каждаю новая команда в dockerfile начинается на RUN, правильнее с точки зрения оптимизации наверное использовать && в конце строки, но приведенная выше конфигурация позволяет быстрее выполнить поиск ошибок в dockerfile и их исправление.

```
curl -X GET "localhost:9200/?pretty"
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-05-elasticsearch/elk/elk8.png)


## Задача 2

Ознакомьтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
и добавьте в `Elasticsearch` 3 индекса в соответствии с таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

Индекс 1:
```
curl -X PUT "localhost:9200/ind-1?pretty" -H 'Content-Type: application/json' -d'
 {
   "settings": {
     "number_of_shards": 1,
     "number_of_replicas": 0
   }
 }
 '
```

Индекс2:
```
curl -X PUT "localhost:9200/ind-2?pretty" -H 'Content-Type: application/json' -d'
 {
   "settings": {
     "number_of_shards": 2,
     "number_of_replicas": 1
   }
 }
 '
```

Индекс3:
```
curl -X PUT "localhost:9200/ind-3?pretty" -H 'Content-Type: application/json' -d'
 {
   "settings": {
     "number_of_shards": 4,
     "number_of_replicas": 2
   }
 }
 '
```

Получите список индексов и их статусов, используя API, и **приведите в ответе** на задание.
```
curl -X GET 'http://localhost:9200/_cat/indices?v'
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-05-elasticsearch/elk/elk9.png)

Получите состояние кластера `Elasticsearch`, используя API.

Индекс 1:
```
curl -X GET 'http://localhost:9200/_cluster/health/ind-1?pretty'
```

![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-05-elasticsearch/elk/elk10.png)

Индекс 2:
```
curl -X GET 'http://localhost:9200/_cluster/health/ind-2?pretty'
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-05-elasticsearch/elk/elk11.png)

Индекс 3:
```
curl -X GET 'http://localhost:9200/_cluster/health/ind-3?pretty'
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-05-elasticsearch/elk/elk12.png)


Как вы думаете, почему часть индексов и кластер находятся в состоянии yellow?

Для корректной работы индексов (состояние green) в нашей конфигурации не хватило нод, для размещения всех реплик необходимо создать дополнительные 3 ноды.

Удалите все индексы.
```
curl -XDELETE http://localhost:9200/ind-1,ind-2,ind-3
```
![Alt text](https://github.com/LeonidKhoroshev/bd-dev-homeworks/blob/main/06-db-05-elasticsearch/elk/elk13.png)


**Важно**

При проектировании кластера Elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

## Задача 3

В этом задании вы научитесь:

- создавать бэкапы данных,
- восстанавливать индексы из бэкапов.

Создайте директорию `{путь до корневой директории с Elasticsearch в образе}/snapshots`.

Используя API, [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository) 
эту директорию как `snapshot repository` c именем `netology_backup`.

**Приведите в ответе** запрос API и результат вызова API для создания репозитория.

Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.

[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) 
состояния кластера `Elasticsearch`.

**Приведите в ответе** список файлов в директории со `snapshot`.

Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `Elasticsearch` из `snapshot`, созданного ранее. 

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.

Подсказки:

- возможно, вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить `Elasticsearch`.

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

