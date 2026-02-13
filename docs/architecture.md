# Архітектура системи — TeslaBox підхід

## Огляд

Raspberry Pi 4B емулює USB Mass Storage пристрій для Tesla.
Tesla записує Dashcam/Sentry відео як на звичайну флешку.
RPi моніторить нові файли, обробляє їх і відправляє в Telegram через 4G.

## USB Gadget Mode

Raspberry Pi 4B підтримує USB OTG (On-The-Go), що дозволяє йому
працювати як USB-пристрій (gadget). Модуль `g_mass_storage` створює
віртуальний USB-диск, який Tesla бачить як звичайну флешку.

### Налаштування
```
# /boot/config.txt
dtoverlay=dwc2

# /etc/modules
dwc2
g_mass_storage

# Створення образу диска
dd if=/dev/zero of=/piusb.bin bs=512 count=131072000  # ~64GB
mkdosfs /piusb.bin -F 32 -I
```

## Структура файлів Tesla на USB

```
/TeslaCam/
├── RecentClips/           # Останні 60 хв (циклічний)
│   ├── 2026-02-13_12-00-front.mp4
│   ├── 2026-02-13_12-00-left_repeater.mp4
│   ├── 2026-02-13_12-00-right_repeater.mp4
│   └── 2026-02-13_12-00-back.mp4
├── SavedClips/            # Збережені (кнопка/сигнал)
│   └── 2026-02-13_12-05/
│       ├── 2026-02-13_12-05-front.mp4
│       └── ...
└── SentryClips/           # Sentry Mode події
    └── 2026-02-13_14-30/
        ├── 2026-02-13_14-30-front.mp4
        └── ...
```

## Потік даних

```
1. Tesla → пише .mp4 файли на USB (g_mass_storage)
2. RPi → inotifywait моніторить SavedClips/ та SentryClips/
3. Новий кліп → ffmpeg стискає + генерує GIF
4. Telegram Bot API → відправляє GIF + відео + локацію
5. Бекап → зберігає на microSD або NAS (через WiFi вдома)
```

## Програмні компоненти

### 1. USB Mass Storage daemon
- Монтує /piusb.bin як USB-диск для Tesla
- Одночасно монтує той самий образ для читання на RPi

### 2. File watcher (inotify)
- Моніторить SentryClips/ та SavedClips/
- Тригерить обробку при появі нових файлів

### 3. Video processor (ffmpeg)
- Стискає відео для відправки через 4G
- Генерує GIF прев'ю (10 сек, low quality)
- Об'єднує кути камер в мозаїку (опціонально)

### 4. Telegram Bot
- Відправляє GIF → потім повне відео
- Inline кнопки: вибір камери, стрім, статус
- Команди: /status, /live, /clips, /settings

### 5. 4G Manager
- NetworkManager або ModemManager для USB dongle
- Автопідключення при старті
- Fallback на WiFi вдома

### 6. Tailscale VPN (опціонально)
- Прямий доступ до RPi з будь-якої точки
- Веб-інтерфейс для стріму та архіву
