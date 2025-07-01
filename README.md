# Dough Rise Detector – IR Sensor with Raspberry Pi Pico WH

**Author**: Olof Gilland / og222ig

In this project, I created an IoT dough-rise detection system using a Raspberry Pi Pico WH and an IR break beam sensor. The system uses LEDs to visually indicate dough rising and has been tested for future expansion with AWS email alerts. I came up with this idea because I often bake pizza and realized that it was tedious to monitor the dough constantly while waiting for it to rise to twice it's size.

The project is estimated to take **3–7 hours**, depending on experience.

---

## Objective

I wanted to automate the process of tracking dough rise while baking pizza. By placing dough in a clear plastic rain gauge, I could use a non-contact IR beam sensor to detect when the dough had doubled in size and blocked the beam. This lets me:

- Avoid constant visual checking.
- Collect time-based insights for improving rise predictions.
- (Future) Track impact of temperature/humidity using additional sensors.
- (Future) Send notifications to email or smartphone.

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

Currently running fully **offline** on a Raspberry Pi Pico WH with no cloud or server. The only planned extension is to use **AWS SES** + **Lambda** to send an email when dough has risen.  

Platform choice:
- **Local** to reduce complexity.
- **Pico WH** chosen for built-in Wi-Fi, affordable price, and strong MicroPython support.

For scaling:
- Add cloud services (AWS, Azure IoT)
- Add temperature/humidity sensors
- Use InfluxDB + Grafana for tracking rise data over time

---

## The Code

```python
from machine import Pin
import time

sensor = Pin(19, Pin.IN, Pin.PULL_UP)
led_green = Pin(14, Pin.OUT)
led_red = Pin(15, Pin.OUT)

while True:
    if sensor.value() == 1:
        print("Beam FREE – Dough not risen")
        led_green.on()
        led_red.off()
    else:
        print("Beam BLOCKED – Dough has risen!")
        led_green.off()
        led_red.on()
    time.sleep(0.5)
```

Uploaded via Thonny to the Raspberry Pi Pico WH.

---

## Transmitting Data (Future)

Currently no data is transmitted.

**Planned**:
- Wi-Fi connection with AWS SES + Lambda
- Email sent only when dough has risen (red LED on)
- Throttled: only 1 email every 1–2 minutes using DynamoDB timestamp check

**Protocols**:
- Wi-Fi for network
- AWS SDK (Boto3 in Lambda) for sending email
- No MQTT or webhook in this version

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

*Created on 2025-07-01*
