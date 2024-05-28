# Установка PostgreSQL.

Занятие от 08.04.2024

## Выполнение домашнего задания:

 - создать ВМ с Ubuntu 20.04/22.04 или развернуть докер любым удобным способом
 - поставить на нем Docker Engine

Команда линукс:
```
curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER && newgrp docker
```

 - сделать каталог /var/lib/postgres

Команда линукс:
```
sudo mkdir /var/lib/postgres
```

 - развернуть контейнер с PostgreSQL 15 смонтировав в него /var/lib/postgresql
Команда линукс:
```
sudo docker network create pg-net
```

 - развернуть контейнер с клиентом postgres

Команда линукс:
```
sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
```

 - подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк

Команда линукс:
```
sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres
```

Текст запросов:
```
create table t1 (number int);
insert into t1 values(1);
insert into t1 values(2);
```

Результат запроса ***select * from t1***:
```
 number
--------
      1
      2
```
 - подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера
 - удалить контейнер с сервером

Команда линукс:
```
sudo docker rm pg-server
```

 - создать его заново

Команда линукс:
```
sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
```

 - подключится снова из контейнера с клиентом к контейнеру с сервером

Команда линукс:
```
sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres
```

 - проверить, что данные остались на месте 

Результат запроса ***select * from t1***:
```
 number
--------
      1
      2
```

 - оставляйте в ЛК ДЗ комментарии что и как вы делали и как боролись с проблемами

 Комментарий: всё делал по лекции, проблем не возникло. 