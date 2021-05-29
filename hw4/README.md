## Homework 4 logical level

1. создайте новый кластер PostgresSQL 13 (на выбор - GCE, CloudSQL) 
Создал инстанс и установил postgrsql  
2. зайдите в созданный кластер под пользователем postgres 

```
root@hw4:~# su - postgres
postgres@hw4:~$ psql
```
3. создайте новую базу данных testdb 

`postgres=# create database testdb;`

4. зайдите в созданную базу данных под пользователем postgres 

```
postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".
testdb=#
```
5. создайте новую схему testnm 

`testdb=# create schema testnm;`
6. создайте новую таблицу t1 с одной колонкой c1 типа integer 

`testdb=# create table t1 (c1 integer);`
7. вставьте строку со значением c1=1 

```
testdb=# insert into t1(c1) values(1);
INSERT 0 1
```
8. создайте новую роль readonly 

```
testdb=# create role readonly;
CREATE ROLE
```
9. дайте новой роли право на подключение к базе данных testdb 

```
grant connect on database testdb TO readonly;
```
10. дайте новой роли право на использование схемы testnm 

```
grant usage on schema testnm to readonly;
```
11. дайте новой роли право на select для всех таблиц схемы testnm 

```
testdb=# grant select on all tables in schema testnm to readonly;
GRANT
```
12. создайте пользователя testread с паролем test123 

```
create user testread with password 'test123';
```
13. дайте роль readonly пользователю testread 

```
testdb=# grant readonly to testread;
```
14. зайдите под пользователем testread в базу данных testdb 

```
\c testdb testread - вроде должно быть так, но не получалось подключиться, почучал ошибку:

FATAL:  Peer authentication failed for user "testread"

Соответственно, для 127.0.0.1 поменял тип аутентикации за md5и завевлось.
```

15. сделайте select * from t1; 
16. получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже) 
17. напишите что именно произошло в тексте домашнего задания 
Не получилось:
```
testdb=> select * from t1;
ERROR:  permission denied for table t1
```
18. у вас есть идеи почему? ведь права то дали? 

```
testdb-> \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | t1   | table | postgres
(1 row)

таблица в схеме public, а мы пытаемся селектить из нее.
```
19. посмотрите на список таблиц 
20. подсказка в шпаргалке под пунктом 20 
21. а почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально) 
22. вернитесь в базу данных testdb под пользователем postgres 

```
 \c testdb
You are now connected to database "testdb" as user "postgres".
```
23. удалите таблицу t1 

```
testdb=# drop table t1;
DROP TABLE
```
24. создайте ее заново но уже с явным указанием имени схемы testnm 

```
create table testnm.t1(c1 integer);
```
25. вставьте строку со значением c1=1 

```
insert into testnm.t1 values(1);
```
26. зайдите под пользователем testread в базу данных testdb 
27. сделайте select * from testnm.t1; 
28. получилось?  

`Не получилось. Оказывается доступ выдался только на существовавшие таблицы. `
29. есть идеи почему? если нет - смотрите шпаргалку 
30. как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку 

Подсмотрел в шпаргалке. Оказывается

\c testdb postgres; alter default privileges in schema testnm grant select on tables to readonly; \c testdb testread;

31. сделайте select * from testnm.t1; 
32. получилось? 
33. есть идеи почему? если нет - смотрите шпаргалку 
31. сделайте select * from testnm.t1; 
32. получилось? 
33. ура! 

```
select * from testnm.t1;
 c1
----
  1
(1 row)
```
34. теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2); 
35. а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly? 
36. есть идеи как убрать эти права? если нет - смотрите шпаргалку 

Предположил, что search_path по умолчанию public, посмотрел в шпаргалку - ага, так и есть. Но можно сделать revoke на создание в схеме public.  
37. если вы справились сами то расскажите что сделали и почему, если смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды 
38. теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2); 
39. расскажите что получилось и почему 
После revoke, прав на создание табоицы в дефолтной схеме public не будет.
