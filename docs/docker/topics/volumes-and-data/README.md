# Тома и данные

## Цель
Понимать, где данные контейнера временные, а где постоянные, и как выбирать между volume и bind-mount.

## Как хранится состояние
- Записываемый слой контейнера удаляется вместе с контейнером.
- Для персистентных данных используйте **volume**.
- Для локальной разработки и подмены файлов удобен **bind-mount**.

## Volume: постоянные данные

```bash
docker volume create pgdata
docker run -d --name pg -e POSTGRES_PASSWORD=secret -v pgdata:/var/lib/postgresql/data -p 5432:5432 docker.io/library/postgres:16
```

- `docker volume create pgdata` создает именованный том, который живет отдельно от контейнера.
- В `docker run`:
  - `-v pgdata:/var/lib/postgresql/data` подключает том в каталог данных PostgreSQL.
  - `-e POSTGRES_PASSWORD=secret` задает пароль суперпользователя при первом старте.
  - `-p 5432:5432` публикует порт БД на хост.
- Такой том сохраняет данные даже если контейнер удалить и создать заново с тем же `-v pgdata:...`.

Проверка результата:

```bash
docker volume ls
docker volume inspect pgdata
docker rm -f pg
docker run -d --name pg2 -e POSTGRES_PASSWORD=secret -v pgdata:/var/lib/postgresql/data docker.io/library/postgres:16
```

## Bind-mount: файлы с хоста

```bash
docker run --rm -v "$PWD:/app" -w /app docker.io/library/node:20 npm test
```

- `-v "$PWD:/app"` монтирует текущую папку хоста в `/app` внутри контейнера.
- `-w /app` делает `/app` рабочей директорией, поэтому `npm test` запускается по файлам хоста.
- Это удобно для разработки: изменили код на хосте, и контейнер сразу видит изменения без пересборки образа.

Проверка результата:

```bash
docker run --rm -v "$PWD:/app" -w /app docker.io/library/node:20 sh -lc 'pwd && ls -la'
```

## Безопасность записи
Если сервис не должен писать в корень ФС контейнера:

```bash
docker run --rm --read-only -v app-tmp:/tmp -v app-data:/data myapp:1.0
```

- `--read-only` делает корневую файловую систему контейнера только для чтения.
- `-v app-tmp:/tmp` и `-v app-data:/data` дают явные места, куда запись разрешена.
- Если приложение пытается писать в каталоги вроде `/var`, `/root` или другой путь вне смонтированных томов, оно получит ошибку записи.

Проверка результата:

```bash
docker run --rm --read-only -v app-tmp:/tmp docker.io/library/alpine:3.20 sh -lc 'echo ok >/tmp/test && echo fail >/root/test'
```

## Частые ошибки
- Монтировать пустой каталог хоста поверх каталога приложения в образе и «терять» файлы.
- Игнорировать права UID/GID при bind-mount.
- Выполнять `docker compose down -v` без понимания, что это удалит данные томов.
