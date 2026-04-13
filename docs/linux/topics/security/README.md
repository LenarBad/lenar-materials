# Linux Security

## Цель
Покрыть базовые практики безопасности Linux: минимальные права, обновления, контроль доступа и аудит.

## Baseline hardening checklist
| Область | Минимум | Команда проверки |
| --- | --- | --- |
| SSH | `PermitRootLogin no`, ключи вместо паролей | `sshd -T | rg 'permitrootlogin|passwordauthentication'` |
| Права | least privilege для пользователей и сервисов | `id`, `sudo -l` |
| Обновления | регулярные security updates | `apt list --upgradable` / `dnf check-update` |
| Порты | только нужные listeners | `ss -tulpen` |
| Firewall | deny-by-default + явные allow | `ufw status` / `nft list ruleset` |
| Аудит | логирование auth/privileged событий | `journalctl -u ssh -n 100` |

## Что выбирать в типовых развилках
| Развилка | Рекомендация |
| --- | --- |
| `sudo` или root-shell | использовать `sudo` для отдельных действий; root-shell только для коротких админ-окон |
| Пароль или SSH-ключ | ключи по умолчанию; пароль только как временное исключение с ограничениями |
| `ufw` или `nftables` | `ufw` для простого baseline, `nftables` для детального контроля |
| "Срочный фикс" без логов или с логами | всегда с логами и записью изменений для последующего аудита |

## Runbook: первичный hardening сервера
```bash
# 1) Проверить учетку и sudo-права
id
sudo -l

# 2) Проверить SSH-политику
sudo sshd -T | rg 'permitrootlogin|passwordauthentication|pubkeyauthentication'

# 3) Проверить открытые порты
sudo ss -tulpen

# 4) Проверить firewall
sudo ufw status verbose || sudo nft list ruleset

# 5) Проверить последние auth-события
sudo journalctl -u ssh -n 100 --no-pager
```

## Примеры

<div class="lenar-tip-grid" markdown="block">

!!! note "Пользователи и sudo"

    Кто вы в системе (UID, группы):

    ```bash
    id
    ```

    Что разрешено через `sudo` для текущего пользователя:

    ```bash
    sudo -l
    ```

!!! note "SSH (проверка политики в конфиге)"

    Строки в `sshd_config`, влияющие на root и пароли:

    ```bash
    sudo grep -E "PermitRootLogin|PasswordAuthentication" /etc/ssh/sshd_config
    ```

!!! note "Обновления (Debian/Ubuntu)"

    Обновить индекс пакетов (список доступных версий):

    ```bash
    sudo apt update
    ```

    Установить обновления для уже установленных пакетов:

    ```bash
    sudo apt upgrade -y
    ```

!!! note "Открытые порты"

    Слушающие сокеты, процессы и пользователи (удобно для сверки с firewall):

    ```bash
    sudo ss -tulpen
    ```

</div>

## Частые ошибки
- использовать root для всех операций;
- оставлять парольный вход по SSH там, где можно ключи;
- откладывать обновления безопасности "на потом";
- открывать порт "временно" и забывать закрыть;
- применять hardening без проверки, что сервисы остались доступны.

## Связанные материалы
- [Core Commands](../../core-commands/README.md)
- [Linux Basics](../basics/README.md)
- [Linux Files](../files/README.md)
- [Linux Troubleshooting](../troubleshooting/README.md)

## Где использовать
- [Linux Fundamentals](../../learning/linux-fundamentals.md)
- [Подробные страницы команд](../../core-commands/README.md) (`sudo` и др.)
- [Рецепты Linux](../../recipes/README.md)
- [Выдача TLS-сертификата Let's Encrypt](../../recipes/letsencrypt-certificate.md)
