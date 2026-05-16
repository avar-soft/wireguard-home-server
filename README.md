<div align="center">

<a name="readme-top"></a>

**[🇷🇺 Русский](#русский) · [🇬🇧 English](#english)**

---

</div>

---

<a name="русский"></a>

<div align="center">

# 🛡️ WireGuard Home Server

**Профессиональный WireGuard-сервер с умной geo-маршрутизацией**

[![Version](https://img.shields.io/badge/версия-28.24.21-blue?style=for-the-badge&logo=github)](https://github.com/avar-soft/wireguard-home-server)
[![Bash](https://img.shields.io/badge/Bash-4.3+-green?style=for-the-badge&logo=gnu-bash)](https://www.gnu.org/software/bash/)
[![License](https://img.shields.io/badge/лицензия-MIT-orange?style=for-the-badge)](LICENSE)
[![Platform](https://img.shields.io/badge/платформа-Ubuntu%20%7C%20Debian-purple?style=for-the-badge&logo=linux)](https://ubuntu.com)

*Один скрипт — полный VPN-сервер. Один трафик идёт напрямую, другой через туннель.*  
*GeoIP · Split-DNS · Anti-DPI · Balancer.*

> 🇬🇧 [Switch to English](#english)

---

</div>

## 🤔 Зачем это нужно?

Обычный VPN отправляет **весь** трафик через сервер — это медленно и дорого. Этот скрипт умнее:

- 🇷🇺 **Локальные сайты** — идут **напрямую**, без туннеля, на полной скорости
- 🌍 **Другие ресурсы** — автоматически **через туннель**, без лишних задержек
- ⚖️ **Несколько туннелей** — балансировщик выбирает самый быстрый автоматически

База РФ IP-адресов содержит **8500+ подсетей** (IPv4 и IPv6) и обновляется автоматически.

---

## ✨ Возможности

### 🗺️ Умная маршрутизация

| Режим | Описание |
|-------|----------|
| **🌍 Geo-split** *(по умолчанию)* | РФ напрямую, всё остальное через туннель |
| **🔒 Full-VPN** | Весь трафик через туннель (классический VPN) |
| **🏠 Direct-only** | Весь трафик напрямую (VPN временно выключен) |

Профили переключаются **мгновенно** из меню без перезапуска сервисов.

---

### 🔥 GeoIP — интеллектуальная база IP

- 📦 **Batch-загрузка** — 8500+ подсетей загружаются пачками по 1000, не перегружая сервер
- 🗂️ **Offline-режим** — положи `ru-aggregated.zone` рядом, интернет не нужен
- 🔄 **Авто-обновление** — cron-задача обновляет базу по расписанию
- 🌐 **IPv4 + IPv6** — полная поддержка обоих стеков (`@russia` + `@russia_v6` в nftables)
- ⚡ **nftables** — современный файрвол вместо устаревшего iptables, атомарные обновления

---

### ⚖️ Балансировщик туннелей

- 📡 **До 5 туннелей** одновременно — подключи VPS в разных странах
- 🏓 **Пинг-мониторинг** — каждые N секунд измеряет задержку до каждого туннеля
- 🔀 **Автопереключение** — если активный туннель деградировал (`> 200ms`), трафик уходит на лучший
- 🔁 **Полный цикл** — раз в 5 минут сравниваются все туннели, выбирается абсолютно лучший
- 📊 **Watchdog** — systemd-сервис, восстанавливает правила маршрутизации после перезагрузки

---

### 🌐 Split-DNS

Три режима DNS, переключаемые из меню:

```
geo     → Яндекс DNS (77.88.8.8) для РФ-зон, 8.8.8.8 для остального
tunnel  → Весь DNS через туннель + DoT-шифрование через stubby
public  → 8.8.8.8 / 8.8.4.4 напрямую (максимальная скорость)
```

DNS-сервер — **dnsmasq** с раздельной обработкой зон. В режиме `tunnel` апстримом служит локальный **stubby** (DNS-over-TLS), трафик которого уходит через VPN-туннель.

---

### 🔐 DNS-over-TLS

Все DNS-запросы с сервера шифруются через **stubby** — локальный DoT-резолвер (порт 5353).

**Схема в tunnel-режиме:**
```
Клиент → dnsmasq → stubby (127.0.0.1:5353) → DoT (порт 853) → [через туннель] → Cloudflare / Quad9
```

Провайдер и хостер видят только зашифрованное TLS-соединение.

**Пресеты резолверов:**

| Пресет | Адреса | Особенность |
|--------|--------|-------------|
| ☁️ **Cloudflare** | 1.1.1.1 / 1.0.0.1 | Максимальная скорость |
| 🛡️ **Quad9** | 9.9.9.9 | Блокировка malware/фишинга |
| 🔍 **Google** | 8.8.8.8 / 8.8.4.4 | Классика, высокая надёжность |
| 🚫 **AdGuard** | 94.140.14.14 | Блокировка рекламы и трекеров |
| 🌐 **Все три** | Cloudflare + Quad9 + Google | Round-robin, максимальная отказоустойчивость |

---

### 🔒 Killswitch

Если VPN-соединение обрывается, killswitch **блокирует весь незащищённый трафик**:

- Клиенты не могут выйти в интернет минуя туннель
- Защита от утечки реального IP
- Включается/выключается из меню в одно действие
- Не затрагивает SSH-доступ к серверу

---

### 📊 Лимиты трафика

- 📏 **Лимит по клиенту** — задаётся в GB (суммарный входящий + исходящий)
- ⚡ **Кумулятивный учёт** — работает корректно при перезагрузках и сбросах счётчиков WireGuard
- 🚨 **Два режима превышения**: отключить клиента или только записать в лог
- 🔄 **Watcher** — проверяет каждые 15 минут через cron
- 🗑️ **Сброс счётчиков** — при изменении или снятии лимита счётчики обнуляются автоматически

---

### 🔑 Ротация ключей

- Смена ключей шифрования без потери конфигурации
- Автоматическая раздача новых конфигов клиентам
- PSK (Pre-Shared Key) для дополнительного слоя безопасности

---

### 💾 Бэкап и восстановление

Бэкап сохраняет:
- `/etc/wireguard/` — все конфиги WireGuard
- `/etc/nftables.conf` — правила файрвола
- `/etc/dnsmasq.d/` — конфиги DNS
- `/etc/systemd/system/` — юниты сервисов
- `/etc/telemt.toml` — конфиг MTProxy
- Скрипты из `/usr/local/bin/`

Хранятся последние **10 архивов**, старые удаляются автоматически. Восстановление — одно действие из меню.

---

## 📋 Требования

| Параметр | Значение |
|----------|----------|
| **ОС** | Ubuntu 24.04+ |
| **Права** | root |
| **Bash** | ≥ 4.3 |
| **Архитектура** | amd64 |
| **RAM** | ≥ 512 MB |
| **Диск** | ≥ 1 GB свободного места |
| **Сеть** | Публичный IP (VPS/выделенный сервер) |

**Зависимости устанавливаются автоматически:**  
`wireguard` · `nftables` · `curl` · `iproute2` · `qrencode` · `dnsmasq` · `openssl` · `stubby`

---

## ⚡ Быстрый старт

### Установка за 1 команду

```bash
curl -fsSL https://raw.githubusercontent.com/avar-soft/wireguard-home-server/main/wg-server.sh -o wg-server.sh \
  && chmod +x wg-server.sh \
  && sudo bash wg-server.sh
```

При первом запуске скрипт:
1. Устанавливает зависимости
2. Задаёт несколько вопросов (интерфейс, порт, подсеть)
3. Генерирует ключи WireGuard
4. Настраивает nftables + GeoIP
5. Запускает все сервисы
6. Открывает главное меню

**Весь процесс занимает ~2 минуты.**

### Последующие запуски

```bash
sudo bash wg-server.sh
```

---

## 📖 Использование

### Добавление клиента

```
Главное меню → 1) Клиенты → 1) Добавить клиента
```

Введи имя устройства (например, `iphone` или `work-laptop`). Скрипт:
- Генерирует пару ключей
- Выдаёт IP из подсети
- Показывает QR-код прямо в терминале
- Сохраняет `.conf` файл в `/etc/wireguard/clients/`

QR-код сканируется приложением WireGuard (iOS / Android) — готово.

---

### Добавление туннеля (внешний VPS)

```
Главное меню → 2) Туннели → Добавить туннель
```

Нужно ввести:
- **Endpoint** — `ip:port` или `hostname:port` внешнего VPS
- **Public Key** — публичный ключ WireGuard на внешнем сервере
- **PSK** (опционально) — pre-shared key для усиления шифрования
- **MTU** — обычно 1420, для некоторых провайдеров 1280

Балансировщик автоматически начнёт мониторить новый туннель.

---

### Подключение клиента

**iOS / Android:** отсканируй QR-код из меню → готово.

**Linux:**
```bash
scp root@server:/etc/wireguard/clients/mydevice.conf .
sudo wg-quick up ./mydevice.conf
```

**Windows / macOS:** импортируй `.conf` файл в официальное приложение WireGuard.

---

## 🔧 Интерактивное меню

```
╔═════════════════════════════════════════════════════════════╗
║      🛡️   WireGuard Home Server  —  v28.24.21               ║
║   РФ напрямую │ Зарубежье в туннель │ Split-DNS │ Anti-DPI  ║
╚═════════════════════════════════════════════════════════════╝

  ┌─────────────────────────────────────────────────────────────┐
  │  КЛИЕНТЫ И МАРШРУТИЗАЦИЯ                                    │
  └─────────────────────────────────────────────────────────────┘
   1)  👤  Клиенты               — добавить, QR-код, список, отозвать
   2)  🔀  Туннели               — VPN-соединения до внешних серверов
   3)  🔥  Файрвол / GeoIP       — правила, список РФ IP, обновление
   4)  🗺️   Профили маршрутизации — geo-split / full-vpn / direct-only
   5)  📌  Прямые IP (whitelist) — свои подсети минуя VPN
   6)  🔒  Killswitch            — блокировка при обрыве VPN
   7)  🌐  DNS (geo/tunnel/pub)  — Split-DNS + DNS-over-TLS через stubby
   8)  🛡   Anti-DPI              — обфускация трафика от DPI-систем

  ┌─────────────────────────────────────────────────────────────┐
  │  МОНИТОРИНГ                                                 │
  └─────────────────────────────────────────────────────────────┘
   9)  📊  Статус WireGuard      — пиры, трафик, последний онлайн
  10)  ⚖️   Балансировщик         — watchdog, логи, настройки
  11)  📈  Мониторинг            — трафик клиентов, пинги, live-монитор
  12)  📊  Лимиты трафика        — ограничения по GB на клиента
  13)  📄  Логи                  — журналы всех сервисов
  14)  ✅  Тест системы          — автопроверка что всё работает

  ┌─────────────────────────────────────────────────────────────┐
  │  ПРОЧЕЕ                                                     │
  └─────────────────────────────────────────────────────────────┘
  15)  🦀  Telemt (новый MTProxy)       — Rust, Fake TLS, multi-user
  16)  📡  MTProto Proxy (старый MTG)   — Go-версия, для совместимости
  17)  🔑  Ротация ключей               — смена ключей шифрования
  18)  💾  Бэкап / Восстановление       — сохранить и восстановить конфиги
  19)  🚀  Автозапуск                   — настройка запуска при старте сервера
  20)  🔄  Обновить скрипт              — загрузить новую версию
  21)  🔄  Перезапустить всё
  22)  💣  Удалить всё (НЕОБРАТИМО!)
   0)  🚪  Выход
```

---

## 🔬 Тест системы

Меню **14) Тест системы** проверяет всю цепочку одной командой:

```
── Сервисы ──
  WireGuard сервер (wg0) активен                         ✔ OK
  Балансировщик wg-balance активен                       ✔ OK
  nftables правила загружены                             ✔ OK

── Туннели ──
  Туннель wg1 активен                                    ✔ OK
  Связь с сервером nl.example.com                        ✔ OK

── Маршрутизация ──
  fwmark правило РФ трафика                              ✔ OK
  GeoIP база РФ (8500+ сетей) загружена                  ✔ OK
  systemd units валидны (verify)                         ✔ OK

── Интернет ──
  Ping / связь 8.8.8.8                                   ✔ OK
  DNS резолвинг google.com                               ✔ OK

✔ Все тесты пройдены (15/15)
```

---

## 🔐 Безопасность

- **`set -uo pipefail`** — немедленная остановка при необъявленных переменных
- **Валидация всех входных данных** — имена клиентов, порты, пути, IP-адреса
- **`safe_sed()`** — все замены в конфигах через экранирующую обёртку, инъекции невозможны
- **Именованные функции** вместо `bash -c` — нет интерполяции в строку команды
- **Атомарные операции** — конфиги перезаписываются через `mktemp` + `mv`
- **ERR-trap** — все ненулевые коды возврата логируются с контекстом
- **Бэкап перед опасными операциями** — предлагается автоматически

---

## 📁 Структура файлов

```
/etc/wireguard/
├── wg0.conf                    # Основной WireGuard интерфейс
├── wg1.conf, wg2.conf...       # Туннели
├── .wg-setup.conf              # Конфигурация скрипта
├── .traffic-limits             # Лимиты трафика по клиентам
├── clients/
│   ├── phone.conf
│   └── laptop.conf
└── geoip/
    ├── ru-aggregated.zone      # Offline IPv4 база
    └── ru-aggregated-v6.zone   # Offline IPv6 база

/usr/local/bin/
├── wg-balance.sh               # Балансировщик туннелей
├── update-ru-ipset.sh          # Обновление GeoIP
├── wg-traffic-limit.sh         # Watchdog лимитов трафика
└── telemt                      # Бинарь MTProxy

/etc/telemt.toml                # Конфиг Telemt MTProxy
/etc/stubby/stubby.yml          # Конфиг DNS-over-TLS
/var/lib/wg/limits/             # State-файлы счётчиков трафика
/var/backups/wg-home/           # Бэкапы конфигурации
/var/log/wg-server-trap.log     # Лог ошибок скрипта
```

---

## ⚙️ Переменные окружения

```bash
export WG_VERSION_URL="https://example.com/wg/version.txt"  # URL для авто-обновления
export EDITOR="nano"          # nano | vim | vi
export WG_DEBUG="1"           # Расширенное логирование
export WG_ERR_LOG="/var/log/wg-server-trap.log"
export TELEMT_REPO="telemt/telemt"  # Форк Telemt
```

---

## 🔄 Обновление

```
Главное меню → 20) Обновить скрипт
```

Или вручную — просто запусти новую версию. Все конфиги сохраняются. Обновление происходит **атомарно** через `install` + `mv`.

---

## 🗑️ Удаление

```bash
sudo bash wg-server.sh --remove
```

Или через меню **22) Удалить всё**. Удаляет интерфейсы, правила nftables, systemd-сервисы, конфиги. Пакеты не трогаются.

---

## 📝 Журнал изменений

<details>
<summary><b>v28.24.21</b> — текущая версия</summary>

- 🐛 Очистка state-файлов лимитов трафика при снятии/изменении лимита
- 🛡️ `_statusBar()` защищена от `set -u` при вызове вне `menu()`
- 🔧 `restoreBackup`: `trap RETURN` вместо ручного удаления tmpdir
- 🔧 `geo4`/`geo6` pipelines: добавлен `|| true` против `pipefail`

</details>

<details>
<summary><b>v28.24.20</b></summary>

- 🔧 Смена порта Telemt: `sed` → `safe_sed`
- 🔧 `installTelemt`: проверка занятости альтернативного порта

</details>

<details>
<summary><b>v28.24.19</b></summary>

- 🛡️ `runSystemTest`: убран `bash -c`, именованные `_chk_*` функции
- 🛡️ `menuTelemetConfig`: `sed` → `safe_sed` для `tls_domain`
- 🐛 Валидация `UNAME` перед `_telemt_removeUser`

</details>

<details>
<summary><b>v28.24.18</b></summary>

- 🐛 Исправлена регрессия `bash -c` в тестах системы
- 🔧 Обновлён балансировщик туннелей

</details>

---

## 🤝 Участие в разработке

Pull requests приветствуются. Для значительных изменений — открой Issue.

**Правила:**
- Весь bash через `set -uo pipefail`
- Пользовательский ввод — только через `safe_sed()` и валидацию
- Временные файлы — через `mktemp` с `trap` на очистку
- Комментарии к нетривиальным решениям обязательны

---

## 📄 Лицензия

MIT © [avar-soft](https://github.com/avar-soft)

---

<div align="center">

**Сделано с ❤️ для тех, кто ценит приватность и скорость**

⭐ Если проект полезен — поставь звезду!

[🔝 Наверх](#readme-top) · [🇬🇧 English version](#english)

</div>

---
---

<a name="english"></a>

<div align="center">

# 🛡️ WireGuard Home Server

**Professional WireGuard server with smart geo-routing**

[![Version](https://img.shields.io/badge/version-28.24.21-blue?style=for-the-badge&logo=github)](https://github.com/avar-soft/wireguard-home-server)
[![Bash](https://img.shields.io/badge/Bash-4.3+-green?style=for-the-badge&logo=gnu-bash)](https://www.gnu.org/software/bash/)
[![License](https://img.shields.io/badge/license-MIT-orange?style=for-the-badge)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-Ubuntu%20%7C%20Debian-purple?style=for-the-badge&logo=linux)](https://ubuntu.com)

*One script — a complete VPN server. Some traffic goes direct, some through the tunnel.*  
*GeoIP · Split-DNS · Anti-DPI · Balancer.*

> 🇷🇺 [Переключиться на русский](#русский)

---

</div>

## 🤔 Why?

A regular VPN routes **all** traffic through the server — slow and expensive. This script is smarter:

- 🇷🇺 **Local sites** — go **directly**, no tunnel, full speed
- 🌍 **Everything else** — automatically **through the tunnel**, no extra latency
- ⚖️ **Multiple tunnels** — load balancer picks the fastest one automatically

The Russian IP database contains **8500+ subnets** (IPv4 and IPv6), updated automatically.

---

## ✨ Features

### 🗺️ Smart Routing

| Mode | Description |
|------|-------------|
| **🌍 Geo-split** *(default)* | RU traffic direct, everything else through tunnel |
| **🔒 Full-VPN** | All traffic through tunnel (classic VPN) |
| **🏠 Direct-only** | All traffic direct (VPN temporarily disabled) |

Profiles switch **instantly** from the menu without restarting services.

---

### 🔥 GeoIP — intelligent IP database

- 📦 **Batch loading** — 8500+ subnets loaded in chunks of 1000, no server overload
- 🗂️ **Offline mode** — place `ru-aggregated.zone` locally, no internet required
- 🔄 **Auto-update** — cron job updates the database on schedule
- 🌐 **IPv4 + IPv6** — full dual-stack support (`@russia` + `@russia_v6` sets in nftables)
- ⚡ **nftables** — modern firewall instead of legacy iptables, atomic updates

---

### ⚖️ Tunnel Balancer

- 📡 **Up to 5 tunnels** simultaneously — connect VPS in different countries
- 🏓 **Ping monitoring** — measures latency to each tunnel every N seconds
- 🔀 **Auto-failover** — if the active tunnel degrades (`> 200ms`), traffic shifts to the best one
- 🔁 **Full cycle** — all tunnels compared every 5 minutes, absolute best selected
- 📊 **Watchdog** — systemd service that restores routing rules after reboot

---

### 🌐 Split-DNS

Three DNS modes, switchable from the menu:

```
geo     → Yandex DNS (77.88.8.8) for RU zones, 8.8.8.8 for everything else
tunnel  → All DNS through tunnel + DoT encryption via stubby
public  → 8.8.8.8 / 8.8.4.4 direct (maximum speed)
```

DNS server — **dnsmasq** with split zone handling. In `tunnel` mode, upstream is local **stubby** (DNS-over-TLS), whose traffic goes through the VPN tunnel.

---

### 🔐 DNS-over-TLS

All DNS queries from the server are encrypted via **stubby** — a local DoT resolver (port 5353).

**Flow in tunnel mode:**
```
Client → dnsmasq → stubby (127.0.0.1:5353) → DoT (port 853) → [via tunnel] → Cloudflare / Quad9
```

Your ISP and hosting provider only see encrypted TLS connections — no queries, no responses.

**Resolver presets:**

| Preset | Addresses | Feature |
|--------|-----------|---------|
| ☁️ **Cloudflare** | 1.1.1.1 / 1.0.0.1 | Maximum speed |
| 🛡️ **Quad9** | 9.9.9.9 | Malware/phishing blocking |
| 🔍 **Google** | 8.8.8.8 / 8.8.4.4 | Classic, high reliability |
| 🚫 **AdGuard** | 94.140.14.14 | Ad and tracker blocking |
| 🌐 **All three** | Cloudflare + Quad9 + Google | Round-robin, maximum redundancy |

---

### 🔒 Killswitch

If the VPN connection drops, the killswitch **blocks all unprotected traffic**:

- Clients cannot access the internet without going through the tunnel
- Protects against real IP leaks
- Toggle on/off from the menu in one action
- Does not affect SSH access to the server

---

### 📊 Traffic Limits

- 📏 **Per-client limit** — set in GB (combined inbound + outbound)
- ⚡ **Cumulative accounting** — works correctly across reboots and WireGuard counter resets
- 🚨 **Two enforcement modes**: disconnect the client or log only
- 🔄 **Watcher** — checks every 15 minutes via cron
- 🗑️ **Counter reset** — counters are cleared automatically when a limit is changed or removed

---

### 🔑 Key Rotation

- Rotate encryption keys without losing configuration
- Automatic distribution of new configs to clients
- PSK (Pre-Shared Key) for an additional layer of security

---

### 💾 Backup & Restore

Backup includes:
- `/etc/wireguard/` — all WireGuard configs
- `/etc/nftables.conf` — firewall rules
- `/etc/dnsmasq.d/` — DNS configs
- `/etc/systemd/system/` — service units
- `/etc/telemt.toml` — MTProxy config
- Scripts from `/usr/local/bin/`

The last **10 archives** are kept, older ones deleted automatically. Restore is one action from the menu.

---

## 📋 Requirements

| Parameter | Value |
|-----------|-------|
| **OS** | Ubuntu 24.04+ |
| **Privileges** | root |
| **Bash** | ≥ 4.3 |
| **Architecture** | amd64 |
| **RAM** | ≥ 512 MB |
| **Disk** | ≥ 1 GB free |
| **Network** | Public IP (VPS / dedicated server) |

**Dependencies installed automatically:**  
`wireguard` · `nftables` · `curl` · `iproute2` · `qrencode` · `dnsmasq` · `openssl` · `stubby`

---

## ⚡ Quick Start

### One-command install

```bash
curl -fsSL https://raw.githubusercontent.com/avar-soft/wireguard-home-server/main/wg-server.sh -o wg-server.sh \
  && chmod +x wg-server.sh \
  && sudo bash wg-server.sh
```

On first run the script will:
1. Install dependencies
2. Ask a few questions (interface, port, subnet)
3. Generate WireGuard keys
4. Configure nftables + GeoIP
5. Start all services
6. Open the main menu

**The entire process takes ~2 minutes.**

### Subsequent runs

```bash
sudo bash wg-server.sh
```

---

## 📖 Usage

### Adding a client

```
Main menu → 1) Clients → 1) Add client
```

Enter a device name (e.g. `iphone` or `work-laptop`). The script will:
- Generate a key pair
- Assign an IP from the subnet
- Display a QR code in the terminal
- Save a `.conf` file to `/etc/wireguard/clients/`

Scan the QR code with the WireGuard app (iOS / Android) — done.

---

### Adding a tunnel (external VPS)

```
Main menu → 2) Tunnels → Add tunnel
```

You will need to enter:
- **Endpoint** — `ip:port` or `hostname:port` of the external VPS
- **Public Key** — WireGuard public key on the remote server
- **PSK** (optional) — pre-shared key for stronger encryption
- **MTU** — usually 1420, some providers require 1280

The balancer will automatically start monitoring the new tunnel.

---

### Connecting a client

**iOS / Android:** scan the QR code from the menu → done.

**Linux:**
```bash
scp root@server:/etc/wireguard/clients/mydevice.conf .
sudo wg-quick up ./mydevice.conf
```

**Windows / macOS:** import the `.conf` file into the official WireGuard app.

---

## 🔧 Interactive Menu

```
╔═════════════════════════════════════════════════════════════╗
║      🛡️   WireGuard Home Server  —  v28.24.21               ║
║   RU direct │ Foreign via tunnel │ Split-DNS │ Anti-DPI     ║
╚═════════════════════════════════════════════════════════════╝

  ┌─────────────────────────────────────────────────────────────┐
  │  CLIENTS & ROUTING                                          │
  └─────────────────────────────────────────────────────────────┘
   1)  👤  Clients              — add, QR code, list, revoke
   2)  🔀  Tunnels              — VPN connections to external servers
   3)  🔥  Firewall / GeoIP    — rules, RU IP list, update
   4)  🗺️   Routing profiles    — geo-split / full-vpn / direct-only
   5)  📌  Direct IPs (wlist)  — your subnets bypassing VPN
   6)  🔒  Killswitch          — block traffic on VPN drop
   7)  🌐  DNS (geo/tunnel/pub)— Split-DNS + DNS-over-TLS via stubby
   8)  🛡   Anti-DPI            — traffic obfuscation against DPI

  ┌─────────────────────────────────────────────────────────────┐
  │  MONITORING                                                 │
  └─────────────────────────────────────────────────────────────┘
   9)  📊  WireGuard status    — peers, traffic, last seen
  10)  ⚖️   Balancer            — watchdog, logs, settings
  11)  📈  Monitoring          — client traffic, pings, live monitor
  12)  📊  Traffic limits      — per-client GB limits
  13)  📄  Logs                — all service journals
  14)  ✅  System test         — automated health check

  ┌─────────────────────────────────────────────────────────────┐
  │  OTHER                                                      │
  └─────────────────────────────────────────────────────────────┘
  15)  🦀  Telemt (new MTProxy)      — Rust, Fake TLS, multi-user
  16)  📡  MTProto Proxy (old MTG)   — Go version, for compatibility
  17)  🔑  Key rotation              — rotate encryption keys
  18)  💾  Backup / Restore          — save and restore configs
  19)  🚀  Autostart                 — configure startup on boot
  20)  🔄  Update script             — download new version
  21)  🔄  Restart everything
  22)  💣  Remove everything (IRREVERSIBLE!)
   0)  🚪  Exit
```

---

## 🔬 System Test

Menu item **14) System test** checks the entire chain in one command:

```
── Services ──
  WireGuard server (wg0) active                          ✔ OK
  wg-balance balancer active                             ✔ OK
  nftables rules loaded                                  ✔ OK

── Tunnels ──
  Tunnel wg1 active                                      ✔ OK
  Connectivity to nl.example.com                         ✔ OK

── Routing ──
  fwmark rule for RU traffic                             ✔ OK
  GeoIP RU database (8500+ networks) loaded              ✔ OK
  systemd units valid (verify)                           ✔ OK

── Internet ──
  Ping / connectivity 8.8.8.8                            ✔ OK
  DNS resolution google.com                              ✔ OK

✔ All tests passed (15/15)
```

---

## 🔐 Security

- **`set -uo pipefail`** — immediate exit on unset variables or pipeline errors
- **Input validation** — client names, ports, paths, IP addresses
- **`safe_sed()`** — all config substitutions via an escaping wrapper, no sed injection
- **Named functions** instead of `bash -c` — no string interpolation in commands
- **Atomic operations** — configs rewritten via `mktemp` + `mv`, no partial writes
- **ERR-trap** — all non-zero exit codes logged with context
- **Auto-backup** before destructive operations

---

## 📁 File Structure

```
/etc/wireguard/
├── wg0.conf                    # Main WireGuard interface
├── wg1.conf, wg2.conf...       # Tunnels
├── .wg-setup.conf              # Script configuration
├── .traffic-limits             # Per-client traffic limits
├── clients/
│   ├── phone.conf
│   └── laptop.conf
└── geoip/
    ├── ru-aggregated.zone      # Offline IPv4 database
    └── ru-aggregated-v6.zone   # Offline IPv6 database

/usr/local/bin/
├── wg-balance.sh               # Tunnel balancer
├── update-ru-ipset.sh          # GeoIP updater
├── wg-traffic-limit.sh         # Traffic limit watchdog
└── telemt                      # MTProxy binary

/etc/telemt.toml                # Telemt MTProxy config
/etc/stubby/stubby.yml          # DNS-over-TLS config
/var/lib/wg/limits/             # Traffic counter state files
/var/backups/wg-home/           # Configuration backups
/var/log/wg-server-trap.log     # Script error log
```

---

## ⚙️ Environment Variables

```bash
export WG_VERSION_URL="https://example.com/wg/version.txt"  # Auto-update URL
export EDITOR="nano"          # nano | vim | vi
export WG_DEBUG="1"           # Verbose error logging
export WG_ERR_LOG="/var/log/wg-server-trap.log"
export TELEMT_REPO="telemt/telemt"  # Custom Telemt fork
```

---

## 🔄 Updating

```
Main menu → 20) Update script
```

Or manually — just run the new version. All configs are preserved. Update happens **atomically** via `install` + `mv`.

---

## 🗑️ Removal

```bash
sudo bash wg-server.sh --remove
```

Or via menu **22) Remove everything**. Removes interfaces, nftables rules, systemd services, configs. Packages are left untouched.

---

## 📝 Changelog

<details>
<summary><b>v28.24.21</b> — current</summary>

- 🐛 Clear traffic limit state files on limit removal/change
- 🛡️ `_statusBar()` protected against `set -u` when called outside `menu()`
- 🔧 `restoreBackup`: `trap RETURN` instead of manual tmpdir cleanup
- 🔧 `geo4`/`geo6` pipelines: added `|| true` to guard against `pipefail`

</details>

<details>
<summary><b>v28.24.20</b></summary>

- 🔧 Telemt port change: `sed` → `safe_sed` for consistency
- 🔧 `installTelemt`: check whether alternative port is already in use

</details>

<details>
<summary><b>v28.24.19</b></summary>

- 🛡️ `runSystemTest`: removed `bash -c`, replaced with named `_chk_*` functions
- 🛡️ `menuTelemetConfig`: `sed` → `safe_sed` for `tls_domain`
- 🐛 Validate `UNAME` before `_telemt_removeUser`

</details>

<details>
<summary><b>v28.24.18</b></summary>

- 🐛 Fixed `bash -c` regression in system tests
- 🔧 Updated tunnel balancer

</details>

---

## 🤝 Contributing

Pull requests are welcome. For significant changes, please open an Issue first.

**Guidelines:**
- All bash must pass `set -uo pipefail`
- User input only through `safe_sed()` and explicit validation
- Temporary files via `mktemp` with `trap` cleanup
- Comments required for non-obvious decisions

---

## 📄 License

MIT © [avar-soft](https://github.com/avar-soft)

---

<div align="center">

**Built with ❤️ for those who value privacy and speed**

⭐ If this project is useful — give it a star!

[🔝 Back to top](#readme-top) · [🇷🇺 Русская версия](#русский)

</div>
