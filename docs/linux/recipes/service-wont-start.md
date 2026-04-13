# Сервис не запускается или падает после старта

## Индекс

- [Рецепты Linux](README.md)

## Когда использовать

После деплоя, правки конфига или при `systemctl status` в состоянии `failed` / `activating`.

## Пошаговый подход

1. Статус unit и последние сообщения.
2. Логи через journal; при логировании в файл — хвост файла через `tail`.
3. Проверить итоговый unit; при конфликте порта — `ss`.
4. Исправить причину, при смене unit-файла — `daemon-reload`, затем `restart`.

## Команды

Дальше везде `myapp` — заполнитель: подставьте имя своего unit (часто достаточно короткого имени, например `nginx`).

```bash
sudo systemctl status myapp --no-pager
```

```bash
sudo journalctl -u myapp -n 200 --no-pager
```

```bash
sudo systemctl cat myapp
```

Если приложение пишет лог в файл, смотрите конец файла (путь замените на свой):

```bash
sudo tail -n 200 /var/log/myapp/app.log
```

Чтобы непрерывно выводить новые строки (остановка — Ctrl+C):

```bash
sudo tail -f /var/log/myapp/app.log
```

Если лог ротируется и имя файла может пересоздаваться, удобнее следить так:

```bash
sudo tail -F /var/log/myapp/app.log
```

Проверка слушающих портов:

```bash
sudo ss -tulpen
```

После правок unit-файла на диске:

```bash
sudo systemctl daemon-reload
```

Перезапуск сервиса:

```bash
sudo systemctl restart myapp
```

## Проверка результата

```bash
systemctl is-active myapp
```

```bash
sudo journalctl -u myapp -n 50 --no-pager
```

## См. также

- [Рецепты Linux](README.md)
- [Processes](../topics/processes/README.md)
- [Troubleshooting](../topics/troubleshooting/README.md)
- [systemctl](../core-commands/systemctl.md), [journalctl](../core-commands/journalctl.md), [tail](../core-commands/tail.md), [ss](../core-commands/ss.md)
- [Core Commands](../core-commands/README.md)
