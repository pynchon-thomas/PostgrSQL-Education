>поставьте на нее PostgreSQL 15 через sudo apt

* __sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'__
* __wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -__
* __sudo apt-get update__
* __apt install postgresql-15__


>проверьте что кластер запущен через sudo -u postgres pg_lsclusters

<pre>root@user-VirtualBox:~# pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
<font color="#26A269">15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log</font></pre>