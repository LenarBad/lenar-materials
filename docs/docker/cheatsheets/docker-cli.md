# Docker CLI

Короткие команды и заготовки для копирования. Подробные объяснения по темам — в [разделе Topics](../topics/README.md); сценарии отладки — в [рецептах](../recipes/README.md).

## См. тематические источники
- [Обзор тем](../topics/README.md)

## Образы

<div class="lenar-tip-grid" markdown="block">

!!! note "Сборка из текущего каталога"

    ```bash
    docker build -t myapp:1.0 .
    ```

!!! note "Сборка из подкаталога (контекст — корень репозитория)"

    ```bash
    docker build -f docker/Dockerfile -t myapp:1.0 .
    ```

!!! note "Полный лог сборки (удобно ловить ошибку в RUN)"

    ```bash
    docker build --progress=plain --no-cache -t myapp:debug .
    ```

!!! note "Тег из digest (воспроизводимая база)"

    Digest (`sha256:...`) фиксирует конкретный образ; в отличие от тега, не «переедет» на новую версию.

    ```bash
    docker pull docker.io/library/alpine@sha256:abcdef...
    docker tag docker.io/library/alpine@sha256:abcdef... mybase:pinned
    ```

!!! note "Сборка с файлом compose (цель сервиса)"

    ```bash
    docker compose build api
    ```

!!! note "Список локальных образов"

    ```bash
    docker images
    docker image ls --digests
    ```

!!! note "Push / pull (реестр явно)"

    ```bash
    docker tag myapp:1.0 registry.example.com/team/myapp:1.0
    docker push registry.example.com/team/myapp:1.0
    docker pull registry.example.com/team/myapp:1.0
    ```

!!! note "Удалить неиспользуемые данные (осторожно)"

    ```bash
    docker system df
    docker system prune
    docker volume prune
    ```

</div>

## Контейнеры: запуск и жизненный цикл

<div class="lenar-tip-grid" markdown="block">

!!! note "Интерактив в отладку"

    ```bash
    docker run --rm -it myapp:1.0 sh
    ```

!!! note "Фон, имя, порты, переменные"

    ```bash
    docker run -d --name api -p 8080:8080 -e NODE_ENV=production myapp:1.0
    ```

!!! note "Логи и след"

    ```bash
    docker logs -f api
    docker ps -a
    ```

!!! note "Команда внутри работающего контейнера"

    ```bash
    docker exec -it api sh
    ```

!!! note "Остановить и удалить"

    ```bash
    docker stop api
    docker rm api
    ```

!!! note "Код выхода последнего запуска"

    ```bash
    docker inspect api --format '{{.State.ExitCode}}'
    ```

!!! note "Ограничения CPU/RAM (локальная проверка)"

    ```bash
    docker run --rm --cpus="1.5" --memory=512m myapp:1.0
    ```

!!! note "Проброс переменных из файла"

    ```bash
    docker run --rm --env-file ./.env.docker -p 8080:8080 myapp:1.0
    ```

!!! note "Инспекция и копирование файлов"

    ```bash
    docker inspect api | less
    docker cp api:/app/logs/app.log ./app.log
    docker cp ./local.conf api:/etc/app/local.conf
    ```

!!! note "Нагрузка по контейнерам"

    ```bash
    docker stats
    ```

</div>

## Тома и монтирование

<div class="lenar-tip-grid" markdown="block">

!!! note "Именованный том"

    ```bash
    docker volume create appdata
    docker run --rm -v appdata:/data busybox ls /data
    ```

!!! note "Bind-mount каталога проекта (разработка)"

    ```bash
    docker run --rm -v "$PWD:/app" -w /app docker.io/library/node:20 npm test
    ```

!!! note "Том + подмонтировать только один конфиг"

    ```bash
    docker run --rm -v appdata:/data -v "$PWD/app.yaml:/etc/app.yaml:ro" myapp:1.0
    ```

</div>

## Сеть (точечно)

<div class="lenar-tip-grid" markdown="block">

!!! note "Список сетей и контейнеров в сети"

    ```bash
    docker network ls
    docker network inspect bridge
    ```

!!! note "Два запущенных контейнера в одной пользовательской сети"

    ```bash
    docker network create demo
    docker run -d --name web --network demo -p 8080:80 docker.io/library/nginx:alpine
    docker run --rm --network demo curlimages/curl -sS http://web
    ```

</div>

## Compose

<div class="lenar-tip-grid" markdown="block">

!!! note "Поднять стек в фоне"

    ```bash
    docker compose up -d
    ```

!!! note "Логи сервиса"

    ```bash
    docker compose logs -f api
    ```

!!! note "Остановить и убрать контейнеры сети"

    ```bash
    docker compose down
    ```

!!! note "С томами (данные БД тоже удалятся)"

    ```bash
    docker compose down -v
    ```

!!! note "Статус сервисов и перезапуск одного"

    ```bash
    docker compose ps -a
    docker compose restart api
    ```

!!! note "Одноразовая задача с теми же env/сетью, что у сервиса"

    ```bash
    docker compose run --rm api npm run lint
    ```

!!! note "Логи с метками времени"

    ```bash
    docker compose logs -f --timestamps api
    ```

</div>
