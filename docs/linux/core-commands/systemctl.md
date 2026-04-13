# systemctl

## Назначение

Управлять **units** в **systemd**: сервисами, сокетами, таймерами и др. Старт, остановка, автозапуск, просмотр итоговой конфигурации — основной интерфейс администратора на современных дистрибутивах Linux.

## TLDR

Показать состояние unit.

```bash
systemctl status nginx
```

Перезапустить сервис (после смены конфига, если не подходит `reload`).

```bash
sudo systemctl restart nginx
```

Включить автозапуск при загрузке системы.

```bash
sudo systemctl enable nginx
```

Показать итоговый unit-файл с drop-in.

```bash
systemctl cat nginx
```

Перезагрузить конфигурацию systemd после правок unit-файлов.

```bash
sudo systemctl daemon-reload
```

Список активных сервисов.

```bash
systemctl list-units --type=service --state=running
```

## Синтаксис

```text
systemctl [опции] команда [имя ...]
```

Имя unit часто указывают с суффиксом (`.service`), но многие подкоманды принимают короткое имя.

## Поведение и параметры

Подкоманды задают действие: **`status`**, **`start`**, **`stop`**, **`restart`**, **`reload`**, **`enable`**, **`disable`**, **`cat`**, **`edit`**, **`daemon-reload`** и др. Часть операций изменяет состояние системы и требует **root** (через `sudo`).

| Подкоманда / опция | Что делает |
| --- | --- |
| `status [unit]` | Текущее состояние, последние строки лога, код выхода |
| `start` / `stop` / `restart` | Управление жизненным циклом |
| `reload` | Перечитать конфиг без полного рестарта (если unit поддерживает) |
| `enable` / `disable` | Автозапуск при boot |
| `cat` / `show` | Конфигурация и свойства unit |
| `daemon-reload` | Перечитать определения unit после изменений на диске |

После правки файлов в `/etc/systemd/system` обычно нужны **`daemon-reload`** и затем **`restart`** (или указанный в документации шаг).

## Примеры

Проверить, что сервис включён и активен.

```bash
systemctl is-enabled sshd
systemctl is-active sshd
```

## См. также

- `journalctl`, `loginctl`, `systemd-analyze`
- `man systemctl`

## Связанные материалы

- [Core Commands](README.md)
- [Linux Processes](../topics/processes/README.md)
