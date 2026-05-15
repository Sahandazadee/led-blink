# ESP8266 Dual LED Alternating Blink

[![Platform](https://img.shields.io/badge/platform-ESP8266-blue)](https://www.espressif.com/en/products/socs/esp8266)
[![Framework](https://img.shields.io/badge/framework-Arduino-teal)](https://www.arduino.cc/)
[![Language](https://img.shields.io/badge/language-C++-orange)](https://isocpp.org/)

پروژه آموزشی چشمک‌زن دو LED روی میکروکنترلر ESP8266 — دو LED به صورت متناوب روشن و خاموش می‌شوند.

---

## فهرست مطالب

- [توضیح پروژه](#توضیح-پروژه)
- [لیست قطعات](#لیست-قطعات)
- [سیم‌کشی و نقشه مدار](#سیم‌کشی-و-نقشه-مدار)
- [پیش‌نیازها و نصب](#پیش‌نیازها-و-نصب)
- [مراحل اجرا](#مراحل-اجرا)
- [رفع خطا](#رفع-خطا)
- [پیشنهادات بهبود](#پیشنهادات-بهبود)
- [لیست قطعات کامل (BOM)](BOM.md)
- [توضیح کد](CODE_EXPLANATION.md)

---

## توضیح پروژه

این پروژه پایه‌ای‌ترین قدم برای شروع کار با میکروکنترلر ESP8266 است. هدف این است که:

- با محیط PlatformIO آشنا شوید
- نحوه کامپایل و آپلود firmware را یاد بگیرید
- با پین‌های GPIO کار کنید
- زنجیره توسعه VM → Raspberry Pi → ESP8266 را تجربه کنید

**عملکرد:** دو LED با فاصله یک ثانیه به صورت متناوب روشن و خاموش می‌شوند — وقتی LED1 روشن است، LED2 خاموش است و برعکس.

---

## لیست قطعات

- میکروکنترلر: ESP8266MOD روی برد NodeMCU v2
- برد واسط: Raspberry Pi 4B
- LED (هر رنگی): ۲ عدد
- مقاومت ۲۲۰ اهم: ۲ عدد
- کابل Micro-USB: ۱ عدد (اتصال Pi به ESP8266)
- سیم جامپر نر-نر: ۴ عدد

> **چرا مقاومت ۲۲۰ اهم؟**
> پین‌های ESP8266 ولتاژ ۳.۳ ولت دارند. جریان ایمن برای LED حدود ۱۵ میلی‌آمپر است:
> `R = V / I = 3.3V / 0.015A ≈ 220Ω`
> بدون مقاومت، جریان بیش از حد LED را می‌سوزاند.

---

## سیم‌کشی و نقشه مدار

### جدول پین‌ها

- پین `D0` (GPIO16) → مقاومت 220Ω → آند LED1 → کاتد به GND
- پین `D1` (GPIO5)  → مقاومت 220Ω → آند LED2 → کاتد به GND
- پین `GND` → GND مشترک هر دو LED
- Raspberry Pi → کابل Micro-USB → پورت USB برد NodeMCU

### نقشه مدار

```
                 ┌─────────────────────────────────┐
                 │        ESP8266 NodeMCU v2        │
                 │                                 │
   Raspberry Pi  │                                 │
   USB ──────────┤ Micro-USB                       │
                 │                                 │
                 │             D1 (GPIO5)  ─────────┼──[220Ω]──┬──►|──┐
                 │                                 │          │ LED2  │
                 │             D0 (GPIO16) ─────────┼──[220Ω]──┼──►|──┤
                 │                                 │          │ LED1  │
                 │                          GND ───┼──────────┴───────┘
                 └─────────────────────────────────┘

   ►| = LED  (آند به سمت راست، کاتد به سمت چپ)
```

### نحوه اتصال قدم به قدم

1. پایه بلندتر LED1 (آند) را از طریق مقاومت ۲۲۰Ω به پین `D0` وصل کنید
2. پایه کوتاه‌تر LED1 (کاتد) را به `GND` وصل کنید
3. همین کار را برای LED2 با پین `D1` تکرار کنید
4. ESP8266 را از طریق Micro-USB به Raspberry Pi وصل کنید

---

## پیش‌نیازها و نصب

### روی کامپیوتر توسعه (VM یا لپ‌تاپ)

```bash
pip3 install platformio
```

### روی Raspberry Pi

```bash
# نصب esptool برای فلش کردن ESP8266
pip3 install esptool

# دسترسی به پورت سریال بدون نیاز به sudo
sudo usermod -aG dialout $USER
```

> پس از دستور `usermod` باید log out و دوباره log in کنید تا تغییرات اعمال شود.

---

## مراحل اجرا

### گام ۱ — کلون کردن ریپو

```bash
git clone https://github.com/Sahandazadee/led-blink.git
cd led-blink
```

### گام ۲ — کامپایل

```bash
pio run
```

PlatformIO کد را با toolchain مخصوص ESP8266 کامپایل می‌کند و فایل `firmware.bin` را در مسیر `.pio/build/esp8266mod/` می‌سازد.

خروجی موفق:
```
RAM:   [===       ]  34.3% (used 28104 bytes from 81920 bytes)
Flash: [===       ]  25.0% (used 261511 bytes from 1044464 bytes)
========================= [SUCCESS] =========================
```

### گام ۳ — انتقال firmware به Raspberry Pi

```bash
scp .pio/build/esp8266mod/firmware.bin sahand@192.168.0.148:/tmp/firmware.bin
```

`scp` فایل باینری را از طریق SSH به Pi منتقل می‌کند.

### گام ۴ — فلش کردن روی ESP8266

روی Raspberry Pi یا از طریق SSH:

```bash
python3 -m esptool --port /dev/ttyACM0 --baud 115200 write-flash 0x00000 /tmp/firmware.bin
```

- `--port /dev/ttyACM0` — پورت USB که ESP8266 به آن وصل است
- `--baud 115200` — سرعت ارتباط سریال
- `write-flash 0x00000` — نوشتن از ابتدای حافظه flash
- `/tmp/firmware.bin` — فایل باینری

خروجی موفق:
```
Detecting chip type... ESP8266
Wrote 265664 bytes at 0x00000000 in 17.9 seconds
Hash of data verified.
Hard resetting via RTS pin...
```

پس از اتمام، ESP8266 به صورت خودکار reset می‌شود و LEDها شروع به چشمک زدن می‌کنند.

---

## رفع خطا

### خطا: board ID ناشناخته

```
Error: Unknown board ID 'esp8266mod'
```

**علت:** PlatformIO این نام را نمی‌شناسد.

**راه‌حل:** در `platformio.ini` مقدار `board` را به `nodemcuv2` تغییر دهید:

```bash
pio boards espressif8266   # لیست board های معتبر
```

---

### خطا: پورت سریال پیدا نشد

```
Could not open /dev/ttyUSB0: No such file or directory
```

**علت:** پورت اشتباه است. بسته به تراشه USB-to-Serial روی برد، پورت می‌تواند `ttyUSB0` یا `ttyACM0` باشد.

**راه‌حل:** پورت واقعی را پیدا کنید:

```bash
ls /dev/tty* | grep -E 'USB|ACM'
```

NodeMCU با تراشه CH340 معمولاً روی `ttyACM0` می‌آید.

---

### خطا: دسترسی رد شد

```
Permission denied: '/dev/ttyACM0'
```

**علت:** کاربر عضو گروه `dialout` نیست.

**راه‌حل:**

```bash
sudo usermod -aG dialout $USER
# سپس log out و log in کنید
```

---

### خطا: هویت git ناشناخته

```
Author identity unknown — fatal: unable to auto-detect email address
```

**راه‌حل:**

```bash
git config --global user.email "your@email.com"
git config --global user.name "YourName"
```

---

### LED روشن نمی‌شود

- بررسی کنید آند و کاتد LED جابجا نباشند (پایه بلندتر = آند)
- بررسی کنید مقاومت سر جای خود است
- با مولتی‌متر ولتاژ پین D0 و D1 را هنگام اجرا اندازه بگیرید (باید بین ۰ و ۳.۳ ولت تغییر کند)
- اطمینان حاصل کنید firmware با موفقیت فلش شده است

---

## پیشنهادات بهبود

- **کنترل سرعت با پتانسیومتر:** اضافه کردن پتانسیومتر به پین A0 برای تغییر سرعت چشمک‌زنی بدون کامپایل مجدد
- **کنترل از طریق Serial:** دریافت دستور از Serial Monitor برای روشن/خاموش کردن LEDها
- **Fade با PWM:** به جای on/off ناگهانی، از `analogWrite` برای fade-in/fade-out استفاده کنید
- **WiFi Control:** با استفاده از WiFi داخلی ESP8266، یک وب‌سرور ساده بسازید و LEDها را از مرورگر کنترل کنید
- **MQTT:** اتصال به یک MQTT broker برای کنترل از راه دور با پروتکل استاندارد IoT
- **گسترش به ۸ LED:** استفاده از شیفت رجیستر 74HC595 برای کنترل ۸ LED با تنها ۳ پین

---

## توضیح کد

برای توضیح کامل و خط به خط کد، فایل [CODE_EXPLANATION.md](CODE_EXPLANATION.md) را مطالعه کنید.
