> Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.

<pre>postgres=# alter system set log_lock_waits=on;
ALTER SYSTEM
</pre>
<pre>postgres=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)
</pre>
* __создадим таблицу:__
CREATE TABLE accounts(
  acc_no integer PRIMARY KEY,
  amount numeric
);
INSERT INTO accounts VALUES (1,1000.00), (2,2000.00), (3,3000.00);

* __в первой сессии начнем транзакцию и выполним UPDATE__
<pre>ocks=# begin;
BEGIN
locks=*# UPDATE accounts SET amount = amount + 100 WHERE acc_no = 1;</pre>
* __во второй сесси запутим создание индексов__
<pre>locks=# CREATE INDEX ON accounts(acc_no);
</pre>
* __идем в журнал логов и видим информацию о блокировках:__
<pre>2023-05-05 01:36:46.062 +05 [23991] postgres@postgres STATEMENT:  UPDATE accounts SET amount = amount + 100 WHERE acc_no = 1;
2023-05-05 01:39:34.542 +05 [24345] postgres@postgres STATEMENT:  CREATE INDEX ON accounts(acc_no);
2023-05-05 01:39:47.159 +05 [24348] postgres@locks LOG:  process 24348 still waiting for ShareLock on relation 16490 of database 16489 after 1007.752 ms
2023-05-05 01:39:47.159 +05 [24348] postgres@locks DETAIL:  Process holding the lock: 24332. Wait queue: 24348.
2023-05-05 01:39:47.159 +05 [24348] postgres@locks STATEMENT:  CREATE INDEX ON accounts(acc_no);
</pre>

> Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах. Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны. Пришлите список блокировок и объясните, что значит каждая.

__поочередно запускаем команду на обновление одной и той же строки в трех сессиях__

[__вывод select * from pg_locks__](https://github.com/pynchon-thomas/PostgrSQL-Education/blob/main/pg_locks.txt)

__кроме того что мы видим блокироваки на уровне таблицы и собстьевнного номера транзакции(RowExclusiveLock и ExclusiveLock), есть блокировка на уровне версии строк(tuple), которая есть и в режиме ожидания получения блокировки, так как мы осуществляем UPDATE строки. СУБД атоматически использует блокировку на уровне строк в зависимости от контеста__

> Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?

* __в первой сессии:__
<pre>locks=# begin;
BEGIN
locks=*# UPDATE accounts SET amount = amount + 100 WHERE acc_no = 1;
UPDATE 1
</pre>
* __во второй сессии:__
<pre>locks=# UPDATE accounts SET amount = amount + 100 WHERE acc_no = 1
;
</pre>
* __в первой сессии:__
<pre>locks=*# UPDATE accounts SET amount = amount + 100 WHERE acc_no = 2;
UPDATE 1
</pre>
* __в третьей сессии:__
<pre>ocks=# begin;
BEGIN
locks=*# UPDATE accounts SET amount = amount + 100 WHERE acc_no = 2
;
UPDATE 1
locks=*# UPDATE accounts SET amount = amount + 100 WHERE acc_no = 1
;
ERROR:  deadlock detected
DETAIL:  Process 24590 waits for ExclusiveLock on tuple (0,11) of relation 16490 of database 16489; blocked by process 24348.
Process 24348 waits for ShareLock on transaction 53602; blocked by process 24332.
Process 24332 waits for ShareLock on transaction 53604; blocked by process 24590.
HINT:  See server log for query details.</pre>

__СУБД в атоматическом режиме определяет ситуацию с deadlock-ом. Информации которая пишется в лог при обноружении данной ситуации в принципе достаточно понять какие процессы лочили друг друга и очереди котрые они выполняли при этом то же логируются в журнал__

> Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?

__если про deadlock, то нет__



