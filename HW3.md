## Homework 3

>поставить на нем Docker Engine

__curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER__

>сделать каталог /var/lib/postgres

__sudo mkdir /var/lib/postgres__

>развернуть контейнер с PostgreSQL 15 смонтировав в него /var/lib/postgresql

* __создаем user-defined bridge network:__ 
 sudo docker network create pg-net
* __создание и запуск контейнера с подлкючением к созданной сети и bind mount директории с файловой системы хоста в контейнер:__

sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/ postgres:15

>развернуть контейнер с клиентом postgres.
подключится из контейнера с клиентом к контейнеру с сервером и сделать
таблицу с парой строк

* __sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-server -U postgres__
* __create database docker; \c docker, create table test(id serial, data text); insert into test(data) values ('some'),('data'),('another'),('data');__

>подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера

__psql -p 5432 -U postgres -h 10.0.2.15 -d postgres -W__

>удалить контейнер с сервером

* sudo docker stop pg-server
* sudo docker rm pg-server

> создать его заново
• подключится снова из контейнера с клиентом к контейнеру с сервером
• проверить, что данные остались на месте

Данные отсутсвовали. При анализе конифигурации контейнера через 
docker inspect pg-server, выяснилось следующее:

>Mounts": [
            {
                "Type": "bind",
                "Source": "/var/lib/postgres",
                "Destination": "/var/lib/postgresql",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            },
            {
                "Type": "volume",
                "Name": "bfd8763416630a3a97dd27d7cd4f8c17a99fa34d2d4d78237df75c28661f3ae6",
                "Source": "/var/lib/docker/volumes/bfd8763416630a3a97dd27d7cd4f8c17a99fa34d2d4d78237df75c28661f3ae6/_data",
                "Destination": "/var/lib/postgresql/data",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }

При старте контейнера автоматически создавались два объекта монтирования каталогов: объект типа bind mount и объект volume.В директории , которая монтировалась к данному volume-у внутри контейнера кластер Postgre и хранил БД и метаинформацию, но на уровне ОС хоста, на котором работает docker engine это совершенно две разные директории. Проблема была решена заменой параметра для bind mount, в команде запуска контейнера:
 -v /var/lib/postgres:/var/lib/postgresql/data. После этого в конфигурации контейнера фигурирует только один объект монтирования и данные не пропали после удаления и последующей установки контейнера