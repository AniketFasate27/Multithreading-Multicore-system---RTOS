# ⚡ ESP32 FreeRTOS Motor Control System
### *Because your motor deserves a real-time babysitter.*

[![FreeRTOS](https://img.shields.io/badge/FreeRTOS-Enabled-brightgreen?style=for-the-badge&logo=freertos)](https://www.freertos.org/)
[![Platform](https://img.shields.io/badge/Platform-ESP32-blue?style=for-the-badge&logo=espressif)](https://www.espressif.com/)
[![Framework](https://img.shields.io/badge/Framework-Arduino-teal?style=for-the-badge&logo=arduino)](https://www.arduino.cc/)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

---

```
 ╔═══════════════════════════════════════════════╗
 ║   CORE 0          ║          CORE 1           ║
 ║  🔧 Motor Task    ║   📊 Monitor Task         ║
 ║  Ramp. Hold.      ║   Read RPM. Log State.    ║
 ║  Reverse. Stop.   ║   Detect Faults.          ║
 ║                   ║   🚨 Supervisor Task       ║
 ║                   ║   (Sleeping... until       ║
 ║                   ║    things go 💥 boom)      ║
 ╚═══════════════════════════════════════════════╝
```

---

## 🤔 What even is this?

A **dual-core FreeRTOS application** for the ESP32 that controls a DC motor via an L298N driver — while simultaneously monitoring its speed, detecting faults, and recovering gracefully. No blocking loops. No `delay()` crimes. Just clean, real-time task orchestration the way the scheduler gods intended.

> Built as a hands-on companion to the article  
> 📖 [**"Real-Time Magic: Using FreeRTOS to Supercharge Your ESP32 Projects"**](https://medium.com/embedded-connected-avinya-intelligence/freertos-3698428fc772)

---

## ✨ Features

| Feature | Details |
|---|---|
| 🧵 **Dual-Core Task Pinning** | Motor control on Core 0, monitoring on Core 1 |
| 📈 **Speed Profiling** | Ramp up → Hold → Ramp down → Brake → Reverse → Stop |
| 📡 **ISR-Based RPM Measurement** | Tachometer pulses counted via hardware interrupt |
| 🔒 **Mutex-Protected Shared State** | Safe concurrent access to motor state struct |
| ⏸️ **Suspend & Resume** | Motor task suspended on fault detection |
| 💥 **Fault Recovery** | Supervisor task wakes, applies emergency stop, recovers |
| 🗑️ **Self-Deleting Supervisor** | `vTaskDelete(NULL)` — cleans up after itself like an adult |

---

## 🔌 Hardware Setup

```
ESP32 GPIO          L298N Motor Driver
──────────          ──────────────────
GPIO 25 (PWM)  ──►  ENA  (Speed Control)
GPIO 26        ──►  IN1  (Direction A)
GPIO 27        ──►  IN2  (Direction B)
GPIO 34 (IRQ)  ◄──  Tachometer / Encoder Output
GND            ──►  GND
```

> ⚠️ **GPIO 34** is an input-only pin on the ESP32 — perfect for interrupt-based tachometer reading.  
> 🔋 Power the L298N motor supply (6–12V) **separately** from the ESP32 3.3V rail.

---

## 🏗️ Task Architecture

```
                    ┌─────────────────────────────────┐
                    │         FreeRTOS Scheduler       │
                    └────────┬──────────┬──────────────┘
                             │          │
              ┌──────────────▼──┐   ┌───▼──────────────────┐
              │  motorCtrlTask  │   │     monitorTask       │
              │  Core 0 | P:2   │   │     Core 1 | P:2      │
              │                 │   │                       │
              │  Speed profile  │   │  RPM via ISR          │
              │  Ramp up/down   │   │  500ms periodic log   │
              │  Direction ctrl │   │  Fault detection      │
              └────────┬────────┘   └──────────┬────────────┘
                       │  Mutex (motorMutex)    │
                       └───────────┬────────────┘
                                   │  On Fault →
                              ┌────▼─────────────┐
                              │  supervisorTask  │
                              │  Core 1 | P:1    │
                              │                  │
                              │  Starts SUSPENDED│
                              │  Emergency stop  │
                              │  3s cooldown     │
                              │  Resume motor    │
                              │  vTaskDelete()   │
                              └──────────────────┘
```

---

## 🚀 Getting Started

### Prerequisites
- [Arduino IDE](https://www.arduino.cc/en/software) or [PlatformIO](https://platformio.org/)
- ESP32 board support package installed
- L298N motor driver + DC motor

### Clone & Flash

```bash
git clone https://github.com/YOUR_USERNAME/esp32-freertos-motor-control.git
cd esp32-freertos-motor-control
# Open in Arduino IDE or PlatformIO and flash to your ESP32
```

### Serial Monitor Output (115200 baud)

```
=== ESP32 FreeRTOS Motor Control System ===
[MotorTask]    Started on Core 0
[MonitorTask]  Started on Core 1
[Supervisor]   Started on Core 1
[MotorTask]    Phase: Ramp UP Forward
[Monitor]  Duty:  50 | Dir: FORWARD  | RPM:  320 | Fault: NO
[Monitor]  Duty: 100 | Dir: FORWARD  | RPM:  640 | Fault: NO
[Monitor]  Duty: 200 | Dir: FORWARD  | RPM: 1280 | Fault: NO
[MotorTask]    Phase: Hold MAX Speed
[MotorTask]    Phase: Ramp DOWN
[MotorTask]    Phase: Braking
[MotorTask]    Phase: REVERSE
[MotorTask]    Phase: STOP
```

---

## 📁 Project Structure

```
esp32-freertos-motor-control/
│
├── src/
│   └── main.cpp          # All task logic, ISR, setup
│
├── README.md             # You're reading it 👀
└── LICENSE
```

---

## 🧠 FreeRTOS Concepts at a Glance

| Concept | Function Used | Where |
|---|---|---|
| Pin task to core | `xTaskCreatePinnedToCore()` | `setup()` |
| Non-blocking delay | `vTaskDelay()` / `vTaskDelayUntil()` | Motor & Monitor tasks |
| Suspend a task | `vTaskSuspend(handle)` | Monitor → suspends Motor on fault |
| Resume a task | `vTaskResume(handle)` | Supervisor → resumes Motor after recovery |
| Self-delete | `vTaskDelete(NULL)` | Supervisor after recovery |
| Mutual exclusion | `xSemaphoreCreateMutex()` | Shared `MotorState_t` struct |
| Hardware ISR | `attachInterrupt()` | Tachometer pulse counting |

---

## 📖 Learn More

This project is a companion to my Medium article series on embedded systems and IoT:

🔗 [**Real-Time Magic: Using FreeRTOS to Supercharge Your ESP32 Projects**](https://medium.com/embedded-connected-avinya-intelligence/freertos-3698428fc772)  
✍️ Published in [**Embedded & Connected**](https://medium.com/embedded-connected-avinya-intelligence)

---

## 👨‍💻 Author

**Aniket Fasate**  
M.S. Electrical & Computer Engineering @ Northeastern University  
Embedded Systems | IoT | FreeRTOS | ESP32

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/aniketfasate27/)
[![Medium](https://img.shields.io/badge/Medium-Follow-black?style=flat-square&logo=medium)](https://medium.com/@fasateaniket5)

---

## 📜 License

MIT License — go build something cool with it. 🚀
