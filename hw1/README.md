## SQL и реляционные СУБД. Введение в PostgreSQL 

1. Создал виртуальную машину с  ubuntu 20 на борту с именем postgres в проекте arctic-welder-300508 в GCP  
2. Накатил туда postgres12 с помощью команды

`apt update && apt instal postgresql`

3. Зашел в двух пользовательских сессиях в psql из-под юзера postgres:

`sudo -u postgres psql`

4. Выключил auto_commit:

```
postgres=# \set AUTOCOMMIT off
postgres=# \echo :AUTOCOMMIT
off
```

5. Создал таблицу с данными:

```
create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
```

Результат:

```
CREATE TABLE
INSERT 0 1
INSERT 0 1
WARNING:  there is no transaction in progress
COMMIT
```

6. текущий уровень изоляции: show transaction isolation level

Результат:
```
postgres=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 row)
```

7. Начинаем новую транзакцию в обоих сессиях: 
BEGIN;

В первой сессии выполняем 

```
insert into persons(first_name, second_name) values('sergey', 'sergeev');
```

Во второй сессии получаем
```
postgres=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)
```

А после завершения транзакции:

```
postgres=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```

Транзакция завершилась, запись в базу произведена, мы имеем доступ к новой записи в таблице только теперь, так как уровень изхоляции read committed.

8. Теперь установим уровень изоляции в repeatable read

```
set transaction isolation level repeatable read;
```

9. Начнем новые транзакции в обоих сессиях и в первой сессии запишем новые данные в таблицу:

```
insert into persons(first_name, second_name) values('sveta', 'svetova');
```

До завершения транзакции мы не увидим новых данных в таблице, так как, цитата из доки Postgre:

 В режиме Repeatable Read видны только те данные, которые были зафиксированы до начала транзакции, но не видны незафиксированные данные и изменения, произведённые другими транзакциями в процессе выполнения данной транзакции.

 После завершения транзакции в первой сессии: мы видим:

```
 postgres=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
  4 | sveta      | svetova
(4 rows)
```