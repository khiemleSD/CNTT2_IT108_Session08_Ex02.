# Hướng dẫn cài đặt cảm biến độ ẩm cho trang trại Lan

Cảm biến đo độ ẩm không khí trong nhà lưới. Raspberry Pi đọc số liệu và bật máy bơm phun sương khi độ ẩm xuống thấp. Chữ thường trong tài liệu là phần giải thích. Nội dung trong khối code hoặc dạng `code` là lệnh và tên biến, cần gõ đúng nguyên văn.

## Thiết bị cần chuẩn bị

* \[ ] Cảm biến độ ẩm DHT22 (AM2302)
* \[ ] Raspberry Pi
* \[ ] Thẻ nhớ đã cài hệ điều hành
* \[ ] Dây nối Dupont
* \[ ] Module relay và máy bơm phun sương
* \[ ] Tua vít, băng keo, ốc vít

## So sánh hai loại cảm biến

Độ ẩm lý tưởng cho Lan là 60–80%. DHT11 chỉ đo đến 80%, nên không thấy được khi nhà lưới quá ẩm. DHT22 đo đủ dải và sai số nhỏ hơn.

|Tiêu chí|DHT11 (loại cũ)|DHT22 (loại mới)|
|-|:-:|:-:|
|Dải đo độ ẩm|20–80%|0–100%|
|Sai số độ ẩm|±5%|±2%|
|Dải đo nhiệt độ|0 đến 50 °C|−40 đến 80 °C|
|Sai số nhiệt độ|±2 °C|±0,5 °C|
|Tần suất đọc|1 lần mỗi giây|1 lần mỗi 2 giây|

## Nối dây

Chân thứ ba của DHT22 là NC, để trống.

|Chân DHT22|Chân Raspberry Pi (BCM)|
|-|-|
|VCC|3.3V (Pin 1)|
|GND|GND (Pin 6)|
|DATA|GPIO4 (Pin 7)|

Chân IN của relay nối vào GPIO17 (Pin 11).

## Cài đặt

Mở Terminal, chạy `sudo apt update` để làm mới danh sách gói, rồi cài thư viện đọc cảm biến:

```bash
sudo apt update \\\&\\\& sudo apt upgrade -y
sudo apt install -y git python3-pip python3-rpi.gpio
sudo pip3 install Adafruit\\\_DHT
```

## Code Python điều khiển máy bơm

Đổi `HUMIDITY\\\_MIN` để chỉnh ngưỡng bật bơm, đổi `PUMP\\\_SECONDS` để chỉnh thời gian phun.

```python
import time
import Adafruit\\\_DHT
import RPi.GPIO as GPIO

SENSOR = Adafruit\\\_DHT.DHT22
DHT\\\_PIN = 4           # GPIO4, chân DATA của cảm biến
PUMP\\\_PIN = 17         # GPIO17, nối vào chân IN của relay
HUMIDITY\\\_MIN = 60     # dưới 60% thì bật bơm
PUMP\\\_SECONDS = 10

humidity, temp = Adafruit\\\_DHT.read\\\_retry(SENSOR, DHT\\\_PIN)

if humidity is None:
    print("Không đọc được cảm biến")
else:
    print(f"Temp={temp:.1f}C Humidity={humidity:.1f}%")
    if humidity < HUMIDITY\\\_MIN:
        GPIO.setmode(GPIO.BCM)
        GPIO.setup(PUMP\\\_PIN, GPIO.OUT)
        GPIO.output(PUMP\\\_PIN, GPIO.HIGH)   # relay kích mức cao
        time.sleep(PUMP\\\_SECONDS)
        GPIO.output(PUMP\\\_PIN, GPIO.LOW)
        GPIO.cleanup()
```

Lưu file thành `pump\\\_control.py`, chạy thử bằng `python3 pump\\\_control.py`. Kết quả mẫu:

```text
Temp=28.4C Humidity=67.1%
```

## Tự động chạy mỗi 5 phút

Gõ `crontab -e`, thêm dòng sau vào cuối file:

```bash
\\\*/5 \\\* \\\* \\\* \\\* python3 /home/pi/pump\\\_control.py
```

## Ký tự đặc biệt và ký tự escape

Ở chữ thường, Markdown hiểu `\\\*` là in nghiêng và `#` là tiêu đề. Muốn hiển thị nguyên dạng thì đặt dấu `\\\\` phía trước. Dòng cron có năm trường, viết là \*/5 \* \* \* \*, nghĩa là mỗi 5 phút.

\# Dòng này bắt đầu bằng dấu thăng nhưng vẫn là chữ thường.

Trong khối code thì không cần escape, vì mọi ký tự đã hiển thị nguyên dạng.

> Độ ẩm lý tưởng cho Lan là 60–80%. Thấp hơn mức này thì cần phun sương nhiều hơn.



# Source-code
