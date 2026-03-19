
# Ultrasonic Distance Sensor with Raspberry Pi Pico

> **HC-SR04 distance measurement — TRIG pulse fires the sensor, ECHO pulse width measured in µs, converted to cm and inches.**

[![Language](https://img.shields.io/badge/Language-C%20(C11)-blue?style=flat-square&logo=c)](https://en.cppreference.com/w/c/11)
[![SDK](https://img.shields.io/badge/SDK-Pico%20SDK-brightgreen?style=flat-square)](https://github.com/raspberrypi/pico-sdk)
[![Series](https://img.shields.io/badge/Series-Episode%2003-cyan?style=flat-square)](#)

---

**[🌐 Project Demo Site](https://senthilvel-k.github.io/ultrasonic-rpi-pico/)** · **[👤 Portfolio](https://senthilvel-k.github.io/)** · **[📄 Full Report](./docs/Ultrasonic_Report.pdf)**

---

## Overview

Interfaces an **HC-SR04 ultrasonic sensor** with the Raspberry Pi Pico. The Pico generates a 10µs trigger pulse, then measures how long the ECHO pin stays HIGH. That duration encodes the round-trip time of a 40kHz sound pulse — dividing by 29µs/cm (and ÷2 for round trip) gives distance in centimeters. Results print to serial console via USB CDC.

---

## What You'll Learn

- **Time-of-flight** physics — how sound speed gives distance
- Generating a precise **10µs TRIG pulse** with `gpio_put()` + `sleep_us(10)`
- **Polling the ECHO pin** to measure pulse width in µs
- Converting pulse time → **cm and inches** with the speed of sound formulas
- Timeout handling — what to do if no echo returns

---

## Distance Formulas

```
Distance (cm)   = pulse_width_µs / 29 / 2
Distance (inch) = pulse_width_µs / 74 / 2

Why ÷ 2? The sound travels TO the object AND BACK.
Speed of sound: ~343 m/s = 1cm per 29µs
```

---

## Signal Timing

```
TRIG  ──┐10µs┌────────────────────────────
         └────┘

ECHO  ────────┐←── width µs ──→┐──────────
              └────────────────┘
              (stays HIGH during echo)
```

---

## Hardware

| HC-SR04 Pin | Raspberry Pi Pico |
|-------------|-------------------|
| VCC | 3.3V |
| GND | GND |
| TRIG | GPIO 2 (OUTPUT) |
| ECHO | GPIO 3 (INPUT) |

> Output printed to PuTTY serial console at 115200 baud via USB.

---

## Core Code

```c
uint64_t getPulse(uint trigPin, uint echoPin) {
    gpio_put(trigPin, 1);
    sleep_us(10);                    // 10µs TRIG pulse
    gpio_put(trigPin, 0);

    uint width = 0;
    while (gpio_get(echoPin) == 0)   // wait ECHO HIGH
        tight_loop_contents();
    while (gpio_get(echoPin) == 1) { // count µs
        width++;
        sleep_us(1);
        if (width > 26100) return 0; // timeout
    }
    return width;
}

int getCm(uint trig, uint echo) {
    return getPulse(trig, echo) / 29 / 2;
}
```

---

## Build & Flash

```bash
git clone https://github.com/senthilvel-k/ultrasonic-rpi-pico.git
cd ultrasonic-rpi-pico && mkdir build && cd build
cmake .. && make -j4
```

---

## Series

| # | Project | Concept |
|---|---------|---------|
| 01 | [LCD Display](https://github.com/senthilvel-k/lcd-rpi-pico) | 4-bit parallel |
| 02 | [Interrupts](https://github.com/senthilvel-k/interrupt-rpi-pico) | GPIO IRQ, ISR |
| **03** | **Ultrasonic Sensor** | **Pulse timing, ToF** |
| 04 | [Keypad Matrix](https://github.com/senthilvel-k/keypad-rpi-pico) | Matrix scan |
| 05 | [Potentiometer](https://github.com/senthilvel-k/potentiometer-rpi-pico) | ADC, PWM |

> **[senthilvel-k.github.io](https://senthilvel-k.github.io/)**

*Senthil Vel K · skumara4 · Visteon Corporation · October 2023 · MIT License*
