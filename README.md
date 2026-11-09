# 3x-ui-pro

🇬🇧 [English version](README_EN.md)

Автоматическая установка панели [3x-ui](https://github.com/MHSanaei/3x-ui) с nginx, SSL, Clash-подпиской и диагностикой сети.

- Debian 12 / Ubuntu 24
- Достаточно одного домена или поддомена (для панели и подписок)
- Защита от блокировок ТСПУ / DPI (РКН): VLESS + REALITY + XTLS-Vision + Chrome uTLS
- Автоматический подбор и верификация доверенных сайтов-маскировок (TLS 1.3 + H2)
- Автоматическое получение и продление SSL-сертификатов Let's Encrypt для панели
- Поддержка VLESS+REALITY, VLESS+WebSocket, VLESS+XHTTP, Trojan+gRPC — всё через единый порт 443

---

## Что устанавливается

| Компонент | Описание |
|-----------|----------|
| 3x-ui | VPN-панель с веб-интерфейсом |
| nginx | Обратный прокси, SNI-роутинг |
| certbot | Let's Encrypt SSL |
| Clash-подписка | Автоматическая выдача `clash.yaml` по User-Agent |
| Диагностика | MTR-трейсер + тест скорости в браузере |
| Фейковый сайт | Случайный HTML-сайт-прикрытие |
| Бэкап | Скрипт резервного копирования |
| AdGuard Home | Опционально: DNS с блокировкой рекламы (DoH) — отдельный скрипт |

---

## Установка

**Шаг 1 — скачать скрипт**

```bash
wget -qO x-ui-latest.sh https://raw.githubusercontent.com/mozaroc/3x-ui-pro/main/x-ui-latest.sh
```

**Шаг 2 — запустить**

```bash
bash x-ui-latest.sh
```

---
## Патч

Применить текущие фиксы к существующей установке (без изменений БД):

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/mozaroc/3x-ui-pro/main/x-ui-patch.sh)
```

---

## AdGuard Home (опционально)

Устанавливает [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) на домен панели — без отдельного домена и открытых портов, всё через существующий 443:

- **DNS-over-HTTPS** для клиентов: `https://<домен-панели>/dns-query`
- **Админка** — на случайном пути `/adg-<random>/` (логин и пароль выводит скрипт)

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/mozaroc/3x-ui-pro/main/x-ui-adguard.sh)
```

Повторный запуск безопасен (настройки и пароль сохраняются). После установщика или патча запустите скрипт ещё раз — они перезаписывают конфиг nginx.

Удаление:

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/mozaroc/3x-ui-pro/main/x-ui-adguard.sh) -uninstall y
```

---

## Удаление

```bash
bash x-ui-latest.sh -uninstall y
```

---

## Параметры запуска

| Параметр | Описание |
|----------|----------|
| `-install n` | Пропустить установку системных пакетов (по умолчанию `y`) |
| `-subdomain <домен>` | Домен панели и подписок |
| `-reality_target <домен>` | Внешний домен для маскировки REALITY (по умолчанию автоподбор проверенного, например `gateway.icloud.com`) |
| `-reality_domain <домен>` | Устаревший синоним `-reality_target` |
| `-auto_domain y` | Автоматический выбор проверенного целевого домена REALITY |
| `-version <версия>` | Установить конкретную версию 3x-ui (например `3.4.2`), по умолчанию — последняя |
| `-uninstall y` | Полное удаление |

---

## Clash-подписка

Работает через определение User-Agent — один URL, разное поведение:

- **Clash / Mihomo / Stash** → получают `clash.yaml` с готовой конфигурацией
- **Обычный браузер / другие клиенты** → получают стандартную страницу подписки 3x-ui

Ссылку для импорта выводит скрипт после установки.

---

## Бэкап и восстановление

**Установить скрипт бэкапа**

```bash
wget -qO /usr/local/bin/x-ui-backup https://raw.githubusercontent.com/mozaroc/3x-ui-pro/main/assets/backup/x-ui-backup.sh
chmod +x /usr/local/bin/x-ui-backup
```

**Создать бэкап**

```bash
x-ui-backup backup
```

**Список бэкапов**

```bash
x-ui-backup list
```

**Восстановить из бэкапа** (на чистом сервере, пакеты ставятся автоматически)

```bash
x-ui-backup restore /var/backups/x-ui/x-ui-backup-20260101-120000.tar.gz
```

Бэкап включает: конфиги nginx, БД панели, бинарник 3x-ui, SSL-сертификаты, веб-контент, systemd-юниты, cron, правила UFW.

---

## Диагностика сети

После установки доступна по ссылке, которую выводит скрипт. Включает:

- MTR-трейс до вашего IP
- Тест скорости загрузки и отдачи (512 МБ файлы)
