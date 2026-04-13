# journalctl

## Назначение

Читать **журнал systemd** (`journal`): логи сервисов, ядра и системных компонентов с фильтрами по unit, времени и уровню. Основной инструмент разбора инцидентов рядом с `systemctl status`.

## TLDR

Последние 100 строк лога для unit.

```bash
journalctl -u nginx -n 100 --no-pager
```

Логи с момента времени (удобно после инцидента).

```bash
journalctl -u myapp --since "2026-04-10 10:20:00" --no-pager
```

Следить за новыми строками в реальном времени.

```bash
journalctl -u nginx -f
```

Все загрузки системы (boot), кратко.

```bash
journalctl --list-boots
```

Логи текущей загрузки.

```bash
journalctl -b
```

Только ошибки и более серьёзные уровни (зависит от версии; см. man).

```bash
journalctl -p err -b --no-pager
```

## Синтаксис

```text
journalctl [опции]
```

Без фильтров вывод может быть **очень большим**; ограничивайте `-u`, `--since`, `-n` или используйте pager осознанно.

## Поведение и параметры

Журнал хранится бинарно; `journalctl` форматирует записи для терминала. По умолчанию часто открывается **pager** (`less`); для скриптов и pipe используйте **`--no-pager`**.

| Параметр | Что делает |
| --- | --- |
| `-u unit` | Фильтр по systemd unit (например `nginx.service`) |
| `-n N` | Показать только последние N строк |
| `-f`, `--follow` | Поток новых записей (как `tail -f`) |
| `--since` / `--until` | Интервал времени (строка или относительная форма) |
| `-b` | Текущая загрузка; `-b -1` — предыдущая |
| `--no-pager` | Вывести всё в stdout без pager |
| `-o cat` | Упрощённый вывод сообщения |

Форматы **`--since`** описаны в `man journalctl` (`yesterday`, `1 hour ago`, ISO-время и т.д.).

## Примеры

Сузить окно по времени и сразу отдать в файл для тикета.

```bash
journalctl -u myapp --since "2026-04-11 08:00" --until "2026-04-11 09:00" --no-pager > /tmp/myapp-incident.log
```

## См. также

- `systemctl`, `dmesg`, `logger`
- `man journalctl`

## Связанные материалы

- [Core Commands](README.md)
- [Linux Processes](../topics/processes/README.md)
- [Linux Troubleshooting](../topics/troubleshooting/README.md)
