# ESP8266 Dual LED Alternating Blink

دو LED به صورت متناوب با فاصله یک ثانیه چشمک می‌زنند — یکی روشن، دیگری خاموش، بعد برعکس.

---

## سخت‌افزار مورد نیاز

| قطعه | مشخصات |
|------|---------|
| میکروکنترلر | ESP8266MOD روی برد NodeMCU v2 |
| برد واسط | Raspberry Pi (هر نسخه‌ای) |
| LED | ۲ عدد |
| مقاومت | ۲ عدد ۲۲۰ اهم (برای محافظت LED) |

### اتصال پین‌ها

```
ESP8266 D0 (GPIO16) ──[ 220Ω ]──[ LED1 ]── GND
ESP8266 D1 (GPIO5)  ──[ 220Ω ]──[ LED2 ]── GND
```

> **نکته:** پین‌های D0 و D1 روی برد NodeMCU برچسب‌گذاری شده‌اند. D0 در واقع GPIO16 و D1 در واقع GPIO5 است.

---

## پیش‌نیازها

### روی Raspberry Pi

```bash
# نصب esptool برای فلش کردن ESP8266
pip3 install esptool

# اطمینان از دسترسی به پورت سریال (بدون sudo)
sudo usermod -aG dialout $USER
# بعد از این دستور، باید log out و دوباره log in کنی
```

### روی کامپیوتر توسعه

```bash
# نصب PlatformIO CLI
pip3 install platformio
```

---

## ساختار پروژه

```
led-blink/
├── platformio.ini   # تنظیمات PlatformIO: برد، پورت، فریم‌ورک
├── src/
│   └── main.cpp     # کد اصلی برنامه
└── .gitignore       # حذف پوشه .pio/ از git (فایل‌های موقت کامپایل)
```

---

## کد برنامه

```cpp
#include <Arduino.h>

#define LED1 D0   // GPIO16
#define LED2 D1   // GPIO5

void setup() {
  pinMode(LED1, OUTPUT);
  pinMode(LED2, OUTPUT);
}

void loop() {
  digitalWrite(LED1, HIGH);  // LED1 روشن
  digitalWrite(LED2, LOW);   // LED2 خاموش
  delay(1000);               // یک ثانیه صبر

  digitalWrite(LED1, LOW);   // LED1 خاموش
  digitalWrite(LED2, HIGH);  // LED2 روشن
  delay(1000);               // یک ثانیه صبر
}
```

---

## مراحل اجرا

### ۱. کامپایل کردن

```bash
pio run
```

**این دستور چه می‌کند؟**
PlatformIO کد C++ را با toolchain مخصوص ESP8266 (xtensa-lx106) کامپایل می‌کند و فایل `firmware.bin` را در پوشه `.pio/build/esp8266mod/` می‌سازد.

خروجی موفق:
```
RAM:   [===       ]  34.3% (used 28104 bytes from 81920 bytes)
Flash: [===       ]  25.0% (used 261511 bytes from 1044464 bytes)
========================= [SUCCESS] =========================
```

---

### ۲. انتقال firmware به Raspberry Pi

```bash
scp .pio/build/esp8266mod/firmware.bin sahand@192.168.0.148:/tmp/firmware.bin
```

**این دستور چه می‌کند؟**
`scp` (Secure Copy) فایل را از کامپیوتر ما از طریق SSH به Raspberry Pi کپی می‌کند. `/tmp/` یک پوشه موقت روی Pi است.

---

### ۳. فلش کردن روی ESP8266

روی Raspberry Pi (یا از طریق SSH):

```bash
python3 -m esptool --port /dev/ttyACM0 --baud 115200 write-flash 0x00000 /tmp/firmware.bin
```

**هر بخش این دستور چه معنایی دارد؟**

| بخش | توضیح |
|-----|-------|
| `python3 -m esptool` | اجرای esptool به عنوان ماژول Python |
| `--port /dev/ttyACM0` | پورت USB که ESP8266 به آن وصل است |
| `--baud 115200` | سرعت ارتباط سریال (bits per second) |
| `write-flash` | دستور نوشتن روی حافظه flash |
| `0x00000` | آدرس شروع نوشتن — ابتدای flash |
| `/tmp/firmware.bin` | فایل باینری که باید نوشته شود |

خروجی موفق:
```
Detecting chip type... ESP8266
Wrote 265664 bytes (195753 compressed) at 0x00000000 in 17.9 seconds
Hash of data verified.
Hard resetting via RTS pin...
```

---

## خطاهایی که ممکن است پیش بیاید

### خطا ۱: board ID ناشناخته
```
Error: Unknown board ID 'esp8266mod'
```
**علت:** PlatformIO این نام را نمی‌شناسد.
**راه‌حل:** در `platformio.ini` بنویس `board = nodemcuv2`

برای دیدن لیست board های معتبر:
```bash
pio boards espressif8266
```

---

### خطا ۲: پورت سریال پیدا نشد
```
Could not open /dev/ttyUSB0: No such file or directory
```
**علت:** پورت اشتباه است. بسته به نوع تراشه USB-to-Serial روی برد، پورت می‌تواند `ttyUSB0` یا `ttyACM0` باشد.
**راه‌حل:** پورت واقعی را پیدا کن:
```bash
ls /dev/tty* | grep -E 'USB|ACM'
```
NodeMCU معمولاً روی `ttyACM0` می‌آید.

---

### خطا ۳: دسترسی به پورت رد شد
```
Permission denied: '/dev/ttyACM0'
```
**علت:** کاربر عضو گروه `dialout` نیست.
**راه‌حل:**
```bash
sudo usermod -aG dialout $USER
# سپس log out و log in کن
```

---

### خطا ۴: هویت git ناشناخته
```
Author identity unknown — fatal: unable to auto-detect email address
```
**علت:** git هیچ‌وقت روی این سیستم تنظیم نشده.
**راه‌حل:**
```bash
git config --global user.email "your@email.com"
git config --global user.name "YourName"
```

---

## تنظیمات platformio.ini

```ini
[env:esp8266mod]
platform = espressif8266
board = nodemcuv2          ; برد NodeMCU v2 با ماژول ESP-12E
framework = arduino        ; استفاده از فریم‌ورک Arduino
upload_port = /dev/ttyACM0 ; پورت سریال روی Raspberry Pi
monitor_port = /dev/ttyACM0
monitor_speed = 115200     ; سرعت Serial Monitor
```
