>создайте новый кластер PostgresSQL 14

__apt install postgresql-14__

>зайдите в созданный кластер под пользователем postgres

__sudo -u postgres psql__

>создайте новую базу данных testdb

__create database testdb;__

> зайдите в созданную базу данных под пользователем postgres
создайте новую схему testnm
оздайте новую таблицу t1 с одной колонкой c1 типа integer
7 вставьте строку со значением c1=1

* __\c testdb__
* __create schema testnm;__
* __create table t1(c1 int);__
* __insert into t1 values (1);__

>- создайте новую роль readonly
>- дайте новой роли право на подключение к базе данных testdb
>- дайте новой роли право на использование схемы testnm
>- дайте новой роли право на select для всех таблиц схемы testnm

* __create role readonly with password  'readonly';__
* __grant connect on database testdb to readonly;__
* __grant usage on schema testnm to readonly;__
* __grant select on all tables in schema testnm to readonly;__

>- создайте пользователя testread с паролем test123
>- дайте роль readonly пользователю testread

* __create user testread with password 'test123';__
* __grant readonly to testread;__

>- зайдите под пользователем testread в базу данных testdb
>- сделайте select * from t1;
>- получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)
>- напишите что именно произошло в тексте домашнего задания
>- у вас есть идеи почему? ведь права то дали?

* __psql -U testread -d testdb -h localhost__

<pre>Password for user testread: 
psql (14.7 (Ubuntu 14.7-0ubuntu0.22.04.1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type &quot;help&quot; for help.

testdb=&gt; 
</pre>

<pre>testdb=&gt; select * from t1;
ERROR:  permission denied for table t1
testdb=&gt;</pre>
Ошибка доступа к объекту. Через роль readonly мы передали права на схему testnm и таблицы, которые там находятся. Но так как 
<pre>testdb=# show search_path;
   search_path   
-----------------
 &quot;$user&quot;, public
(1 row)
</pre>
то созданная нами таблица автоматом создалась в схеме public, на данный объект права мы не грантили

>- удалите таблицу t1
>- создайте ее заново но уже с явным указанием имени схемы testnm
>- вставьте строку со значением c1=1
>- зайдите под пользователем testread в базу данных testdb
>- сделайте select * from testnm.t1;

* __drop table t1;__
* __create table testnm.t1(c1 int);__
* __select * from testnm.t1;__
<pre>testdb=&gt; select * from testnm.t1
testdb-&gt; ;
ERROR:  permission denied for table t1
testdb=&gt;</pre>

>есть идеи почему?

__данная таблица является новым объектом для СУБД(oid) на который мы еще права не выдавали. Проблему можно решить либо выдать таргетно права на данный объект(или на все объекты данного типа), либо поменять дефолтные права которые наследуются при создании какой-то ролью необходимого типа объекта, например: alter default privileges for role ... in schema ... grant ... on tables to readonly;__

>попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);

__данные комманды выполнились, так как объект создался в схеме public. По умолчания все могут создавать объекты(и вставлять в них значения так как при создании ты становишся owner-ом) и просматривать данную схему.__

__необходимо делать revoke хотя бы create привилегии: revoke create on schema public from public;__

>create table t3(c1 integer); insert into t2 values (2);

__комманда на создание таблицы не выполнилась, так как я отобрал привилегию на создание . Вторая комманда выплнилась успешно , так как роль testread является владельцем таблицы t2__





