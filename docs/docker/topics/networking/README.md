# Сеть контейнеров

## Цель
Разобраться, как контейнеры общаются между собой и с хостом: публикация портов, пользовательские сети, DNS-имена сервисов.

## Публикация портов
`-p host:container` публикует порт контейнера на хост.

```bash
docker run -d --name web -p 8080:80 docker.io/library/nginx:alpine
```

Проверка с хоста: запрос на `localhost:8080` должен вернуть страницу Nginx (HTTP 200).

```bash
curl -I http://localhost:8080
```

Важно: `EXPOSE` в Dockerfile не открывает порт наружу сам по себе.

## `localhost` внутри контейнера
Внутри контейнера `localhost` указывает на **этот же контейнер**.  
Для доступа к другому сервису используйте DNS-имя контейнера/сервиса в одной сети.

## Пользовательская сеть

```bash
docker network create demo
docker run -d --name web --network demo docker.io/library/nginx:alpine
docker run --rm --network demo curlimages/curl -sS http://web
```

Что проверяем: контейнер `curl` резолвит имя `web` через встроенный DNS Docker и получает ответ по порту 80.

## Доступ из контейнера к хосту
Иногда приложению внутри контейнера нужен сервис, запущенный на хосте (например, локальная БД или API).

```bash
docker run --rm curlimages/curl -sS http://host.docker.internal:8080
```

Ожидаемый результат: запрос доходит до сервиса на хосте.  
Примечание для Linux: если имя `host.docker.internal` не работает, добавьте маппинг:

```bash
docker run --rm --add-host host.docker.internal:host-gateway curlimages/curl -sS http://host.docker.internal:8080
```

## Диагностика сети

```bash
docker network ls
docker network inspect demo
docker port web
docker run --rm --network demo curlimages/curl -I http://web
```

Что проверяем:
- сеть `demo` существует и содержит ожидаемые контейнеры;
- у `web` опубликован порт `8080 -> 80/tcp`;
- сервис доступен по DNS-имени внутри сети `demo`.

## Очистка ресурсов после примеров

```bash
docker rm -f web || true
docker network rm demo || true
```

## Частые ошибки
- Перепутать стороны в `-p` (`8080:3000` значит снаружи 8080, внутри 3000).
- Пытаться достучаться до БД по `localhost` из соседнего контейнера.
- Считать, что опубликованный порт `-p` обязателен для связи контейнеров между собой в одной сети.
- Использовать `host.docker.internal` без проверки платформенных особенностей на Linux.
- Забрасывать все сервисы в default bridge и ожидать предсказуемой схемы как в Compose.
