# JetARM Manual Control with PSoC 6

[![Board](https://img.shields.io/badge/Board-CY8CPROTO--062--4343W-blue)](https://www.infineon.com/)
[![Framework](https://img.shields.io/badge/Framework-ModusToolbox%203.3-red)](#)

A firmware implementation for manually controlling the HiWonder JetARM robotic manipulator using the Infineon PSoC 6 CY8CPROTO-062-4343W prototyping kit.

[Insert a high-quality image or GIF here showing the PSoC 6 connected to the JetARM and the arm in motion]

## 📌 Overview

This project bypasses the stock JetARM controller and drives the arm's **HiWonder serial bus servos** directly from the PSoC 6. The PSoC talks to the servos over UART through a **HiWonder Bus Linker V3.0** (TTL ↔ single-wire servo bus adapter). You control the arm from a serial terminal on your PC using the keyboard.

**Key Features:**
* Independent control of 6 servos (base, shoulder, elbow, wrist, wrist rotation, gripper).
* Keyboard jogging with adjustable step size, and typed absolute positions.
* Soft motion limits per joint. At startup they are narrowed to the angle limits stored in each servo.
* No movement on power-up: the firmware only reads the current positions until you give a command.
* Torque on/off per joint, so you can pose the arm by hand.
* Live status (position, torque, temperature, voltage), a bus ID scan, and built-in diagnostics.

## 🛠️ Hardware Requirements

* **Microcontroller:** PSoC 6 Wi-Fi BT Prototyping Kit (CY8CPROTO-062-4343W)
* **Robotics Platform:** HiWonder JetARM (Powered via its own external 7.4V/8.4V supply)
* **Servo interface:** HiWonder Bus Linker V3.0
* **Inputs:** Serial terminal on a PC (through the kit's KitProg3 USB port)
* **Misc:** Jumper wires, and a logic level shifter if the Bus Linker's TX line idles at 5 V (see below).

## 🔌 Pin Configuration & Wiring

> **⚠️ WARNING:** Never power the JetARM servos directly from the PSoC 6's 5V or 3.3V pins. The servos require a dedicated, high-current power supply. Ensure the ground (GND) of the external power supply is tied to the PSoC 6 GND.

```
PC ──USB── CY8CPROTO-062-4343W            Bus Linker V3.0              JetARM
           P10_1 (UART TX) ──────────────► RX
           P10_0 (UART RX) ◄────────────── TX        servo bus ───► servos (daisy-chained)
           GND ─────────────────────────── GND
                                           DC in ◄── servo power supply
```

| PSoC 6 Pin | Bus Linker V3.0 | Function / Notes |
| :--- | :--- | :--- |
| `P10_1` | RX | UART TX to the servo bus (SCB1) |
| `P10_0` | TX | UART RX from the servo bus. PSoC 6 pins are **not 5 V tolerant**, so measure the Bus Linker TX idle voltage first and add a level shifter if it is 5 V. |
| `GND` | GND | Common ground (required) |
| — | 5V / VCC | Leave unconnected |
| — | USB port | Leave **unplugged**. Its USB-serial chip shares the TX/RX lines. |

Do not use `P5_0`/`P5_1` (KitProg debug UART) or `P2_x` (Wi-Fi SDIO) for the servo bus.

## 📡 Servo Protocol

The JetARM servos use HiWonder's bus servo protocol at **115200 baud, 8N1**:

```
0x55 0x55 | ID | LEN | CMD | PARAMS... | CHECKSUM
LEN      = number of params + 3
CHECKSUM = ~(ID + LEN + CMD + PARAMS) & 0xFF
```

Positions run from 0 to 1000, which is 0–240°. Write commands (move, torque) get no reply; read commands do. The driver is in `hiwonder_bus_servo.cpp/.h`.

## 💻 Software Setup & Build Instructions

This project was developed using ModusToolbox 3.3 (make-based build, GCC_ARM).

1. **Clone the repository:**
   ```bash
   git clone https://github.com/Kushal-Naik/Manual-Control-For-JetARM-Using-PSoC-6-Prototyping-Kit.git
   cd Manual-Control-For-JetARM-Using-PSoC-6-Prototyping-Kit
   ```
2. **Point make at your ModusToolbox tools**, if they are not in `~/ModusToolbox`:
   ```bash
   export CY_TOOLS_PATHS=/opt/Tools/ModusToolbox/tools_3.3
   ```
3. **Fetch the libraries** (into `../mtb_shared`):
   ```bash
   make getlibs
   ```
4. **Build and flash** over the kit's KitProg3:
   ```bash
   make program
   ```
5. **Open a serial terminal** on the KitProg3 COM port at **115200 baud, 8N1**, then switch on the servo power.

## 🎮 Controls

| Key | Action | Key | Action |
| :--- | :--- | :--- | :--- |
| `1`–`6` | Select joint | `w` / `s` | Previous / next joint |
| `a` / `d` | Jog selected joint − / + | `[` / `]` | Halve / double the step |
| `g` | Go to an absolute position | `p` | Status of all joints |
| `e` / `E` | Hold selected / all joints | `x` / `X` | Torque off selected / **all** joints |
| `space` | Hold all at current position | `c` | Scan the bus for servo IDs |
| `H` / `h` | Save / go to home pose | `?` | Help |
| `D` | Raw bus diagnostics | `L` | UART loopback self-test |

> `X` releases every joint and the arm will drop, so support it first.

## 🔧 First-Time Setup

1. Press `c` to scan the bus, then set the real servo IDs in the `joints[]` table in `jetarm_control.cpp`. The defaults are IDs 1–6.
2. Press `X` while supporting the arm, move each joint by hand through its safe range, and read the ends with `p`. Enter those values as `min`/`max` in `joints[]`.
3. Rebuild and flash with `make program`.

## 🩺 Troubleshooting

* **All joints show `NO` / the scan finds nothing:** check that TX and RX are crossed, that the grounds are shared, and that the Bus Linker USB is unplugged.
* **`L` loopback test:** disconnect the Bus Linker and jumper `P10_1` to `P10_0`. `PASS` means the PSoC UART and pins work.
* **`D` diagnostics:** sends a position read to each joint and prints the raw bytes received. A healthy reply looks like `55 55 <id> 05 1C <lo> <hi> <chk>`.

## 📁 Project Structure

| Path | Description |
| :--- | :--- |
| `main.c` | Board init, debug UART, starts the control console |
| `jetarm_control.cpp/.h` | Keyboard console, joint table, limits, diagnostics |
| `hiwonder_bus_servo.cpp/.h` | HiWonder bus servo driver (`cyhal_uart`) |
| `bsps/` | Board support packages (CY8CPROTO-062-4343W is the active target) |
| `deps/` | Library dependencies (retarget-io) |
| `Makefile` | ModusToolbox build settings |

## 📄 License

The project's own code is released under the [MIT License](LICENSE). Files derived from the Infineon/Cypress ModusToolbox template (`main.c` and the `bsps/` folder) remain under the [Cypress End User License Agreement](LICENSE-Cypress).
