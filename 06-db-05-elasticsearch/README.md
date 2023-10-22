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
``
FROM centos:7

LABEL maintainer = khoroshevlv@gmail.com

ENV TZ=Europe/Moscow
RUN adduser elasticsearch

RUN yum install wget  perl-Digest-SHA  -y
RUN wget --no-check-certificate [https://sourceforge.net/projects/elasticsearch.mirror/files/v8.10.4/](https://sourceforge.net/projects/elasticsearch.mirror/files/v8.10.4/)Elasticsearch%208.10.4%20source%20code.tar.gz/download
RUN tar -xzf download
RUN rm -f download
RUN yum -y remove wget
RUN yum clean all
RUN cd elastic-elasticsearch-8d0aed9/

COPY elasticsearch.yml /elastic-elasticsearch-8d0aed9/config/

RUN chown -R elasticsearch /elastic-elasticsearch-8d0aed9/
RUN mkdir /var/lib/elasticsearch/
RUN chown -R elasticsearch /var/lib/elasticsearch/

EXPOSE 9200

CMD /elastic-elasticsearch-8d0aed9/distribution/src/bin/elasticsearch

```
Создаем elasticsearch.yml
```
nano elasticsearch.yml
```

```
node.name: "netology_test"
cluster.name: my_cluster
node.roles: [ master ]
path.data: /var/lib/elasticsearch

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

Требования к `elasticsearch.yml`:

- данные `path` должны сохраняться в `/var/lib`,
- имя ноды должно быть `netology_test`.

В ответе приведите:

- текст Dockerfile-манифеста,
- ссылку на образ в репозитории dockerhub,
- ответ `Elasticsearch` на запрос пути `/` в json-виде.

Подсказки:

- возможно, вам понадобится установка пакета perl-Digest-SHA для корректной работы пакета shasum,
- при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml,
- при некоторых проблемах вам поможет Docker-директива ulimit,
- Elasticsearch в логах обычно описывает проблему и пути её решения.

Далее мы будем работать с этим экземпляром Elasticsearch.

## Задача 2

В этом задании вы научитесь:

- создавать и удалять индексы,
- изучать состояние кластера,
- обосновывать причину деградации доступности данных.

Ознакомьтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
и добавьте в `Elasticsearch` 3 индекса в соответствии с таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

Получите список индексов и их статусов, используя API, и **приведите в ответе** на задание.

Получите состояние кластера `Elasticsearch`, используя API.

Как вы думаете, почему часть индексов и кластер находятся в состоянии yellow?

Удалите все индексы.

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

