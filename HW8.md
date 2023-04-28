> Установить на него PostgreSQL 15 с дефолтными настройками

* __sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'__
* __wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -__
* __sudo apt-get update__
* __apt install postgresql-15__

> Создать БД для тестов: выполнить pgbench -i test
Запустить pgbench -c8 -P 6 -T 60 -U postgres test

<pre>pgbench (15.2 (Ubuntu 15.2-1.pgdg22.04+1))
starting vacuum...end.
progress: 6.0 s, 420.5 tps, lat 18.885 ms stddev 11.411, 0 failed
progress: 12.0 s, 458.8 tps, lat 17.431 ms stddev 7.281, 0 failed
progress: 18.0 s, 455.2 tps, lat 17.561 ms stddev 7.765, 0 failed
progress: 24.0 s, 457.1 tps, lat 17.464 ms stddev 7.329, 0 failed
progress: 30.0 s, 263.3 tps, lat 30.347 ms stddev 25.051, 0 failed
progress: 36.0 s, 448.1 tps, lat 17.854 ms stddev 7.454, 0 failed
progress: 42.0 s, 458.7 tps, lat 17.429 ms stddev 7.015, 0 failed
progress: 48.0 s, 453.5 tps, lat 17.625 ms stddev 7.479, 0 failed
progress: 54.0 s, 462.7 tps, lat 17.267 ms stddev 10.233, 0 failed
progress: 60.0 s, 443.0 tps, lat 18.040 ms stddev 15.999, 0 failed
transaction type: &lt;builtin: TPC-B (sort of)&gt;
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 25933
number of failed transactions: 0 (0.000%)
latency average = 18.495 ms
latency stddev = 11.514 ms
initial connection time = 22.977 ms
tps = 431.947031 (without initial connection time)
</pre>

> Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
Протестировать заново
Что изменилось и почему?

<pre>pgbench (15.2 (Ubuntu 15.2-1.pgdg22.04+1))
starting vacuum...end.
progress: 6.0 s, 434.5 tps, lat 18.264 ms stddev 8.442, 0 failed
progress: 12.0 s, 454.0 tps, lat 17.607 ms stddev 10.871, 0 failed
progress: 18.0 s, 452.5 tps, lat 17.654 ms stddev 13.695, 0 failed
progress: 24.0 s, 459.0 tps, lat 17.423 ms stddev 12.245, 0 failed
progress: 30.0 s, 399.1 tps, lat 19.978 ms stddev 16.924, 0 failed
progress: 36.0 s, 457.6 tps, lat 17.515 ms stddev 13.543, 0 failed
progress: 42.0 s, 458.5 tps, lat 17.429 ms stddev 13.243, 0 failed
progress: 48.0 s, 457.3 tps, lat 17.470 ms stddev 12.645, 0 failed
progress: 54.0 s, 448.7 tps, lat 17.825 ms stddev 12.113, 0 failed
progress: 60.0 s, 455.3 tps, lat 17.537 ms stddev 11.648, 0 failed
transaction type: &lt;builtin: TPC-B (sort of)&gt;
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 26867
number of failed transactions: 0 (0.000%)
latency average = 17.847 ms
latency stddev = 12.686 ms
initial connection time = 33.386 ms
tps = 447.723652 (without initial connection time)</pre>
__увеличилось количество транзакций в секунду, изменились задержки. в предоставленных параметрах  большее влияние оказали настройки связанные с увеличением памяти буферноо кэша и work_mem, так же молги повлиять и настройки связанные со статистикой(увеличения количества объема статистики для столбцов ) что влияет на выбор оптимального плана оптимизатором запросов__

>Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк

<pre>test=# create table random_data(data text);
CREATE TABLE
test=# insert into random_data(data) select substring(&apos;kdhfjg234jfdgjdkjsfwieufkjjbk&apos;,(round(random() * 25) :: integer),8) from generate_series(1,1000000);
INSERT 0 1000000</pre>

>Посмотреть размер файла с таблицей

<pre>test=# SELECT pg_size_pretty(pg_total_relation_size(&apos;random_data&apos;));
-[ RECORD 1 ]--+------
pg_size_pretty | 41 MB</pre>

>5 раз обновить все строчки и добавить к каждой строчке любой символ


<pre>do $$
begin
for i in 1..5 loop
execute &apos;update random_data set data = &apos;||&apos;&apos;&apos;a&apos;&apos;&apos;||&apos;||&apos;||&apos;data&apos;;  
end loop;
end
$$
;
</pre>

>Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум

SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = 'random_data';

<pre>test=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float &quot;ratio%&quot;, last_autovacuum FROM pg_stat_user_TABLEs WHERE relname = &apos;random_data&apos;;
-[ RECORD 1 ]---+------------------------------
relname         | random_data
n_live_tup      | 1000000
n_dead_tup      | 0
ratio%          | 0
last_autovacuum | 2023-04-28 04:45:49.225777+05</pre>
__автовакуум успел пройти__
>5 раз обновить все строчки и добавить к каждой строчке любой символ
Посмотреть размер файла с таблицей

<pre>SELECT pg_size_pretty(pg_total_relation_size(&apos;random_data&apos;));
-[ RECORD 1 ]--+-------
pg_size_pretty | 275 MB</pre>

>Отключить Автовакуум на конкретной таблице
10 раз обновить все строчки и добавить к каждой строчке любой символ
Посмотреть размер файла с таблицей
Объясните полученный результат

<pre>test=# ALTER TABLE random_data SET (autovacuum_enabled = off);
ALTER TABLE
test=# do $$                                                        
begin
for i in 1..10 loop
execute &apos;update random_data set data = &apos;||&apos;&apos;&apos;a&apos;&apos;&apos;||&apos;||&apos;||&apos;data&apos;;  
end loop;
end
$$
;
DO
test=# SELECT pg_size_pretty(pg_total_relation_size(&apos;random_data&apos;));
-[ RECORD 1 ]--+-------
pg_size_pretty | 585 MB
</pre>

__из-за использования MVCC при каждом update-е появляются dead tuples, автовакуум ищет их , удаляет и позволяет в результате освободивщимся ресурсам быть переиспользованными, но он не освобождает пространство на жеском диске. Для этого необходим vacuum full который пересоздает таблицы, уменьшая их физический размер__   







