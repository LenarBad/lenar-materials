# Выдача TLS-сертификата Let's Encrypt (nginx + Certbot)

## Индекс

- [Рецепты Linux](README.md)

## Введение

Типовой путь: **Certbot с плагином nginx** (`certbot --nginx`) на Debian/Ubuntu.

## Когда использовать

Нужен доверенный HTTPS для домена на сервере с **nginx**, с последующим автоматическим продлением.

## Пошаговый подход

1. Убедиться, что **DNS** (A/AAAA) указывает на этот хост и запись применилась — иначе проверка Let's Encrypt не пройдёт.
2. Установить **Certbot** и плагин **python3-certbot-nginx**.
3. В конфиге nginx для сайта задать **`server_name`** с нужными именами хоста, проверить синтаксис (`nginx -t`), перезагрузить конфиг (`reload`).
4. Обеспечить доступ снаружи к **порту 80** (проверка по HTTP) и **443** (HTTPS): security group в облаке, правила у провайдера, другой брандмауэр. Команды **ufw** из раздела ниже — только если на этом хосте включён именно ufw; иначе раздел про ufw не делают.
5. Запустить **`certbot --nginx -d …`**, ответить на вопросы (в т.ч. про редирект HTTP→HTTPS).
6. Проверить **автопродление**: `certbot.timer` (если ставили из apt) и `certbot renew --dry-run`.

## Команды

Ниже везде вместо **`example.com`** (и при необходимости **`www.example.com`**) подставьте **свой** домен: в `server_name`, в опциях `-d`, в пути `/etc/letsencrypt/live/…`, в аргументах `openssl` и `s_client`.

### Установка и конфигурация nginx

Установка (Debian/Ubuntu):

```bash
sudo apt update
sudo apt install -y certbot python3-certbot-nginx
```

Правка конфига сайта (путь как у вас принято, чаще `sites-available/…`), в блоке `server` указать имена:

```nginx
server_name example.com www.example.com;
```

Проверка и применение конфига nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

### Брандмауэр: пример с ufw (только если на хосте включён ufw)

Если ufw не используете — раздел пропускают; порты уже должны быть открыты по шагу 4.

Когда ufw активен, часто переводят профиль с только HTTP на полный (и 80, и 443), если раньше был только `Nginx HTTP`:

```bash
sudo ufw allow 'Nginx Full'
sudo ufw delete allow 'Nginx HTTP'
```

### Выпуск сертификата и проверки

Выпуск и подключение:

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

Проверка таймера автопродления (типично для пакета из apt):

```bash
sudo systemctl status certbot.timer
```

Тест продления без смены сертификата:

```bash
sudo certbot renew --dry-run
```

Проверка сертификата на сервере (путь после выпуска обычно такой):

```bash
sudo openssl x509 -in /etc/letsencrypt/live/example.com/cert.pem -noout -dates -subject
```

С другой машины:

```bash
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -dates -subject
```

## Проверка результата

- Браузер открывает `https://…` без предупреждений о сертификате.
- `renew --dry-run` завершается без ошибки; по желанию `systemctl status certbot.timer` показывает активный таймер.

## См. также

- [Рецепты Linux](README.md)
- [Security](../topics/security/README.md)
- [Network](../topics/network/README.md)
- [systemctl](../core-commands/systemctl.md), [ss](../core-commands/ss.md), [curl](../core-commands/curl.md)
- [Core Commands](../core-commands/README.md)
