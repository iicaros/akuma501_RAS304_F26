---
title: Component Selection
tags:
- tag1
- tag2
---

## Overview

Sensor: A SparkFun LIS3DH 3-axis accelerometer breakout (with headers) captures motion and vibration data when an object starts to slip. It talks to the PIC18F57Q43 over I2C (RB1 = SCL, RB2 = SDA), and its INT1 pin (RA0) can alert the microcontroller the moment a motion threshold is crossed. The original plan used an analog ADXL335BCPZ with op-amp filters and peak detectors; that design was dropped because the ADXL335BCPZ only comes in a surface-mount package (see below).

Power Source: A 9V battery feeds the barrel jack and an onboard L7805 regulator, which provides 5V for the Curiosity Nano, LEDs, and push button. An LD1117V33 steps the 5V rail down to 3.3V for the accelerometer breakout and its I2C bus.

Team Connections: The 2x4 interboard connector (J2) runs to our main hub (Sakiya: Board 1). Pin 1 receives UART data from the hub, Pin 2 sends UART data to the hub, Pin 3 carries the shared ESTOP signal, Pin 4 sends the SLIP_INT alert, and Pin 5 is ground.

---

## Major Component Selections

### **3-Axis Accelerometer (Slip Detection Subsystem)**

1. Analog Devices `ADXL335BCPZ` 3-Axis Analog Accelerometer

    * $8.88/each
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/analog-devices-inc/ADXL335BCPZ/1999326)

    | Pros | Cons |
    | --- | --- |
    | Small size fits directly on the PCB without needing a separate daughterboard | Very small pins make it harder to solder by hand |
    | Outputs analog signals for X, Y, and Z axes to go straight into the filters | Needs extra capacitors on the board to set the filter frequency |
    | Good sensitivity for picking up small slip vibrations and in stock on DigiKey | Only runs on 3.3V, so it needs a voltage regulator |

1. NXP `MMA7361LCR1` 3-Axis Analog Accelerometer

    * $(no info)/each
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/nxp-usa-inc/MMA7361LCR1/2508910)

    | Pros | Cons |
    | --- | --- |
    | Can switch between two sensitivity modes | Discontinued part with no pricing or stock available on DigiKey |
    | Outputs analog signals that work with the filter design | Pads are hidden underneath the chip, making it hard to check solder joints |
    | Uses very little power while running | Can max out too easily if the gripper moves fast |

1. ROHM `KXTC9-2050-FR` 3-Axis Analog Accelerometer

    * $19.54/each
    * [WinSource Electronics Product Page](https://www.win-source.net/products/detail/kionix-inc/kxtc9-2050-fr.html)

    | Pros | Cons |
    | --- | --- |
    | Direct X, Y, and Z analog outputs that work with the filter stages | Most expensive option at $19.54 and only available through a third-party broker (WinSource) |
    | Uses very low power (240 uA) | Tiny 3mm x 3mm package is very difficult to place and solder |
    | Filter bandwidth can be adjusted with external capacitors | More signal noise at lower vibration speeds and requires a 3.3V line |

1. Analog Devices `ADXL345BCCZ` 3-Axis Digital Accelerometer

    * $8.01/each
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/analog-devices-inc/ADXL345BCCZ/2034829)

    | Pros | Cons |
    | --- | --- |
    | Least expensive in-stock accelerometer on DigiKey ($8.01) | Outputs digital signals, so it will not work with the analog high-pass filters and peak detectors |
    | Has built-in tap and motion detection | Requires writing extra code to read data over SPI or I2C |
    | Mounts directly to the PCB with fewer extra parts | Hidden pads underneath the package are hard to inspect after soldering |

1. SparkFun `SEN-20659` Triple Axis Accelerometer Breakout - LIS3DH (with Headers)

    * $7.70/each
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/sparkfun-electronics/20659/21703932)

    | Pros | Cons |
    | --- | --- |
    | Comes with 0.1" through-hole headers, so it plugs into female sockets on the PCB with no surface-mount soldering | Breakout board takes up more PCB space and height than a bare chip |
    | Digital I2C output removes the need for the high-pass filters, peak detectors, and op-amps | Requires writing I2C code to configure the sensor and read data |
    | Built-in motion and free-fall interrupts (INT1) can flag a slip immediately; has its own I2C pull-ups and decoupling | Limited DigiKey stock (22 at time of ordering) and must run on 3.3V (1.71V - 3.6V) |

**Original Choice:** Option 1: Analog Devices `ADXL335BCPZ`

**Original Rationale:** The original design needed an analog accelerometer that mounts directly onto the PCB and sends X, Y, and Z signals into the high-pass filters and peak detectors. Unlike the `MMA7361L` (which is discontinued with no pricing) and the `KXTC9-2050` (which costs $19.54 from a third-party broker), the `ADXL335BCPZ` is reasonably priced ($8.88) and directly available on DigiKey. It also picks up small vibrations well without maxing out when the gripper moves, and is easier to check for bad solder joints.

**Final Choice:** Option 5: SparkFun `SEN-20659` LIS3DH Breakout (with Headers)

**Rationale for the Change:** After the schematic review, the `ADXL335BCPZ` could not be used on this board. It only comes in a 4 mm x 4 mm surface-mount LFCSP package, and this PCB has to use through-hole parts. None of the other analog options solve this, since they are also small surface-mount packages. With instructor approval, an accelerometer breakout board was used instead. The SparkFun LIS3DH breakout (with headers) was chosen because its 0.1" through-hole pins plug directly into two 1x6 female headers on the PCB, it runs off the existing 3.3V regulator, and it costs less than the `ADXL335BCPZ` ($7.70 vs. $8.88). Because it outputs digital data over I2C, the op-amp high-pass filters and peak detectors were removed from the design, which cut the parts count and cost.

---

### **3.3V Voltage Regulator (Power Supply Subsystem)**

1. STMicroelectronics `LD1117V33` 3.3V Linear Regulator

    * $1.27/each
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/stmicroelectronics/LD1117V33/586012)

    | Pros | Cons |
    | --- | --- |
    | Outputs a steady 3.3V and provides way more current (800 mA) than the accelerometer needs | Most expensive regulator option ($1.27 each) and uses up to 10 mA just to stay on |
    | Can step 5V down to 3.3V without any issues | Takes up more space on the PCB than smaller regulators |
    | Large through-hole package is easy to solder and has built-in overheat protection | |

1. Microchip `MCP1700-3302E/TO` 3.3V Linear Regulator

    * $0.50/each
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/microchip-technology/MCP1700-3302E-TO/652680)

    | Pros | Cons |
    | --- | --- |
    | Least expensive regulator option ($0.50 each) and wastes almost no power | Will burn out if connected to anything over 6V |
    | Small through-hole size saves space on the board | Only supplies up to 250 mA and cannot handle as much heat |
    | Works with standard 1 uF ceramic capacitors | |

1. STMicroelectronics `LM317T` Adjustable Regulator

    * $1.13/each
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/stmicroelectronics/LM317T/591677)

    | Pros | Cons |
    | --- | --- |
    | Can handle high current (1.5A) and wide input voltages | Needs at least 5.3V input to make 3.3V, so it will not work properly off the 5V line |
    | Rugged through-hole package | Almost as expensive as the `LD1117V33` ($1.13) plus needs two extra resistors to set the voltage |

**Choice:** Option 1: STMicroelectronics `LD1117V33`

**Rationale:** Since the accelerometer needs 3.3V and the board's main rail is 5V (from the onboard L7805 regulator), a regulator is needed that can drop 5V down to 3.3V. The `LD1117V33` does this reliably without needing extra resistors like the `LM317T`, and its rugged TO-220 package is much harder to accidentally damage during testing than the smaller `MCP1700`.

---

### **External Power Source (Power Supply Subsystem)**

1. CUI Inc `SWI12-9-N-P5` Regulated Wall Supply (via Main Hub / Curiosity Nano 5V Line)

    * $17.71/each
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/cui-inc./SWI12-9-N-P5/102-3428-ND/5287237)

    | Pros | Cons |
    | --- | --- |
    | Gives steady power (up to 1300 mA) without dying during testing | Highest upfront cost ($17.71) out of the power options |
    | Easily powers the Curiosity Nano, accelerometer, and 3.3V regulator | Keeps the gripper plugged into a wall outlet and is not portable |

1. Tenergy 9V 600mAh Rechargeable Li-Ion Battery (`30003`)

    * $11.99/each
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/tenergy-corporation/30003/16634139)

    | Pros | Cons |
    | --- | --- |
    | Allows testing the board without wall cords and costs less than the wall adapter ($11.99) | Needs an extra 5V regulator before going into the board |
    | Can be recharged instead of buying new batteries | Needs a separate battery charger |

1. Energizer `EN22` 9V Alkaline Battery

    * $2.75/each
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/energizer-battery-company/EN22/706980)

    | Pros | Cons |
    | --- | --- |
    | Lowest upfront cost ($2.75) and easy to swap in for quick testing | Runs out after about 4.8 hours of use and cannot be recharged |
    | Small and easy to clip onto a test board | Needs a 5V regulator and turns extra voltage into heat |

**Choice:** Option 3: 9V Alkaline Battery through the barrel jack

**Rationale:** Having all five team boards plugged into wall outlets at the same time would be difficult to manage, so each board runs from its own 9V battery through a barrel jack. The onboard L7805 regulator turns the 9V into a steady 5V for the Curiosity Nano, LEDs, and push button, and the LD1117V33 then supplies 3.3V to the accelerometer breakout. The battery is inexpensive, easy to swap, and keeps the gripper portable.


## Pin Table

| MCU Pin | Mode | Direction | Connected To | Description |
| --- | --- | --- | --- | --- |
| `RB1` | `I2C SCL` | Output | LIS3DH Breakout (SCL), J3 Pin 5 | I2C clock to the accelerometer (3.3V bus) |
| `RB2` | `I2C SDA` | Bidirectional | LIS3DH Breakout (SDA), J3 Pin 6 | I2C data to/from the accelerometer (3.3V bus) |
| `RA0` | `Digital I/O` (interrupt-on-change) | Input | LIS3DH Breakout (INT1) | Motion/slip interrupt from the accelerometer |
| `RA1` | `Digital I/O` | Output | J2 Pin 4 (SLIP_INT) | Slip alert to Main Hub (Board 1) |
| `RA2` | `UART TX` | Output | J2 Pin 2 (UART_TX) | Serial data to Main Hub (Board 1) |
| `RC4` | `UART RX` | Input | J2 Pin 1 (UART_RX) | Serial data from Main Hub (Board 1) |
| `RB0` | `Digital I/O` (interrupt) | Input | J2 Pin 3 (ESTOP) | Shared emergency stop from Main Hub |
| `RA4` | `Digital I/O` | Input | Push button SW1 (10k pull-up) | Test slip alert / clear alert (reads low when pressed) |
| `RD0` | `Digital I/O` | Output | Green debug LED (D3) | Status indicator |
| `RD1` | `Digital I/O` | Output | Red debug LED (D4) | Slip / fault indicator |
| `RA3, RA7, RB3, RB5, RC0, RC1, RC2, RC3, RC5, RD3, RD4, RD5` | `Digital I/O` | Unused | J3 Extra Header (EH1-EH12) | Spare GPIO for debugging and expansion |

> ## Component Selection Rationale Overview

To meet the requirements of creating this PCB Node, I selected a SparkFun LIS3DH accelerometer breakout, an LD1117V33 3.3V regulator, and a 9V alkaline battery with a barrel jack connector.
The ADXL335BCPZ was originally chosen because it was the most reasonably priced and readily available analog accelerometer, but it only comes in a surface-mount package, which the through-hole PCB cannot use. With instructor approval, it was replaced by the SparkFun LIS3DH breakout, which plugs into female headers on the PCB and sends its data digitally over I2C. This removed the need for op-amp filters and peak detectors.
While a wall adapter is an option, having all five of our team members' boards plugged into the wall at the same time would be difficult to manage.
Instead, a 9V battery with a barrel jack connector feeds into the onboard L7805 regulator to provide a steady 5V for the system while allowing mobility.
From this 5V rail, a dedicated LD1117V33 voltage regulator steps the power down to safely supply 3.3V to the accelerometer breakout.