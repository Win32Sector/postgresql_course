## Homework 3
## Установка Postgres в контейнере Docker

1. Создал виртуалку в GCP
2. Обновил и поставил нужные пакеты:

```
apt update && apt install docker docker-compose
```
3. Добавим на машину docker-compose файл для запуска pg в контейнере и клиента

```
version: '3.1'


services:
  pg_db:
    container_name: pg_db
    image: postgres:13-alpine
    restart: always
    environment:
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_USER=postgres
    volumes:
      - /var/lib/postgres:/var/lib/postgresql
    ports:
      - 5432:5432

  pg_client:
    container_name: pg_client
    image: postgres:13-alpine
    environment:
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_USER=postgres
    restart: always
```

4. Зайдем через клиентский контейнер в базу в контейнере pg_db

```
docker exec -u postgres -ti pg_client psql -h pg_db
```

5. Создадим таблицу с парой строк:

```
postgres=# create table docker(image text);
CREATE TABLE
postgres=# insert into docker values('pg');
INSERT 0 1
postgres=# insert into docker values('nginx');
INSERT 0 1
postgres=# select * from docker;
 image
-------
 pg
 nginx
(2 rows)
```

6. Для того, чтобы подключиться извне нужно дать доступ через VPC Network -> Firewall, точнее сначала создаем правило для доступа по порту 5432 и цепляем его на тег postgres. После того, как правило создано, редактируем нашу виртуальную машину и вешаем на нее тег postgres. Через некоторое время на тачку можно цепляться на 5432 порт.

Далее подключаемся удаленно с ноутубка:

```
psql -h35.232.56.14 -U postgres -W
Password:
psql (13.2, server 13.3)
Type "help" for help.

postgres=# select * from docker;
 image
-------
 pg
 nginx
(2 rows)
```

7. убиваем контейнер, каталог в хостовой системе остается. Соответственно, если стартуем заново, то используется тот же каталог.

В чем были сложности? Вспомнить как писать docker-compose файлы и куда постгресс кладет свои файлы - /var/lib/postgresql