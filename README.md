# Dough Rise Detector – IR Sensor with Raspberry Pi Pico WH

**Author**: Olof Gilland / og222ig

In this project, I created an IoT dough-rise detection system using a Raspberry Pi Pico WH and an IR break beam sensor. The project visually indicates the dough's rise using LEDs and can also be extended to send email alerts via AWS when the dough is ready. The green LED shows when the beam is unbroken, and a red LED lights up when the dough interrupts the beam—indicating it has risen.

This project is a great introduction to hardware prototyping with sensors and microcontrollers, and can be completed in a few hours depending on experience.

---

## Objective

I wanted to create a simple, automated way to detect when dough has risen during baking (in my case pizza). Traditional methods require manual observation, so this project uses an IR break beam sensor to track when the dough expands and breaks the beam. The idea is to notify the user either visually (using LEDs) or remotely (via email) when the rise is complete. This is done by placing dough in a rain gauge which easily lets us see when the dough has been doubled.

---

## Material

| Component                        | Purpose                                  |
|----------------------------------|------------------------------------------|
| Raspberry Pi Pico WH             | Microcontroller with Wi-Fi capability    |
| IR Break Beam Sensor (5mm)       | Detect dough rise (beam interruption)    |
| Breadboard + Jumper Wires        | Connect components                       |
| 330Ω Resistors                   | Current limiting for LEDs                |
| Red LED                          | Indicates dough has risen (beam blocked) |
| Green LED                        | Indicates beam is unbroken               |
| USB-A to micro USB cable         | Power and programming                    |
| Rain gauge                       | Accurate measurement of dough rise       |

---

## Raspberry Pi Pico WH

The **Raspberry Pi Pico WH** is a Wi-Fi-enabled microcontroller used to control the IR sensor and LEDs, and potentially to send email notifications via AWS.

---

## IR Break Beam Sensor

This sensor includes:
- **IR Transmitter (LED)**
- **IR Receiver (Phototransistor)**

When the dough blocks the IR beam, the sensor signals the Pico, which turns on the red LED.

---

## LED Indicators

- **Green LED**: Beam is unbroken, dough hasn't risen past the beam
- **Red LED**: Beam is blocked, dough has risen

Each LED uses a 330Ω resistor to limit current.

---

## Wiring Setup

Here’s how the system is wired:

- IR LED (transmitter):  
  - VCC → Red rail (3.3V)  
  - GND → Blue rail (GND)  

- IR Receiver (phototransistor):  
  - VCC → 3.3V  
  - OUT → GP19 (can vary)  
  - 10kΩ Resistor: OUT → GND (pull-down)

- **Green LED**:  
  - Anode → GP14 via 220Ω resistor  
  - Cathode → GND  

- **Red LED**:  
  - Anode → GP15 via 220Ω resistor  
  - Cathode → GND  

Use a breadboard for connections and jumper wires to make links between the Pico and the components.

---

## Code (Python / MicroPython)

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

---

## Extensions

This project can be expanded to:
- **Send an email notification** using AWS SES and Lambda when the beam is blocked.
- **Monitor temperature/humidity** using a DHT11 or DHT22 sensor.
- **Show dough status** on an OLED screen.

---

## Final Result

The setup successfully detects the rise of the dough using the IR sensor and indicates the result using two LEDs. With Wi-Fi built into the Pico WH, it's ready for future upgrades like sending cloud-based alerts or connecting to an IoT dashboard.

---

*Created on 2025-07-01*
