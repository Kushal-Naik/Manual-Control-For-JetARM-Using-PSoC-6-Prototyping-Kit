# JetARM Manual Control with PSoC 6

[![Board](https://img.shields.io/badge/Board-CY8CPROTO--062--4343W-blue)](https://www.infineon.com/)
[![Framework](https://img.shields.io/badge/Framework-ModusToolbox-red)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)

A firmware implementation for manually controlling the HiWonder JetARM robotic manipulator using the Infineon PSoC 6 CY8CPROTO-062-4343W prototyping kit. 

[Insert a high-quality image or GIF here showing the PSoC 6 connected to the JetARM and the arm in motion]

## 📌 Overview

This project bypasses the stock JetARM controller, allowing direct manual actuation of the robotic arm's servos via the PSoC 6 microcontroller. It demonstrates hardware PWM generation, precise timing control for servos, and [mention any control input method you used, e.g., UART commands, analog joysticks, or push buttons].

**Key Features:**
* Independent control of [Number] servos.
* Configurable motion limits to prevent hardware binding.
* Real-time manual jogging of the manipulator.

## 🛠️ Hardware Requirements

* **Microcontroller:** PSoC 6 Wi-Fi BT Prototyping Kit (CY8CPROTO-062-4343W)
* **Robotics Platform:** HiWonder JetARM (Powered via its own external 7.4V/8.4V supply)
* **Inputs:** [e.g., 2x Analog Joysticks, Serial Terminal, etc.]
* **Misc:** Logic level shifters (if required for the servos), jumper wires.

## 🔌 Pin Configuration & Wiring

> **⚠️ WARNING:** Never power the JetARM servos directly from the PSoC 6's 5V or 3.3V pins. The servos require a dedicated, high-current power supply. Ensure the ground (GND) of the external power supply is tied to the PSoC 6 GND.

| JetARM Servo / Input | PSoC 6 Pin | Function / Notes |
| :--- | :--- | :--- |
| Base Servo (ID 1) | `P9_0` (Example) | PWM Output (50Hz) |
| Shoulder Servo (ID 2)| `P9_1` | PWM Output (50Hz) |
| [Add Joystick X/Y] | `P10_0` | ADC Input |

*(Add a link to a wiring diagram in your `/docs` folder here)*

## 💻 Software Setup & Build Instructions

This project was developed using [ModusToolbox / Eclipse / VS Code].

1. **Clone the repository:**
   ```bash
   git clone [https://github.com/yourusername/jetarm-psoc6-control.git](https://github.com/yourusername/jetarm-psoc6-control.git)
   cd jetarm-psoc6-control
