# Linux Network

## Цель
Дать справочный набор для сетевой диагностики Linux: быстро локализовать проблему между DNS, маршрутизацией, портом, локальным listener и firewall.

## Диагностический flow
1. Проверить имя и DNS-резолвинг.
2. Проверить базовую достижимость узла.
3. Проверить маршрут до хоста.
4. Проверить доступность целевого порта.
5. Проверить локальные сокеты и правила фильтрации.

## Что выбирать в типовых развилках
| Сценарий | Команда | Когда использовать |
| --- | --- | --- |
| Нужен быстрый DNS-ответ | `dig +short host` | краткая проверка A/AAAA |
| Нужна детальная DNS-диагностика | `dig host`, `resolvectl query host` | TTL, NS, цепочка резолвинга |
| Проверить порт удаленного сервиса | `nc -zv host 443` | TCP-порт открыт/закрыт |
| Проверить HTTP(S)-слой | `curl -vk https://host` | TLS/HTTP ошибки и коды |
| Проверить локальные listeners | `ss -tulpen` | кто слушает порт локально |
| Проверить путь до узла | `traceroute host`/`tracepath host` | где теряется трафик |

## Матрица симптомов
| Симптом | Проверка | Следующее действие |
| --- | --- | --- |
| Имя не резолвится | `dig host`, `resolvectl status` | проверить DNS-серверы в `resolv.conf` и policy DNS |
| Хост резолвится, но недоступен | `ping -c 4 host`, `traceroute host` | локализовать сегмент потери маршрута |
| Хост доступен, порт закрыт | `nc -zv host port` | проверить firewall/SG и процесс на стороне сервиса |
| Локально сервис не слушает порт | `ss -tulpen | rg :PORT` | проверить unit/процесс и bind address |
| Подключение есть, но HTTP 4xx/5xx | `curl -vk` | перейти к application/log-level диагностике |

## Примеры

<div class="lenar-tip-grid" markdown="block">

!!! note "DNS"

    Краткий ответ резолвера (A/AAAA), без лишнего вывода.

    ```bash
    dig example.com +short
    ```

    Запрос через systemd-resolved (если используется): видно, откуда пришёл ответ.

    ```bash
    resolvectl query example.com
    ```

!!! note "Достижимость и путь"

    ICMP до хоста; если тишина — не значит, что TCP тоже мёртв (ICMP часто режут).

    ```bash
    ping -c 4 example.com
    ```

    На каком хопе теряется или растёт задержка (UDP по умолчанию; на части систем есть `tracepath`).

    ```bash
    traceroute example.com
    ```

!!! note "Порт и HTTP(S)"

    Есть ли TCP-сессия до порта (без проверки содержимого ответа приложения).

    ```bash
    nc -zv example.com 443
    ```

    Уровень HTTP/TLS: редиректы, сертификат, код ответа, тело.

    ```bash
    curl -vk https://example.com/health
    ```

!!! note "Локальный стек IP"

    Адреса и состояние интерфейсов.

    ```bash
    ip a
    ```

    Куда уходит трафик по умолчанию и есть ли нужные префиксы.

    ```bash
    ip route
    ```

    Политики маршрутизации (несколько таблиц, policy routing).

    ```bash
    ip rule
    ```

!!! note "Локальные сокеты"

    Кто слушает порты и с какими опциями (процесс, uid, сокет).

    ```bash
    ss -tulpen
    ```

</div>

## Частые ошибки
- делать вывод только по `ping` (ICMP может быть отключен);
- проверять не тот порт или не тот hostname;
- игнорировать DNS и сразу считать проблему "сетевой";
- проверять только внешний хост и не смотреть локальные listeners/маршруты;
- диагностировать только L3 и не доходить до L7 (`curl -v`).

## Связанные материалы
- [Core Commands](../../core-commands/README.md)
- [Linux Processes](../processes/README.md)
- [Linux Security](../security/README.md)
- [Linux Troubleshooting](../troubleshooting/README.md)
- [Сервис недоступен по сети (рецепт)](../../recipes/service-unreachable-network.md)

## Где использовать
- [Core Commands](../../core-commands/README.md)
- [Подробные страницы команд](../../core-commands/README.md) (`dig`, `ping`, `nc`, `curl`, `ip`, `ss` и др.)
- [Рецепты Linux](../../recipes/README.md)
- [Сервис недоступен по сети](../../recipes/service-unreachable-network.md) — пошаговый runbook
- [Выдача TLS-сертификата Let's Encrypt](../../recipes/letsencrypt-certificate.md)
