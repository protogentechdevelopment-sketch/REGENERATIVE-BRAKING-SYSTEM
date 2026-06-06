# ⚡ REGENERATIVE BRAKING SYSTEM

> An embedded IoT system for kinetic energy recovery during braking in two-wheelers — built on ESP8266 CH340 with L298N motor driver, Li-ion battery, BMS, and real-time LCD feedback.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [System Architecture](#system-architecture)
- [Hardware Components](#hardware-components)
- [Software Tools](#software-tools)
- [Communication Protocols](#communication-protocols)
- [Circuit & Pin Connections](#circuit--pin-connections)
- [How It Works](#how-it-works)
- [Energy Recovery Algorithm](#energy-recovery-algorithm)
- [Results](#results)
- [Future Scope](#future-scope)

---

## Overview

Conventional two-wheelers rely entirely on friction-based braking, dissipating all kinetic energy as heat during deceleration. This project eliminates that waste with an **intelligent, in-wheel regenerative braking system** built on embedded hardware.

When the rider applies the brake, a DC motor switches to generator mode, recovering kinetic energy and storing it in a Li-ion battery — the total bill of energy wastage is updated in real time on an LCD display. **No external server, no manual switching, no wasted braking energy.**

---

## Features

- ✅ Automatic braking event detection via brake pressure sensor
- ✅ DC motor operating in dual mode — drive motor during acceleration, generator during braking
- ✅ Real-time energy recovery with Li-ion battery storage via BMS (1S C TYPE)
- ✅ Battery Management System (BMS) — overcharge, over-discharge, and short circuit protection
- ✅ Real-time voltage and power monitoring via voltage sensor + ADC
- ✅ 16×2 I2C LCD display — shows battery voltage, energy recovered, and system mode
- ✅ L298N H-bridge motor driver — PWM-controlled bidirectional motor operation
- ✅ 5V relay module — safe switching between standard braking and regenerative circuits
- ✅ Hall effect throttle controller — smooth acceleration and regenerative intensity modulation
- ✅ LED indicators for real-time system state feedback
- ✅ Standalone operation — no network or external server required
- ✅ Scalable for IoT/cloud integration via ESP8266 Wi-Fi

---

## System Architecture

```
┌───────────────────────────────────────────────────────────────┐
│  INPUT LAYER       Brake Pressure Sensor → Braking Event      │
│                    Throttle Controller  → Rider Demand        │
│                    Voltage Sensor       → Battery Monitoring  │
├───────────────────────────────────────────────────────────────┤
│  PROCESSING LAYER  ESP8266 CH340 (Central MCU)                │
│                    Arduino-based firmware (ADC + PWM + I2C)   │
├───────────────────────────────────────────────────────────────┤
│  CONTROL LAYER     L298N Motor Driver → DC Motor (dual mode)  │
│                    5V Relay Module    → Circuit Switching     │
├───────────────────────────────────────────────────────────────┤
│  STORAGE LAYER     Li-ion Battery ← BMS (charge protection)   │
├───────────────────────────────────────────────────────────────┤
│  OUTPUT LAYER      16×2 I2C LCD Display                       │
│                    LED Indicators (system state)              │
└───────────────────────────────────────────────────────────────┘
```

### Block Diagram

```
Throttle Controller ──► ESP8266 CH340 ──► L298N Motor Driver ──► 12V DC Motor
                              │                                        │
                              │◄──── Voltage Sensor ◄──── Li-ion Battery ◄── (Generator mode)
                              │
                              ├──► 5V Relay Module ──► LED [LOAD]
                              │                              │
                              │                         1S C TYPE BMS
                              │                              │
                              │                         Li-ion Battery
                              │
                              └──► 16×2 LCD Display
```

---

## Hardware Components

| Component | Model | Interface | Role |
|---|---|---|---|
| Main MCU | ESP8266 CH340 (Xtensa LX106, 80 MHz, Wi-Fi) | — | Central processing, sensor fusion, motor control |
| Motor Driver | L298N Dual H-Bridge | PWM + GPIO | Bidirectional DC motor control (drive & generator) |
| DC Motor | 12V DC Motor | Dual terminal | Drive motor + energy recovery generator |
| Voltage Sensor | Resistive voltage divider module (0–25V) | ADC (A0) | Battery state monitoring |
| Display | 16×2 LCD with I2C backpack (PCF8574) | I2C | Real-time voltage, power, and system status |
| Relay | 5V Relay Module | GPIO | Switches between braking and regenerative circuits |
| Throttle | Hall Effect Throttle Controller | ADC | Rider acceleration demand input |
| Battery | Li-ion Cell (3.7V nominal) | B+/B− | Primary energy storage |
| BMS | 1S C TYPE BMS | FET-protected | Overcharge, over-discharge, short circuit protection |
| Indicators | LEDs (multiple) | GPIO | Visual feedback on system state |

---

## Software Tools

| Tool | Purpose |
|---|---|
| **Arduino IDE** | Firmware development, compilation, USB flashing via CH340G |
| **ESP8266 Community BSP** | Board support package for ESP8266 in Arduino IDE |
| **Serial Monitor** | Real-time debugging of ADC readings and system states |

---

## Communication Protocols

### I2C — ESP8266 ↔ 16×2 LCD Display

| ESP8266 Pin | LCD Backpack Pin | Signal |
|---|---|---|
| GPIO 4 (D2) | SDA | Serial data |
| GPIO 5 (D1) | SCL | Serial clock |
| 5V | VCC | Power |
| GND | GND | Ground |

- Library: `LiquidCrystal_I2C`
- I2C Address: `0x27`
- Mode: 4-bit parallel (handled by PCF8574 expander on backpack)

### GPIO/PWM — ESP8266 ↔ L298N Motor Driver

| ESP8266 Pin | L298N Pin | Signal |
|---|---|---|
| GPIO 12 (D6) | IN1 | Motor direction A |
| GPIO 13 (D7) | IN2 | Motor direction B |
| GPIO 14 (D5) | ENA | PWM speed / charging current control |
| — | VCC (12V) | Motor power supply |
| GND | GND | Common ground |

### ADC — ESP8266 ↔ Voltage Sensor

| ESP8266 Pin | Sensor Pin | Signal |
|---|---|---|
| A0 | OUT | Scaled analog voltage (0–1V range) |
| 5V | VCC | Sensor power |
| GND | GND | Common ground |

### GPIO — ESP8266 ↔ 5V Relay Module

| ESP8266 Pin | Relay Pin | Signal |
|---|---|---|
| GPIO 2 (D4) | IN (Signal) | Digital control (HIGH = regenerative mode) |
| 5V | VCC | Relay coil power |
| GND | GND | Common ground |

---

## Circuit & Pin Connections

### DC Motor ↔ L298N

| Motor Terminal | L298N Pin | Function |
|---|---|---|
| Terminal A (+) | OUT1 | Forward drive / generator output + |
| Terminal B (−) | OUT2 | Reverse / generator output − |

### Li-ion Battery ↔ BMS

| Battery Terminal | BMS Pin | Function |
|---|---|---|
| B+ | B+ | Battery positive input |
| B− | B− | Battery negative input |
| — | P+/P− | Protected output to load and charging circuits |

### Throttle Controller ↔ ESP8266

| Throttle Pin | ESP8266 Pin | Signal |
|---|---|---|
| OUT (Signal) | GPIO 0 (D3) | Analog voltage 0.8V–4.2V |
| VCC | 5V | Power |
| GND | GND | Common ground |

### LED Indicator ↔ ESP8266

| LED Pin | ESP8266 Pin | Notes |
|---|---|---|
| Anode (+) | GPIO 16 (D0) | Via 330Ω current-limiting resistor |
| Cathode (−) | GND | Common ground |

### Voltage Calculation Formula

```
ADC Raw Value → Scaled Voltage (via calibration formula)

Voltage = map(adcValue, 290, 903, 80, 720) / 100.0

Example:
  ADC Raw = 600, mapped to 3.85V (battery level display)
  PWM     = map(adcValue, 290, 903, 0, 255) → motor drive strength
```

---

## How It Works

1. **Boot** — ESP8266 initializes I2C (LCD), GPIO (relay, LED, motor driver), ADC (voltage sensor, throttle). LCD shows "REGENERATIVE BRAKING SYSTEM" splash for 2 seconds.
2. **Acceleration Mode** — Throttle controller sends analog signal (0.8V–4.2V) to ESP8266 ADC. ESP8266 maps this to a PWM duty cycle, driving the L298N to spin the DC motor forward. Relay stays OFF (standard circuit active).
3. **Braking Detection** — Brake pressure sensor detects a braking event. ESP8266 evaluates ADC input; when the signal falls below threshold (ADC < 299), relay activates (HIGH), switching the circuit to regenerative mode.
4. **Regenerative Mode** — ESP8266 commands L298N to reverse H-bridge configuration. The spinning motor shaft generates back-EMF (Faraday's law), driving current through the BMS into the Li-ion battery. PWM duty cycle controls the charging current rate.
5. **BMS Protection** — BMS continuously monitors cell voltage and charging current. If voltage exceeds 4.2V per cell or current exceeds safe limits, protection FETs cut the charging circuit, preventing overcharge.
6. **Display Update** — ESP8266 transmits real-time data over I2C to the 16×2 LCD: battery voltage, system mode (ACCEL / REGEN), and energy recovered. LED indicators reflect system state changes.
7. **Mode Transition** — When braking ends, relay returns LOW, reconnecting the standard circuit. Motor driver returns to forward drive mode. System continuously cycles between acceleration and regenerative braking.

### Firmware State Machine

```
ESP8266 reads ADC continuously
  └─► ADC < 299 → RELAY ON → REGEN MODE
        └─► L298N reversed → Motor acts as generator
              └─► Back-EMF → BMS → Li-ion Battery charged
                    └─► LCD shows: "REGEN | V: x.xx"

  └─► ADC ≥ 299 → RELAY OFF → ACCEL MODE
        └─► L298N forward → Motor drives wheel
              └─► Throttle maps to PWM duty cycle
                    └─► LCD shows: "ACCEL | V: x.xx"
```

### ESP8266 Firmware Snippet (Core Loop)

```cpp
void loop() {
    int adcValue = analogRead(A0);
    int pwmValue = map(adcValue, 290, 903, 0, 255);
    pwmValue = constrain(pwmValue, 0, 255);
    if (pwmValue < 60) pwmValue = 0;
    analogWrite(ENA, pwmValue);

    bool relayState;
    if (adcValue < 299) {
        digitalWrite(RELAY, HIGH);  // Regenerative mode
        relayState = true;
    } else {
        digitalWrite(RELAY, LOW);   // Acceleration mode
        relayState = false;
    }

    float voltage = map(adcValue, 290, 903, 80, 720) / 100.0;
    float power   = voltage * current;

    lcd.setCursor(0, 0);
    lcd.print(relayState ? "REGEN" : "ACCEL");
    delay(100);
}
```

---

## Energy Recovery Algorithm

The system uses a **threshold-based sensor fusion algorithm** to maximize energy recovery while maintaining safe braking performance:

- **Brake pressure detection** triggers regenerative mode entry
- **Wheel speed check** prevents regenerative torque at near-zero speeds (stability risk)
- **Regenerative intensity** is proportional to brake pressure — natural braking feel for rider
- **Battery voltage monitoring** via voltage sensor — if battery approaches maximum safe voltage, regenerative braking auto-deactivates to prevent overcharge
- **BMS current limiting** — if charging current exceeds safe rate, algorithm reduces regenerative intensity via PWM modulation
- **Voltage decay simulation** — smooth voltage ramp-down display after peak is reached (regVoltage -= 0.25 per cycle)

---

## Results

| Metric | Conventional Braking | Proposed System |
|---|---|---|
| Braking energy recovery | 0% (all lost as heat) | Measurable energy stored in Li-ion |
| Battery charging | External charger only | Recovered during every braking event |
| Mode transition latency | — | < 100 ms (ADC polling at 10 Hz) |
| LCD update rate | — | Real-time (100 ms refresh) |
| BMS overcharge protection | — | Verified — disconnects at 4.2V/cell |
| Standalone operation | — | ✅ No network or server required |
| Platform | Fixed friction brakes | Compact, modular, adaptable |

- Dual-mode DC motor (drive + generator) successfully demonstrated on hardware prototype
- BMS protection functions verified under overcharge test conditions — circuit correctly interrupted
- LCD and LED indicators provided reliable real-time system state feedback across all test cycles
- System transitions between ACCEL and REGEN modes without disruption to normal vehicle operation

---

## Future Scope

- **Machine Learning** — Adaptive regenerative braking using ML models trained on riding patterns to dynamically optimize energy recovery
- **IoT / Cloud** — Push battery data to AWS IoT / ThingSpeak via ESP8266 Wi-Fi for real-time analytics
- **Mobile App** — Real-time cart monitoring, energy recovered, and battery status on rider's smartphone
- **Supercapacitor Storage** — Add supercapacitors alongside Li-ion battery to handle high peak currents during hard braking
- **ABS Integration** — Combine regenerative braking with Anti-lock Braking System to prevent wheel lockup during energy recovery
- **Bidirectional DC/DC Converter** — Replace L298N with advanced buck-boost converter topology for higher efficiency in both directions
- **OTA Firmware Updates** — Over-the-air updates via ESP8266 Wi-Fi without physical reflashing

> *Recovering the energy that conventional braking throws away — cleaner rides, longer range, zero queues at the charger.*
