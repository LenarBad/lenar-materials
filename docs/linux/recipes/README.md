# Рецепты Linux

## О разделе

Короткие сценарии для повседневного администрирования: что делать в первую очередь и как убедиться, что результат верный. Подробности и справка — в тематических страницах ниже.

## Тематические источники

- [Basics](../topics/basics/README.md) — контекст (`whoami`, пути, безопасность в shell)
- [Files](../topics/files/README.md) — FHS и типичные пути (конфиги, логи, unit); операции с файлами, права, поиск, место на диске
- [Processes](../topics/processes/README.md) — процессы и systemd
- [Network](../topics/network/README.md) — сетевая диагностика
- [Security](../topics/security/README.md) — sudo, SSH, обновления, порты
- [Troubleshooting](../topics/troubleshooting/README.md) — разбор инцидентов

## Логи: `journalctl` и `tail`

Оба инструмента удобны, задачи разные:

- **`journalctl`** — основной вариант для логов **systemd** (юниты, загрузка, агрегированный журнал). Удобны фильтры по времени, unit, приоритету.
- **`tail`** — естественный выбор для **обычных файлов** в `/var/log/...` и логов приложений, которые пишут в файл: последние строки (`tail -n`), слежение в реальном времени (`tail -f` или `tail -F` после ротации).

Менее удобной `tail` не является; просто для сервисов под systemd чаще быстрее начать с `journalctl -u`.

См. также: [tail](../core-commands/tail.md), [journalctl](../core-commands/journalctl.md).

## Оглавление

- [Сервис не запускается или падает после старта](service-wont-start.md)
- [Нехватка места на диске](disk-space-low.md)
- [Безопасная чистка старых логов](safe-old-logs-cleanup.md)
- [«Тяжёлый» процесс: CPU или память](heavy-process-resources.md)
- [Сервис недоступен по сети (с хоста или извне)](service-unreachable-network.md)
- [Обновления безопасности (один хост)](security-updates-single-host.md)
- [Выдача TLS-сертификата Let's Encrypt](letsencrypt-certificate.md)
- [Правка конфига с возможностью отката](config-edit-with-rollback.md)
- [Логи за окно инцидента](logs-incident-window.md)
- [Быстрый pre-flight перед опасной командой](preflight-dangerous-commands.md)

## См. также

- [Core Commands](../core-commands/README.md)
- [Linux Fundamentals](../learning/linux-fundamentals.md)
