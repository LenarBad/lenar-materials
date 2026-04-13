# Логи за окно инцидента

## Индекс

- [Рецепты Linux](README.md)

## Когда использовать

Нужно сопоставить время сбоя с событиями в journal или файловых логах.

## Пошаговый подход

1. Journal с границами времени (одна зона: UTC или локальная — как договорились в команде).
2. Для текстовых логов — последние строки `tail -n` или поток `tail -F`, если смотрите событие «вживую».
3. Системные ошибки за текущую загрузку: `journalctl` с фильтром приоритета.

## Команды

Границы времени и имя unit подставьте свои:

```bash
sudo journalctl -u myapp --since "2026-04-11 10:00" --until "2026-04-11 11:00" --no-pager
```

```bash
sudo journalctl -b -p err..alert --no-pager | tail -n 50
```

```bash
sudo tail -n 200 /var/log/syslog
```

Слежение за файлом при ротации (путь свой):

```bash
sudo tail -F /var/log/myapp/app.log
```

## Проверка результата

Есть связная цепочка событий до и после сбоя; можно сослаться на строки в постмортеме.

## См. также

- [Рецепты Linux](README.md)
- [Troubleshooting](../topics/troubleshooting/README.md)
- [journalctl](../core-commands/journalctl.md), [tail](../core-commands/tail.md)
- [Core Commands](../core-commands/README.md)
