# Інструкція з встановлення

## Крок 1: Підготовка microSD

### 1.1 Завантажити Raspberry Pi OS Lite (64-bit)
- https://www.raspberrypi.com/software/
- Використати Raspberry Pi Imager
- Обрати: Raspberry Pi OS Lite (64-bit, Bookworm)

### 1.2 Налаштування перед першим запуском
В Raspberry Pi Imager → Settings:
- Hostname: `teslabox`
- SSH: увімкнути
- WiFi: ввести домашню мережу (для першого налаштування)
- Username/Password: встановити свої

## Крок 2: Перше налаштування (вдома через WiFi)

```bash
# Підключитись по SSH
ssh pi@teslabox.local

# Оновити систему
sudo apt update && sudo apt upgrade -y

# Встановити необхідне
sudo apt install -y ffmpeg python3-pip git
```

## Крок 3: Налаштування USB Gadget Mode

```bash
# Додати в /boot/firmware/config.txt
echo "dtoverlay=dwc2" | sudo tee -a /boot/firmware/config.txt

# Створити образ USB-диска (64GB)
sudo dd if=/dev/zero of=/piusb.bin bs=1M count=65536
sudo mkdosfs /piusb.bin -F 32 -I

# Створити точку монтування
sudo mkdir /mnt/usb
sudo mount -o loop /piusb.bin /mnt/usb
sudo mkdir /mnt/usb/TeslaCam
sudo umount /mnt/usb
```

## Крок 4: Встановлення TeslaBox

### Варіант A: TeslaBox (з Telegram з коробки)
```bash
curl -sSL https://www.teslarpi.com/install.sh | sudo bash
```
Далі — зареєструватись на teslarpi.com та отримати ліцензію.

### Варіант B: TeslaUSB + свій Telegram бот (безкоштовно)
```bash
# Завантажити teslausb
# Або використати готовий образ:
# https://github.com/marcone/teslausb/releases

# Для свого Telegram бота — встановити додатково:
pip3 install python-telegram-bot
```

## Крок 5: Створити Telegram бота

1. Відкрити @BotFather в Telegram
2. `/newbot` → назва: `Tesla Dashcam Bot`
3. Зберегти токен
4. Отримати chat_id (відправити повідомлення боту, потім:
   `curl https://api.telegram.org/bot<TOKEN>/getUpdates`)

## Крок 6: Налаштування 4G модему

```bash
# Huawei E3372 зазвичай визначається як ethernet
# Перевірити:
lsusb
ip addr

# Якщо потрібен ModemManager:
sudo apt install -y modemmanager
sudo mmcli -L
sudo mmcli -m 0 --simple-connect="apn=internet"
```

## Крок 7: Встановлення в Tesla

1. Вимкнути запис: затиснути іконку камери на екрані (червона → сіра)
2. Почекати 10 секунд
3. Підключити RPi кабелем USB-A → USB-C в бардачку
4. Почекати 30-60 сек (RPi завантажується)
5. Іконка камери повинна стати червоною (запис активний)
6. Перевірити Telegram — надіслати команду `/status`

## Крок 8: Тестування

1. **Dashcam**: натисни іконку камери або посигнал
   → через 1-3 хв отримаєш відео в Telegram
2. **Sentry Mode**: увімкни Sentry, підійди до машини
   → автоматичне сповіщення в Telegram
3. **Стрім**: `/live` в Telegram або http://teslabox.local
