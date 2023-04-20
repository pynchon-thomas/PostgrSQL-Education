>поставьте на нее PostgreSQL 15 через sudo apt

* __sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'__
* __wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -__
* __sudo apt-get update__
* __apt install postgresql-15__


>проверьте что кластер запущен через sudo -u postgres pg_lsclusters

<pre>root@user-VirtualBox:~# pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
<font color="#26A269">15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log</font></pre>



>перезагрузите инстанс и убедитесь, что диск остается примонтированным (если не так смотрим в сторону fstab)

* __был создан LV -- 
lsblk__
<pre>sda                   8:0    0    25G  0 disk 
├─sda1                8:1    0     1M  0 part 
├─sda2                8:2    0   513M  0 part /boot/efi
└─sda3                8:3    0  24,5G  0 part 
  ├─vgubuntu-root   253:0    0  22,6G  0 lvm  /var/snap/firefox/common/host-hunspell
  │                                           /
  └─vgubuntu-swap_1 253:1    0   1,9G  0 lvm  [SWAP]
sdb                   8:16   0    10G  0 disk 
└─postgrevg-lvol0   253:2    0     5G  0 lvm  /mnt/data
</pre>

>сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/
перенесите содержимое /var/lib/postgres/15 в /mnt/data - mv /var/lib/postgresql/15/mnt/data
попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
напишите получилось или нет и почему

__кластер не запустился, так как нет данных в PGDATA__

>задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/15/main который надо поменять и поменяйте его
напишите что и почему поменяли
попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
напишите получилось или нет и почему

__файл отвечающий за конфигурацию кластера: postgresql.conf. Параметр который необходимо поменять в данной файле : data_directory. После того как поменял данный параметр кластер успешно запустился__

>зайдите через через psql и проверьте содержимое ранее созданной таблицы

<pre>postgres=# select * from test;
 c1 
----
 1
(1 row)
</pre>

>не удаляя существующий инстанс ВМ сделайте новый, поставьте на его PostgreSQL, удалите файлы с данными из /var/lib/postgres, перемонтируйте внешний диск который сделали ранее от первой виртуальной машины ко второй и запустите PostgreSQL на второй машине так чтобы он работал с данными на внешнем диске, расскажите как вы это сделали и что в итоге получилось.

__на новой вирутальной машине диск автоматом смонтировался в директорию media/. После остановки кластера и удаления файлов из директории /var/lib/postgresql диск был отмантирован и  вновь примонтирован в директорию /var/lib/postgresql/ после чего кластер был успешно запущен и проверены данные  в таблице__




