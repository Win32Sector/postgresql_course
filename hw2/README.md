Выполнил пункты:

создайте виртуальную машину c Ubuntu 20.04 LTS (bionic) в GCE типа e2-medium в default VPC в любом регионе и зоне, например us-central1-a

поставьте на нее PostgreSQL через sudo apt

проверьте что кластер запущен через sudo -u postgres pg_lsclusters

зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым 

`postgres=# create table test(c1 text); insert into test values('1'); \q`

остановите postgres например через `sudo -u postgres pg_ctlcluster 13 main stop`

создайте новый standard persistent диск GKE через Compute Engine -> Disks в том же регионе и зоне что GCE инстанс размером например 10GB

добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk

проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux

сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/

перенесите содержимое `/var/lib/postgres/13 в /mnt/data - mv /var/lib/postgresql/13 /mnt/data`
попытайтесь запустить кластер - `sudo -u postgres pg_ctlcluster 13 main start`


##Ответы на вопросы и *

Попытка запустить кластер была неуспешной, так как нужно поменять значение параметра `data_directory в /etc/postgresql/12/main/postgresql.conf и указать там data_directory = '/mnt/data/12/main'`

Проверка наличия данных дала положительный результат - данные на месте:

```
postgres=# select * from test;
 c1
----
 1
(1 row)
```

*

1. Создал новую виртуалку
2. Открепил диск от виртуальной машины instance-1, прикрепил ко второй, поставил на новую виртуалку postgresql
3. добавил диск по лейблу LABEL=data в /etc/fstab, создал каталог /mnt/data, сделал mount -a
4. зачовнил директорию chown -R postgres:postgres /mnt/data
5. остановил постгрес на второй виртуалке, поменял настройку data_directory в /etc/postgresql/12/main/postgresql.conf и указал там data_directory = '/mnt/data/12/main'
6. Завел постгресс обратно.
7. сделал селект:

```
postgres=# select * from test;
 c1
----
 1
(1 row)
```