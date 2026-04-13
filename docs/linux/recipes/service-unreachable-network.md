# Сервис недоступен по сети (с хоста или извне)

## Индекс

- [Рецепты Linux](README.md)

## Когда использовать

Таймауты, `connection refused`, подозрение на DNS, firewall или не тот порт.

## Пошаговый подход

1. Проверить резолвинг имени и что вы стучитесь в тот хост и порт, который ожидаете.
2. Проверить доступность порта с клиента и ответ на уровне протокола (например HTTP).
3. На стороне сервера убедиться, что процесс слушает нужный адрес и порт.
4. Проверить firewall и правила в облаке только после того, как локально сервис слушает верно.

## Команды

```bash
dig +short example.com
```

```bash
nc -zv example.com 443
```

```bash
curl -vk https://example.com/health
```

```bash
sudo ss -tulpen
```

## Проверка результата

С клиента успешное подключение к порту и ожидаемый ответ приложения (например health-check).

## См. также

- [Рецепты Linux](README.md)
- [Network](../topics/network/README.md)
- [Security](../topics/security/README.md)
- [dig](../core-commands/dig.md), [nc](../core-commands/nc.md), [curl](../core-commands/curl.md), [ss](../core-commands/ss.md)
- [Core Commands](../core-commands/README.md)
