# Lenar Materials Workspace

Этот workspace настроен под модель:
- контент хранится в Markdown;
- сайт собирается через MkDocs Material;
- публикация выполняется через GitHub Pages (GitHub Actions).

## Локальный запуск
1. `python3 -m venv .venv`
2. `source .venv/bin/activate`
3. `pip install -r requirements.txt`
4. `mkdocs serve`

Локальный адрес обычно: `127.0.0.1:8000`.

## Сборка
- `mkdocs build`

Результат сборки: каталог `site/`.

## Публикация
- workflow `.github/workflows/docs-pages.yml` запускается при push в `main`.
- GitHub Pages получает статический сайт из `site/`.

## Правила архитектуры
- Для агентных сессий действует правило:
  - `.cursor/rules/project-knowledge-architecture.mdc`
- Целевая архитектура и подходы:
  - `docs/PROJECT_APPROACHES.md`
