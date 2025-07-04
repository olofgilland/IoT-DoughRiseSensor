# Dough Rise Detector – IR Sensor with Raspberry Pi Pico WH

**Author**: Olof Gilland / og222gi

In this project, I created an IoT dough-rise detection system using a Raspberry Pi Pico WH and an IR break beam sensor. The system visually indicates the dough’s rise using LEDs and sends an email notification via AWS when the dough has risen. I came up with this idea because I often bake pizza and found it tedious to constantly monitor when the dough had doubled in size.

The project is estimated to take **3–7 hours**, depending on experience.

---

## Objective

I wanted to automate the process of tracking dough rise during pizza baking. By placing dough in a clear plastic rain gauge and using a non-contact IR sensor, I can now detect when the dough has expanded and broken the beam. The system:

- Lights a red LED when the dough has risen, green when not
- Sends an email alert via AWS SES + Lambda
- Enables hands-off baking while improving rise timing accuracy
- (Future) Track impact of temperature/humidity using additional sensors.

---

## Materials

| Component                        | Description                                                            | Price (SEK) |
|----------------------------------|------------------------------------------------------------------------|-------------|
| Raspberry Pi Pico WH             | Wi-Fi microcontroller                                                  | 349         |
| IR Break Beam Sensor (5mm)       | Sensor pair to detect beam break (880nm IR)                            | 99          |
| Breadboard + Jumper Wires        | Prototyping and connection cables                                      | Included    |
| 330Ω Resistors                   | For current-limiting on LEDs                                           | Included    |
| Red LED                          | Lights when dough has risen                                            | Included    |
| Green LED                        | Lights when dough has not yet risen                                    | Included    |
| USB-A to micro USB cable         | Power and data connection                                              | Included    |
| Rain gauge (plastic)             | To contain dough and allow IR beam through side                        | 30          |

All parts were sourced from Electrokit.se except for rain gauge which was bought at Jula.

---

## Computer Setup

- **IDE**: Thonny  
- **Firmware**:  MicroPython for Pico W
- **Tools Installed**:
  - Thonny (from thonny.org)
- **Workflow**:
 	1.	Flash MicroPython firmware to the Pico WH via drag-and-drop while holding the BOOTSEL button.
	2.	Open Thonny and select “MicroPython (Raspberry Pi Pico)” as the interpreter.
	3.	Write and run code directly in Thonny, save as main.py on the Pico.
	4.	Use the Shell in Thonny to view print statements and debug.

---

## Putting Everything Together

A simplified description of the wiring:

- IR Transmitter:  
  - VCC to 3.3V rail  
  - GND to GND rail

- IR Receiver:  
  - OUT to GP19  
  - 10kΩ resistor between OUT and GND (pull-down)

- Green LED:  
  - Anode to GP14 via 330Ω resistor  
  - Cathode to GND rail

- Red LED:  
  - Anode to GP15 via 330Ω resistor  
  - Cathode to GND rail

**Voltage**: 3.3V  
**Resistor Calc**: LEDs use ~20mA; 330Ω limits current to ~10mA which is safe for GPIO.

A full circuit diagram was sketched by hand (image to be included in report or appendix).
<img width="678" alt="bild" src="https://github.com/user-attachments/assets/791a9ff0-938f-4b28-97d8-34b65febc01d" />


---

## Platform

The system includes cloud connectivity. When the dough rises and blocks the beam:

- An HTTP request is sent from the Pico WH to an AWS API Gateway endpoint.
- A Lambda function is triggered, checking a cooldown and then sending an email via AWS SES.
- A DynamoDB table stores the last sent timestamp to prevent spamming.

Device: Raspberry Pi Pico WH (with built-in Wi-Fi)
Cloud Services: AWS Lambda, SES, DynamoDB, API Gateway
Data Transmission: HTTP POST from device to API

---

## The Code

```python
from machine import Pin
import network
import urequests
import time

# Wi-Fi credentials
SSID = "WIFI_NAME"
PASSWORD = "PASSWORD"

# AWS API Gateway endpoint
EMAIL_API_URL = "YOUR_EMAIL_API_URL"

# Sensor and LED setup
ir_sensor = Pin(14, Pin.IN, Pin.PULL_UP)
led_clear = Pin(15, Pin.OUT)     # Green LED
led_blocked = Pin(16, Pin.OUT)   # Red LED

# Track previous state to avoid sending duplicate requests
last_beam_blocked = False

# Connect to Wi-Fi
def connect_wifi():
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    wlan.connect(SSID, PASSWORD)

    print("Connecting to Wi-Fi...")
    while not wlan.isconnected():
        time.sleep(1)

    print("Connected:", wlan.ifconfig())

def send_email():
    try:
        print("Sending request to AWS...")
        response = urequests.post(EMAIL_API_URL)
        print("Response:", response.status_code, response.text)
        response.close()
    except Exception as e:
        print("Error sending request:", e)

# Main program
connect_wifi()
print("Starting dough sensor...")

while True:
    sensor_value = ir_sensor.value()

    if sensor_value == 1:
        # Beam is unbroken – dough not risen
        led_clear.on()
        led_blocked.off()
        last_beam_blocked = False
        print("Beam FREE – Dough not risen")
    else:
        # Beam blocked – dough has risen
        led_clear.off()
        led_blocked.on()
        print("Beam BLOCKED – Dough has risen!")

        if not last_beam_blocked:
            send_email()
            last_beam_blocked = True

    time.sleep(1)


```

Uploaded via Thonny to the Raspberry Pi Pico WH.

---

## Transmitting Data

- Protocol: Wi-Fi
- Data sent via: HTTP POST to AWS API Gateway
- Triggered Function: AWS Lambda (Python)
- Email provider: AWS SES
- Throttling: Email is only sent if >60 seconds have passed since last one (stored in DynamoDB)
  
Design consideration:
- Reliable and low-power IR sensing
- Minimal data transmission (only one request per dough rise event)
- Avoid email spamming with cooldown logic
---

## Final Design

The system consists of:

- Rain gauge to hold dough  
- IR sensor on the side  
- Breadboard with Pico WH and LEDs  
- USB for power and debugging  
- Fully working indicator LEDs

**Lessons learned**:
- IR beam sensors work through plastic!
- Need controlled lighting to avoid interference
- Could be made portable with battery and case

---

![project_photo](placeholder)

---

*Created on 2025-07-04*
