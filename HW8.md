> Установить на него PostgreSQL 15 с дефолтными настройками

* __sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'__
* __wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -__
* __sudo apt-get update__
* __apt install postgresql-15__

