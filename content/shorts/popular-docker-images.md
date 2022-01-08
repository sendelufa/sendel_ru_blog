---
title: "Частые контейнеры Docker"
date: 2022-01-06T20:15:46+03:00
draft: true
description: "Команды для запуска контейнеров из командной строки"
---

## MySQL

### Docker

```docker
docker run \
--detach \
--name=[container-name] \
--env="MYSQL_ROOT_PASSWORD=[root-password]" \
--env="MYSQL_DATABASE=[schema_name]" \
--publish [port]:3306 \
--volume=[path-to-Mysql-data]:/var/lib/mysql \
mysql
```

**Переменные:**

- `[container-name]` - имя контейнера, например library_mysql, todo_mysql.
- `[root-password]` - пароль для пользователя root.
- `[schema_name]` - имя базы данных, создаваемой при старте контейнера
- `[port]` - порт через который можно будет подключится к базе данных в контейнере.
- `[path-to-Mysql-data]` - путь до директории, в которой будут хранятся файлы базы данных.
Если параметр `volume` не указывать, все данные будут потерены если контейнер удалить.

**Пример команды с установленными параметрами:**

- Контейнер с именем sendel_mysql доступен на порту 3336,
пароль root "12345678", база данных сохраняется в директории /home/user/mysql-data:

```shell
docker run \
--detach \
--name=sendel_mysql \
--env="MYSQL_ROOT_PASSWORD=12345678" \
--env="MYSQL_DATABASE=db" \
--publish 3336:3306 \
--volume=/home/user/mysql-data:/var/lib/mysql \
mysql
```

### Docker Compose

Создайте файл 'docker-compose.yml' с содержимым, поменяйте значения параметров
при необходимости.

```docker
version: '3.3'
services:
  db:
    image: mysql:latest
    restart: always
    environment:
      MYSQL_DATABASE: 'db' # имя бд в контейнере
      MYSQL_USER: 'user' # пользователь, права имеет только на MYSQL_DATABASE
      MYSQL_PASSWORD: 'password' # пароль пользователя
      MYSQL_ROOT_PASSWORD: 'password' # пароль пользователя root 
    ports:
      - '3336:3306' # 3336 будет виден снаружи контейнера
    # данные всех бд будут сохранены в /home/user/mysql-data
    volumes:
      - /home/user/mysql-data:/var/lib/mysql
```

Запуск. Перейдите в директорию с файлом `docker-compose.yml` и выполните:

```docker
docker-compose up -d
```

## PostgreSQL

### Docker

```docker
docker run \
--detach \
--name=[container-name] \
--env="POSTGRES_PASSWORD=[postgres-password]" \
--publish [port]:5432 \
--volume=[path-to-postgres-data]:/var/lib/postgresql/data \
postgres
```

**Переменные:**

- `[container-name]` - имя контейнера, например library_postgres, todo_postgres.
- `[postgres-password]` - пароль для пользователя postgres.
- `[port]` - порт через который можно будет подключится к базе данных в контейнере.
- `[path-to-postgres-data]` - путь до директории, в которой будут хранятся файлы базы данных.
Если параметр `volume` не указывать, все данные будут потерены если контейнер удалить.

По-умолчанию, имя пользователя `postgres`.

**Пример команды с установленными параметрами:**

- Контейнер с именем sendel_postgres доступен на порту 5454,
пароль "12345678" у пользователя postgres , база данных сохраняется в директории /home/user/postgres-data:

```docker
docker run \
--detach \
--name=sendel_postgres \
--env="POSTGRES_PASSWORD=12345678" \
--publish 5454:5432 \
--volume=/home/user/postgres-data:/var/lib/postgresql/data \
postgres
```

### Docker Compose

Создайте файл 'docker-compose.yml' с содержимым, поменяйте значения параметров
при необходимости.

```docker
version: '3.3'
services:
  db:
    image: postgres:latest
    restart: always
    environment:
      POSTGRES_PASSWORD: 'password'
    ports:
      - '5432:5432'
    # данные всех бд контейнера будут сохранены в /home/user/postgres-data
    volumes:
      - /home/user/postgres-data:/var/lib/postgresql/data
```

Запуск. Перейдите в директорию с файлом `docker-compose.yml` и выполните:

```docker
docker-compose up -d
```

## MongoDB




Чтобы контейнер не стартовал при старте системы или после его падения, поставьте параметр:

```docker
restart: 



