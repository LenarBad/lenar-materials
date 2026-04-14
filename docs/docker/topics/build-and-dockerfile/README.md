# Сборка и Dockerfile

## Цель
Научиться собирать Docker-образы быстрее и стабильнее за счет:

- корректной структуры `Dockerfile`;
- использования кэша слоев;
- уменьшения контекста сборки через `.dockerignore`;
- применения multi-stage (многоэтапной) сборки для уменьшения финального образа.

Под «предсказуемой сборкой» здесь понимается ситуация, когда одинаковый код и одинаковые зависимости дают одинаковый результат сборки, без случайных изменений.

## Слои и кэш
Каждая инструкция в `Dockerfile` (`FROM`, `COPY`, `RUN` и т.д.) формирует отдельный слой образа.

**Слой** — это неизменяемый «снимок» файловой системы после выполнения конкретного шага.  
**Кэш слоев** — механизм, при котором Docker повторно использует уже собранный слой, если входные данные шага не изменились.

Почему это важно:

- если Docker берет шаг из кэша, он не выполняет его заново;
- это резко ускоряет повторные сборки;
- структура `Dockerfile` напрямую влияет на то, какие шаги будут кэшироваться.

**Антипаттерн**: сначала `COPY . .`, потом установка зависимостей.  
Это плохо, потому что любое изменение в любом файле проекта «ломает» кэш следующих шагов, включая долгую установку зависимостей.

**Лучше**: сначала копировать lock-файлы вашего стека, потом ставить зависимости, и только после этого копировать исходники.

**Lock-файлы** — файлы, которые фиксируют точные версии зависимостей. Если lock-файл не изменился, зависимости можно безопасно брать из кэша.

Важно: `package-lock.json` — это пример для стека Node.js/npm, а не Docker-специфичный файл.
Для других стеков это могут быть, например, `requirements.txt`/`poetry.lock` (Python), `composer.lock` (PHP), `Cargo.lock` (Rust), `gradle.lockfile` (Java/Gradle).

<div class="lenar-tip-grid" markdown="block">

!!! note "Пример для Node.js/npm"

    ```dockerfile
    COPY package.json package-lock.json ./
    RUN npm ci
    COPY src ./src
    ```

    Что происходит в примере:

    - `COPY package.json package-lock.json ./` переносит только файлы зависимостей;
    - `RUN npm ci` устанавливает зависимости строго по lock-файлу;
    - `COPY src ./src` копирует код приложения.

    Этот пример специально показывает Node.js/npm. Принцип кэширования такой же для любого стека: сначала файлы зависимостей, затем установка, затем код.
    Если вы меняете только код в `src`, то шаг `RUN npm ci` обычно остается в кэше и повторно не выполняется.

!!! note "Python (`pip` + `requirements.txt`)"

    ```dockerfile
    COPY requirements.txt ./
    RUN pip install --no-cache-dir -r requirements.txt
    COPY . .
    ```

    Что это значит: сначала копируем файл зависимостей Python, затем устанавливаем пакеты, и только потом добавляем код проекта.  
    Зачем: если меняется только код, слой с `pip install` обычно остается в кэше.

!!! note "Java (Gradle + `gradle.lockfile`)"

    ```dockerfile
    COPY settings.gradle build.gradle gradle.lockfile ./
    RUN ./gradlew --no-daemon dependencies
    COPY src ./src
    ```

    Что это значит: сначала копируем файлы конфигурации и lock-файл Gradle, затем разрешаем зависимости, потом копируем исходники Java/Kotlin.  
    Зачем: изменения в `src` не должны каждый раз заново пересобирать слой разрешения зависимостей.

!!! note "PHP (`composer` + `composer.lock`)"

    ```dockerfile
    COPY composer.json composer.lock ./
    RUN composer install --no-dev --no-interaction --prefer-dist
    COPY . .
    ```

    Что это значит: `composer` ставит зависимости на основе `composer.lock`, а код копируется позже.  
    Зачем: ускорить повторные сборки и получить воспроизводимые версии пакетов.

!!! note "Rust (`cargo` + `Cargo.lock`)"

    ```dockerfile
    COPY Cargo.toml Cargo.lock ./
    RUN cargo fetch
    COPY src ./src
    RUN cargo build --release
    ```

    Что это значит: сначала скачиваются зависимости Rust (`cargo fetch`), затем добавляется исходный код и запускается сборка.  
    Зачем: сеть и загрузка crate-пакетов реже повторяются при обычных изменениях в коде.

!!! note "Проверка кэша на практике"

    ```bash
    docker build -t myapp:dev .
    docker build -t myapp:dev .
    ```

    Ожидаемый результат:

    - первая сборка выполняет все шаги;
    - вторая сборка показывает, что часть шагов взята из кэша (`CACHED`), и завершается заметно быстрее.

</div>

## Минимальный Dockerfile

<div class="lenar-tip-grid lenar-tip-grid--single" markdown="block">

!!! note "Минимальный пример Dockerfile"

    ```dockerfile
    FROM docker.io/library/python:3.12-slim
    WORKDIR /app
    COPY requirements.txt .
    RUN pip install --no-cache-dir -r requirements.txt
    COPY . .
    EXPOSE 8000
    CMD ["python", "-m", "http.server", "8000"]
    ```

    Разбор по шагам:

    - `FROM docker.io/library/python:3.12-slim` — базовый образ (официальный Python в «облегченном» варианте `slim`, обычно меньше по размеру).
    - `WORKDIR /app` — рабочая директория внутри образа; следующие команды выполняются относительно нее.
    - `COPY requirements.txt .` — копируем только список Python-зависимостей.
    - `RUN pip install --no-cache-dir -r requirements.txt` — ставим зависимости; `--no-cache-dir` не сохраняет кэш `pip` в образе и помогает не раздувать размер.
    - `COPY . .` — копируем остальной проект.
    - `EXPOSE 8000` — документируем, что приложение слушает порт `8000` внутри контейнера.
    - `CMD [...]` — команда по умолчанию при запуске контейнера.

    Почему пример построен именно так:

    - сначала зависимости, потом код — для более эффективного кэширования;
    - команда запуска задана в exec-форме (`["python", ...]`), чтобы корректнее обрабатывались сигналы процесса.

</div>

## `.dockerignore`
`Docker build` отправляет в движок Docker **контекст сборки** — набор файлов из текущей папки.

Если контекст слишком большой, сборка становится медленнее.  
Файл `.dockerignore` исключает лишние файлы из контекста.

<div class="lenar-tip-grid lenar-tip-grid--single" markdown="block">

!!! note "Пример `.dockerignore`"

    ```text
    node_modules
    dist
    .git
    .env
    *.log
    .venv
    ```

    Почему это важно:

    - `node_modules`, `.venv`, `dist` часто большие и пересоздаются локально;
    - `.git` почти никогда не нужен внутри образа;
    - `.env` и логи могут содержать чувствительные данные.

    Результат: меньше передаваемых файлов, быстрее сборка и ниже риск случайно утащить секреты в образ.

</div>

## `CMD`, `ENTRYPOINT`, `ARG`, `ENV`

- `CMD` — команда по умолчанию при запуске контейнера.
- `ENTRYPOINT` — фиксирует основной исполняемый файл (процесс) контейнера.
- Если в конце `docker run` указать команду, она обычно заменяет `CMD`.
- `ARG` — переменная только для этапа сборки.
- `ENV` — переменная окружения, доступная в готовом контейнере во время выполнения (runtime).

Термин **runtime** означает «время выполнения контейнера», когда образ уже собран и контейнер запущен.

<div class="lenar-tip-grid" markdown="block">

!!! note "Пример: переопределение `CMD` при запуске"

    ```bash
    docker run --rm myimage ls
    ```

    Пояснение к примеру:

    - `--rm` удаляет контейнер после завершения;
    - `ls` в конце команды передается как новая команда запуска.

    Если в образе был `CMD ["nginx", "-g", "daemon off;"]`, то `ls` заменит этот `CMD`, и `nginx` не стартует.
    Это демонстрирует, почему важно понимать поведение `CMD`: его легко переопределить на запуске.

!!! note "Пример с `ENTRYPOINT` + `CMD`"

    ```dockerfile
    ENTRYPOINT ["python", "-m", "http.server"]
    CMD ["8000"]
    ```

    Что это значит:

    - `ENTRYPOINT` задает базовую команду, которая запускается всегда;
    - `CMD` задает аргументы по умолчанию для этой команды.

    Практический эффект:

    - `docker run myimage` запустит `python -m http.server 8000`;
    - `docker run myimage 9000` не заменит `ENTRYPOINT`, а подставит новый аргумент порта (`9000`).

!!! note "Пример с `ARG` (только на этапе сборки)"

    ```dockerfile
    ARG APP_VERSION=dev
    RUN echo "Build version: ${APP_VERSION}" > /build-info.txt
    ```

    Что это значит:

    - значение можно передать при сборке: `docker build --build-arg APP_VERSION=1.4.2 .`;
    - переменная используется только во время `docker build`;
    - после запуска контейнера эта переменная как окружение недоступна сама по себе.

!!! note "Пример с `ENV` (доступно в runtime)"

    ```dockerfile
    ENV APP_ENV=production
    CMD ["sh", "-c", "echo APP_ENV=$APP_ENV && python app.py"]
    ```

    Что это значит:

    - `ENV` попадает в окружение контейнера;
    - приложение может читать эту переменную во время работы;
    - значение можно переопределить на запуске: `docker run -e APP_ENV=staging myimage`.

!!! note "Запустить команду внутри контейнера"

    ```bash
    # Одноразовый контейнер: выполнить команду и удалить контейнер
    docker run --rm myimage ls -la /app

    # Уже запущенный контейнер: выполнить команду внутри него
    docker exec -it my-running-container sh
    ```

    Что это значит:

    - `docker run ... <команда>` запускает новый контейнер и выполняет указанную команду вместо `CMD` образа;
    - `docker exec ... <команда>` выполняет команду в уже работающем контейнере без его пересоздания.

    Когда использовать:

    - `run` — быстрый разовый запуск/проверка;
    - `exec` — диагностика и проверка состояния в текущем работающем контейнере.

</div>

## Multi-stage сборка

```dockerfile
FROM docker.io/library/node:20 AS build
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM docker.io/library/nginx:1.27-alpine
COPY --from=build /app/dist /usr/share/nginx/html
```

**Multi-stage сборка** — это сборка в несколько этапов в одном `Dockerfile`.

Что происходит в примере:

- этап `build` использует `node:20`, устанавливает зависимости и собирает проект (`npm run build`);
- финальный этап использует `nginx:1.27-alpine` и копирует только готовые статические файлы из `/app/dist`.

Зачем это нужно:

- в финальный образ не попадают инструменты сборки (npm, исходники, dev-зависимости);
- образ получается меньше и обычно безопаснее;
- ускоряется доставка и запуск в окружениях CI/CD и проде.

### Best practice: когда выбирать multi-stage

В большинстве production-сценариев multi-stage — предпочтительный подход, но не «жесткое правило всегда».

Когда обычно лучше multi-stage:

- есть отдельный шаг сборки/компиляции (`npm run build`, `go build`, `mvn package`);
- в runtime не нужны инструменты сборки и dev-зависимости;
- важны меньший размер образа, более быстрая доставка и уменьшение поверхности атаки.

Когда допустим один `FROM`:

- у сервиса нет отдельного build-шага и почти нечего «выносить»;
- это локальный прототип, где важнее простота, чем оптимизация образа;
- финальный образ и так минимален, а выигрыш от разделения этапов почти нулевой.

Практическое правило:

- если у вас появляется build-артефакт (например, `dist/`, бинарник, jar), обычно стоит собирать его на первом этапе и копировать во второй этап только нужные файлы через `COPY --from=build ...`.

### Дополнительные примеры для разных стеков

<div class="lenar-tip-grid" markdown="block">

!!! note "Python (сборка wheel -> runtime без build-инструментов)"

    ```dockerfile
    FROM docker.io/library/python:3.12 AS build
    WORKDIR /app
    COPY pyproject.toml poetry.lock ./
    RUN pip install build && python -m build

    FROM docker.io/library/python:3.12-slim
    WORKDIR /app
    COPY --from=build /app/dist/*.whl /tmp/
    RUN pip install --no-cache-dir /tmp/*.whl && rm -rf /tmp/*.whl
    CMD ["python", "-m", "myapp"]
    ```

    Идея: на первом этапе собирается пакет (`wheel`), на втором остается только runtime и установленный пакет.

!!! note "Java (Maven -> JRE)"

    ```dockerfile
    FROM docker.io/library/maven:3.9-eclipse-temurin-21 AS build
    WORKDIR /app
    COPY pom.xml ./
    RUN mvn -q -DskipTests dependency:go-offline
    COPY src ./src
    RUN mvn -q -DskipTests package

    FROM docker.io/library/eclipse-temurin:21-jre
    WORKDIR /app
    COPY --from=build /app/target/*.jar /app/app.jar
    CMD ["java", "-jar", "/app/app.jar"]
    ```

    Идея: компиляция и Maven-кэш остаются в build-стадии, в финальный образ идет только `jar` + JRE.

!!! note "Go (компиляция бинарника -> минимальный runtime)"

    ```dockerfile
    FROM docker.io/library/golang:1.24 AS build
    WORKDIR /src
    COPY go.mod go.sum ./
    RUN go mod download
    COPY . .
    RUN CGO_ENABLED=0 GOOS=linux go build -o /out/app ./cmd/app

    FROM gcr.io/distroless/static-debian12
    COPY --from=build /out/app /app
    ENTRYPOINT ["/app"]
    ```

    Идея: финальный образ содержит только скомпилированный бинарник, без `go` toolchain.

!!! note "Node.js frontend (build -> nginx static)"

    ```dockerfile
    FROM docker.io/library/node:20 AS build
    WORKDIR /app
    COPY package.json package-lock.json ./
    RUN npm ci
    COPY . .
    RUN npm run build

    FROM docker.io/library/nginx:1.27-alpine
    COPY --from=build /app/dist /usr/share/nginx/html
    ```

    Идея: в финальном образе нет Node.js и dev-зависимостей; остаются только статические файлы сайта.

</div>

## Теги и digest в `FROM`: полный практический минимум

В `FROM` можно указывать базовый образ тремя основными способами:

- по имени без тега: `FROM nginx` (неявно берется `latest`);
- по тегу: `FROM nginx:1.27-alpine`;
- по digest: `FROM nginx@sha256:...`.

Что это значит:

- **Тег** (`:1.27-alpine`) — удобная человекочитаемая ссылка на версию, но она может «переехать» на другой образ.
- **Digest** (`@sha256:...`) — криптографический идентификатор конкретного содержимого образа; он неизменяем.

Почему это важно для воспроизводимости:

- если использовать только тег, повторная сборка через неделю может взять уже другой базовый образ;
- если использовать digest, вы гарантированно получите тот же базовый слой, который тестировали раньше.

Практические паттерны:

- локальная разработка: обычно достаточно тега (быстрее обновляться, проще читать);
- CI/CD и production: лучше фиксировать digest, чтобы сборка была детерминированной;
- регулярные обновления безопасности: digest нужно периодически обновлять осознанно (а не «случайно» через плавающий тег).

<div class="lenar-tip-grid" markdown="block">

!!! note "Базовые варианты `FROM`"

    ```dockerfile
    # Удобно читать, но тег может со временем указывать на другой образ
    FROM docker.io/library/nginx:1.27-alpine

    # Максимально воспроизводимо: всегда один и тот же образ
    FROM docker.io/library/nginx@sha256:0123456789abcdef...
    ```

!!! note "Как задать тег образу"

    ```bash
    # Задать тег во время сборки
    docker build -t myapp:1.0 .

    # Добавить еще один тег уже существующему образу
    docker tag myapp:1.0 registry.example.com/team/myapp:1.0
    ```

    Что делает пример:

    - `-t myapp:1.0` назначает имя и тег создаваемому образу;
    - `docker tag ...` создает дополнительный тег (новый «ярлык») для того же image ID.

!!! note "Как получить и проверить digest"

    ```bash
    # Скачать образ по тегу
    docker pull docker.io/library/nginx:1.27-alpine

    # Посмотреть digest локально
    docker image ls --digests

    # Подтянуть строго по digest
    docker pull docker.io/library/nginx@sha256:0123456789abcdef...
    ```

</div>

Рекомендуемый рабочий процесс:

- в разработке используйте читаемый тег;
- в pipeline фиксируйте digest базового образа;
- при плановом обновлении базового образа меняйте digest явно, прогоняйте тесты и только потом выпускайте релиз.

## Частые ошибки

- Копировать весь репозиторий до установки зависимостей и терять кэш.
- Хранить секреты в `ARG`/`ENV` внутри `Dockerfile` (они могут попасть в историю слоев или метаданные образа).
- Использовать shell-форму `CMD` там, где важна корректная обработка сигналов (например, остановка процесса через `SIGTERM`).
- Фиксировать `FROM` только тегом и не проверять `digest` базового образа.

`Digest` — это точный неизменяемый идентификатор образа.  
Тег (`latest`, `1.27`, `3.12-slim`) может указывать на разные версии в разное время, а digest фиксирует конкретный вариант.
