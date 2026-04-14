# Docker Compose

## Цель
Управлять несколькими связанными сервисами как единым стеком: описывать их в одном `compose.yaml`, поднимать окружение одинаково на локальной машине и в CI, быстро диагностировать проблемы запуска и связности.

## Базовая модель Compose
- `services` описывает процессы (приложение, БД, кэш, воркер).
- `networks` задает, как сервисы видят друг друга по именам.
- `volumes` хранит состояние между перезапусками (например, данные PostgreSQL).
- Конфигурация становится версионируемой и воспроизводимой: один файл для всей команды.

Практически Compose закрывает две задачи:
- **оркестрация локального окружения** (поднять стек одной командой);
- **единообразный runtime-контракт** (какие переменные, порты, тома, healthcheck нужны сервису).

## Из чего состоит `compose.yaml`

Ниже иерархия, которую удобно использовать как «скелет» файла:

- `compose.yaml`
  - **Верхний уровень**
    - `services` — обязательный раздел с контейнерами.
    - `volumes` — именованные тома для состояния.
    - `networks` — явные сети (если дефолтной недостаточно).
    - `name` (опционально) — стабильное имя проекта.
  - **`services`**
    - `<service-name>` (например: `api`, `db`, `redis`)
      - **Источник образа:** `image` или `build`.
      - **Процесс:** `command` и/или `entrypoint`.
      - **Конфигурация:** `environment`, `env_file`.
      - **Сеть и доступ:** `ports`, `networks`.
      - **Хранилище:** `volumes` (bind mount и/или named volume).
      - **Надежность:** `restart`, `healthcheck`, `depends_on`.
  - **Связи между сервисами**
    - Сервисы обращаются друг к другу по имени (`db`, `redis`, `api`).
    - Порядок старта задают через `depends_on`.
    - Готовность зависимости фиксируют `healthcheck` + `condition: service_healthy`.
  - **Варианты запуска**
    - Базовая конфигурация: `compose.yaml`.
    - Локальные dev-надстройки: `compose.override.yaml`.
    - Опциональные части стека: `profiles`.

## Как строить compose-файл с нуля

### Шаг 1. Описать сервисы как процессы
Сначала перечислите «кто у вас есть»: `api`, `db`, `redis`, `worker`, `nginx`.

Минимальный старт:

```yaml
services:
  api:
    build: .
```

### Шаг 2. Добавить зависимости и адреса
Добавьте БД/кэш и сразу пропишите адреса не через `localhost`, а через имена сервисов.

```yaml
services:
  api:
    build: .
    environment:
      DATABASE_URL: postgres://app:app@db:5432/app
  db:
    image: docker.io/library/postgres:16-alpine
```

### Шаг 3. Настроить данные (volumes)
Если сервис хранит состояние, сразу выносите его в именованный том.

```yaml
services:
  db:
    image: docker.io/library/postgres:16-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

### Шаг 4. Настроить готовность и порядок старта
Добавьте `healthcheck` на критичные зависимости и `depends_on` для сервисов, которые от них зависят.

### Шаг 5. Открыть только нужные порты
Публикуйте наружу только то, к чему нужен доступ с хоста (`api`, админка).  
Внутренние сервисы (например, `db`, `redis`) не обязаны иметь `ports`, если к ним ходят только другие контейнеры.

### Шаг 6. Проверить итоговую конфигурацию
Перед первым запуском и при любом рефакторинге:

```bash
docker compose config
docker compose up -d
docker compose ps
```

## Шаблон для большинства проектов

```yaml
services:
  app:
    build: .
    environment:
      APP_ENV: development
      DATABASE_URL: postgres://app:app@db:5432/app
    depends_on:
      db:
        condition: service_healthy
    ports:
      - "8080:8080"

  db:
    image: docker.io/library/postgres:16-alpine
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app
      POSTGRES_DB: app
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app -d app"]
      interval: 5s
      timeout: 3s
      retries: 20
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

## Базовый `compose.yaml`

```yaml
services:
  db:
    image: docker.io/library/postgres:16-alpine
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app
      POSTGRES_DB: app
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  api:
    build: .
    ports:
      - "8080:8080"
    environment:
      DATABASE_URL: postgres://app:app@db:5432/app
    depends_on:
      db:
        condition: service_healthy

volumes:
  pgdata:
```

Что важно в примере:
- сервис `api` обращается к БД по имени сервиса `db`, а не по `localhost`;
- `depends_on` с `condition: service_healthy` учитывает healthcheck БД, а не только факт старта процесса;
- именованный том `pgdata` переживает `docker compose down` (но не `down -v`).

## Healthcheck и готовность зависимостей
`depends_on` без условия `service_healthy` не гарантирует готовность приложения к запросам.  
Для БД и брокеров обычно нужен явный `healthcheck`.

```yaml
services:
  db:
    image: docker.io/library/postgres:16-alpine
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app
      POSTGRES_DB: app
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app -d app"]
      interval: 5s
      timeout: 3s
      retries: 20

  api:
    build: .
    depends_on:
      db:
        condition: service_healthy
```

Интерпретация:
- пока `db` не в `healthy`, `api` не будет стартовать;
- если `api` уже запущен и `db` позже станет `unhealthy`, Compose не «починит» логику приложения автоматически;
- обработка повторных попыток подключения все равно нужна внутри приложения.

## Сети и DNS имен сервисов
Compose создает проектную сеть по умолчанию, где каждый сервис доступен по имени.

Пример:
- `api` подключается к `db:5432`;
- `worker` подключается к `redis:6379`.

Проверка связности:

```bash
docker compose exec api getent hosts db
docker compose exec api sh -lc "nc -zv db 5432"
```

Если DNS не резолвится:
- проверьте, что сервисы в одной сети;
- проверьте имя сервиса в `compose.yaml` (опечатка встречается часто);
- убедитесь, что контейнер назначения действительно запущен.

## Переменные окружения и `.env`
Compose использует переменные из двух источников:
- `.env` рядом с `compose.yaml` (подстановка в сам файл);
- блок `environment` внутри конкретного сервиса (переменные процесса контейнера).

```yaml
services:
  api:
    image: myorg/api:${APP_TAG:-dev}
    environment:
      APP_ENV: ${APP_ENV:-development}
      LOG_LEVEL: ${LOG_LEVEL:-info}
```

Практические правила:
- используйте значения по умолчанию `${VAR:-default}` для локальной разработки;
- не храните секреты в репозитории; для локальной работы используйте локальный `.env`, который исключен из VCS;
- проверяйте финальную конфигурацию через `docker compose config`.

## Тома и данные
Данные БД/очередей почти всегда должны жить в именованных томах:

```yaml
services:
  db:
    image: docker.io/library/postgres:16-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

Поведение:
- `docker compose down` удаляет контейнеры и сеть, но оставляет именованные тома;
- `docker compose down -v` удаляет и тома (данные будут потеряны);
- bind-mount (`./src:/app/src`) удобен для dev-кода, но хуже для постоянных данных.

## Override-файлы и профили
Обычно есть базовый `compose.yaml` и dev-расширение `compose.override.yaml`.

Типичный подход:
- в базовом файле описать общие сервисы и безопасные дефолты;
- в override добавить dev-тома, debug-порты, hot reload;
- для опциональных сервисов использовать `profiles`.

```yaml
services:
  adminer:
    image: docker.io/library/adminer:latest
    profiles: ["debug"]
    ports:
      - "8081:8080"
```

Запуск профиля:

```bash
docker compose --profile debug up -d
```

## Compose Watch (`develop.watch`)
Для локальной разработки Compose может отслеживать изменения файлов и автоматически:
- синхронизировать код в работающий контейнер;
- перезапускать сервис;
- пересобирать образ, если меняются файлы зависимостей.

Минимальный пример:

```yaml
services:
  api:
    build: .
    develop:
      watch:
        - action: sync+restart
          path: .
          target: /app
        - action: rebuild
          path: requirements.txt
```

Запуск:

```bash
docker compose up --watch
```

Когда это полезно:
- быстрее обычного цикла `build -> up` после каждого изменения;
- удобно для backend/API в dev-среде;
- не заменяет production-сборку, а ускоряет локальную итерацию.

## Несколько compose-файлов (`include`)
Когда стек растет, конфигурацию удобно разделять на доменные файлы (например, приложение и инфраструктура отдельно).

Пример:

```yaml
include:
  - path: ./infra.yaml

services:
  api:
    build: .
```

Типичный подход:
- в `compose.yaml` держать app-сервисы;
- в `infra.yaml` вынести БД, кэш, очередь и их тома/healthcheck;
- проверять итоговый merge командой `docker compose config`.

## Основные операции

```bash
docker compose up -d
docker compose up --watch
docker compose logs -f api
docker compose ps -a
docker compose config
docker compose run --rm api python manage.py migrate
docker compose exec api sh
docker compose down
```

Когда использовать:
- `up -d` — поднять/пересоздать сервисы в фоне;
- `up --watch` — запуск с автосинхронизацией/перезапуском в dev;
- `logs -f` — смотреть живые логи конкретного сервиса;
- `config` — увидеть итоговую конфигурацию после подстановок и merge;
- `run --rm` — одноразовая задача в окружении проекта;
- `exec` — зайти в уже работающий контейнер;
- `down` — штатно остановить стек.

## Примеры

### Локальный стек API + PostgreSQL + Redis

```yaml
services:
  api:
    build: .
    ports:
      - "8080:8080"
    environment:
      DATABASE_URL: postgres://app:app@db:5432/app
      REDIS_URL: redis://redis:6379/0
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started

  db:
    image: docker.io/library/postgres:16-alpine
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app
      POSTGRES_DB: app
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app -d app"]
      interval: 5s
      timeout: 3s
      retries: 20
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: docker.io/library/redis:7-alpine

volumes:
  pgdata:
```

### Отладка «почему сервис не поднимается»

```bash
docker compose ps -a
docker compose logs --tail 100 api
docker compose logs --tail 100 db
docker compose config
docker compose exec api env | sort
```

Если сервис циклически перезапускается:
- проверьте код выхода контейнера через `docker compose ps -a`;
- сравните переменные окружения с ожидаемыми;
- убедитесь, что зависимость действительно доступна по сетевому имени и порту.

## Частые ошибки
- Ожидать, что обычный `depends_on` гарантирует готовность БД к запросам.
- Писать адреса зависимостей через `localhost` вместо имени сервиса (`db`, `redis`).
- Случайно удалять данные командой `docker compose down -v`.
- Хранить секреты прямо в `compose.yaml` и коммитить их в репозиторий.
- Не проверять итоговый merge и подстановки через `docker compose config`.
