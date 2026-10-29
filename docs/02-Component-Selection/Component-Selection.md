---
title: Component Selection
tags:
- tag1
- tag2
---

## Overview

Sensor: An ADXL335BCPZ will be used to capture 3-axis motion and vibration data when an object starts to slip. The three signals are ran through high-pass filters and peak detectors using TLV9062 op-amps before entering the microcontroller's ADC (RA0, RA6, RA5).

Power Source: Power is provided through a regulated 3.3V line for the accelerometer and a 5V supply from the PIC18F57Q43 Curiosity Nano powers the signal conditioning array.

Team Connections: Connector 1 will run to our main hub (Sakiya: Board 1) to send slip detection data to the other boards so they can adjust the grip. Pins 1-4 provide digital feedback and Pin 7 sends an analog signal to the main control board.

---

## Major Component Selections

### **3-Axis Accelerometer (Slip Detection Subsystem)**

1. Analog Devices `ADXL335BCPZ` 3-Axis Analog Accelerometer

    ![](images/adxl335bcpz.png)

    * $8.88/each
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/analog-devices-inc/ADXL335BCPZ/1999326)

    | Pros | Cons |
    | --- | --- |
    | Small size fits directly on our PCB without needing a separate daughterboard | Very small pins make it harder to solder by hand |
    | Outputs analog signals for X, Y, and Z axes to go straight into our filters | Needs extra capacitors on the board to set the filter frequency |
    | Good sensitivity for picking up small slip vibrations and in stock on DigiKey | Only runs on 3.3V, so it needs a voltage regulator |

1. NXP `MMA7361LCR1` 3-Axis Analog Accelerometer

    ![](images/mma7361l.png)

    * $(no info)/each
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/nxp-usa-inc/MMA7361LCR1/2508910)

    | Pros | Cons |
    | --- | --- |
    | Can switch between two sensitivity modes | Discontinued part with no pricing or stock available on DigiKey |
    | Outputs analog signals that work with our filter design | Pads are hidden underneath the chip, making it hard to check solder joints |
    | Uses very little power while running | Can max out too easily if the gripper moves fast |

1. ROHM `KXTC9-2050-FR` 3-Axis Analog Accelerometer

    ![](images/kxtc9_2050.png)

    * $19.54/each
    * [WinSource Electronics Product Page](https://www.win-source.net/products/detail/kionix-inc/kxtc9-2050-fr.html)

    | Pros | Cons |
    | --- | --- |
    | Direct X, Y, and Z analog outputs that work with our filter stages | Most expensive option at $19.54 and only available through a third-party broker (WinSource) |
    | Uses very low power (240 uA) | Tiny 3mm x 3mm package is very difficult to place and solder |
    | Filter bandwidth can be adjusted with external capacitors | More signal noise at lower vibration speeds and requires a 3.3V line |

1. Analog Devices `ADXL345BCCZ` 3-Axis Digital Accelerometer

    ![](images/adxl345bccz.png)

    * $8.01/each
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/analog-devices-inc/ADXL345BCCZ/2034829)

    | Pros | Cons |
    | --- | --- |
    | Least expensive in-stock accelerometer on DigiKey ($8.01) | Outputs digital signals, so it will not work with our analog high-pass filters and peak detectors |
    | Has built-in tap and motion detection | Requires writing extra code to read data over SPI or I2C |
    | Mounts directly to the PCB with fewer extra parts | Hidden pads underneath the package are hard to inspect after soldering |

**Choice:** Option 1: Analog Devices `ADXL335BCPZ`

**Rationale:** We need an analog accelerometer that mounts directly onto our PCB and sends X, Y, and Z signals into our high-pass filters and peak detectors. Unlike the `MMA7361L` (which is discontinued with no pricing) and the `KXTC9-2050` (which costs $19.54 from a third-party broker), the `ADXL335BCPZ` is reasonably priced ($8.88) and directly available on DigiKey. It also picks up small vibrations well without maxing out when the gripper moves, and is easier to check for bad solder joints.

---

### **Signal Conditioning Op-Amps (Filter & Peak Detection Subsystem)**

1. Texas Instruments `TLV9062` Dual Op-Amp

    ![](images/tlv9062.png)

    * $1.24/each (3 chips needed for 6 total stages)
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/texas-instruments/TLV9062IDR/7364940)

    | Pros | Cons |
    | --- | --- |
    | Runs directly off our 5V line and uses the full voltage range for the ADC | Highest cost of the three op-amp options ($1.24 each vs. under $0.50) |
    | Fast enough to catch quick slip spikes accurately | Uses slightly more power when idle than slower chips |
    | Holds the peak voltage steady until the microcontroller reads it | Surface-mount SOIC-8 package takes more care to solder than through-hole |

1. Microchip `MCP6002-I/P` Dual Op-Amp

    ![](images/mcp6002.png)

    * $0.45/each (3 chips needed)
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/microchip-technology/MCP6002-I-P/500875)

    | Pros | Cons |
    | --- | --- |
    | Comes in an easy-to-solder through-hole package | Slower response speed might miss fast slip vibrations |
    | Works on our 5V rail with full voltage output | Less accurate, which can add small voltage errors across our filters |
    | Very inexpensive ($0.45 each) and low power | |

1. Texas Instruments `LM358P` Dual Op-Amp

    ![](images/lm358p.png)

    * $0.33/each (3 chips needed)
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/texas-instruments/LM358P/277042)

    | Pros | Cons |
    | --- | --- |
    | Lowest price ($0.33 each) and easy to find in stock | Cuts off the top 1.5V of the signal, so we lose reading range on the ADC |
    | Easy-to-solder through-hole package | Drains the peak detector capacitor too fast, making the signal drop before we can read it |

**Choice:** Option 1: Texas Instruments `TLV9062`

**Rationale:** Each of our three axes goes through a high-pass filter and a peak detector, which takes 6 op-amp stages total. Even though the `TLV9062` costs a little more ($1.24 each) than the `MCP6002` ($0.45) and `LM358P` ($0.33), it runs right off the 5V supply from the Curiosity Nano, catches fast vibration spikes, and keeps the peak detector voltage from dropping before the microcontroller's ADC reads it.

---

### **3.3V Voltage Regulator (Power Supply Subsystem)**

1. STMicroelectronics `LD1117V33` 3.3V Linear Regulator

    ![](images/ld1117v33.png)

    * $1.27/each
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/stmicroelectronics/LD1117V33/586012)

    | Pros | Cons |
    | --- | --- |
    | Outputs a steady 3.3V and provides way more current (800 mA) than our accelerometer needs | Most expensive regulator option ($1.27 each) and uses up to 10 mA just to stay on |
    | Can step 5V down to 3.3V without any issues | Takes up more space on the PCB than smaller regulators |
    | Large through-hole package is easy to solder and has built-in overheat protection | |

1. Microchip `MCP1700-3302E/TO` 3.3V Linear Regulator

    ![](images/mcp1700.png)

    * $0.50/each
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/microchip-technology/MCP1700-3302E-TO/652680)

    | Pros | Cons |
    | --- | --- |
    | Least expensive regulator option ($0.50 each) and wastes almost no power | Will burn out if connected to anything over 6V |
    | Small through-hole size saves space on the board | Only supplies up to 250 mA and cannot handle as much heat |
    | Works with standard 1 uF ceramic capacitors | |

1. STMicroelectronics `LM317T` Adjustable Regulator

    ![](images/lm317t.png)

    * $1.13/each
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/stmicroelectronics/LM317T/591677)

    | Pros | Cons |
    | --- | --- |
    | Can handle high current (1.5A) and wide input voltages | Needs at least 5.3V input to make 3.3V, so it will not work properly off our 5V line |
    | Rugged through-hole package | Almost as expensive as the `LD1117V33` ($1.13) plus needs two extra resistors to set the voltage |

**Choice:** Option 1: STMicroelectronics `LD1117V33`

**Rationale:** Since our accelerometer needs 3.3V and our board gets 5V from the Curiosity Nano, we need a regulator that can drop 5V down to 3.3V. The `LD1117V33` does this reliably without needing extra resistors like the `LM317T`, and its rugged TO-220 package is much harder to accidentally damage during testing than the smaller `MCP1700`.

---

### **External Power Source (Power Supply Subsystem)**

1. CUI Inc `SWI12-9-N-P5` Regulated Wall Supply (via Main Hub / Curiosity Nano 5V Line)

    ![](images/wall_adapter_9v.png)

    * $17.71/each
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/cui-inc./SWI12-9-N-P5/102-3428-ND/5287237)

    | Pros | Cons |
    | --- | --- |
    | Gives steady power (up to 1300 mA) without dying during testing | Highest upfront cost ($17.71) out of the power options |
    | Easily powers the Curiosity Nano, op-amps, and 3.3V regulator | Keeps the gripper plugged into a wall outlet and is not portable |

1. Tenergy 9V 600mAh Rechargeable Li-Ion Battery (`30003`)

    ![](images/tenergy_9v.png)

    * $11.99/each
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/tenergy-corporation/30003/16634139)

    | Pros | Cons |
    | --- | --- |
    | Lets us test the board without wall cords and costs less than the wall adapter ($11.99) | Needs an extra 5V regulator before going into the board |
    | Can be recharged instead of buying new batteries | Needs a separate battery charger |

1. Energizer `EN22` 9V Alkaline Battery

    ![](images/energizer_9v.png)

    * $2.75/each
    * [DigiKey Product Page](https://www.digikey.com/en/products/detail/energizer-battery-company/EN22/706980)

    | Pros | Cons |
    | --- | --- |
    | Lowest upfront cost ($2.75) and easy to swap in for quick testing | Runs out after about 4.8 hours of use and cannot be recharged |
    | Small and easy to clip onto a test board | Needs a 5V regulator and turns extra voltage into heat |

**Choice:** Option 1: Regulated 5V Supply from the PIC18F57Q43 Curiosity Nano / Main Hub (backed by the wall supply or a 9V battery for testing)

**Rationale:** Our board gets its 5V supply through the Curiosity Nano / Main Hub connection, which powers the `TLV9062` op-amps and feeds the 3.3V regulator for the accelerometer. Even though the wall adapter has a higher upfront price, it gives us steady power when hooked up to Board 1 without worrying about dead batteries, while an inexpensive $2.75 9V battery can be used if we need to test away from an outlet.


## Power Budget

### Section A: Major Components & Maximum Current

| Component | Function | Qty | Voltage | Max Current (Each) | Total Max Current |
| --- | --- | --- | --- | --- | --- |
| `ADXL335BCPZ` | 3-Axis Accelerometer | 1 | 3.3V | 0.35 mA | 0.35 mA |
| `TLV9062` | Op-Amp Array (6 stages / 3 dual chips) | 3 | 5.0V | 6.30 mA | 18.90 mA |
| `PIC18F57Q43` Curiosity Nano | Microcontroller | 1 | 5.0V | 50.00 mA | 50.00 mA |

### Section B: Power Rails & 25% Safety Margin

| Power Rail | Components on Rail | Current Sum | +25% Safety Margin | Total Rail Current Needed |
| --- | --- | --- | --- | --- |
| **3.3V Rail** | `ADXL335BCPZ` (0.35 mA) | 0.35 mA | x 1.25 | **0.44 mA** |
| **5.0V Rail** | 3x `TLV9062` (18.90 mA) + `PIC18F57Q43` Nano (50.00 mA) | 68.90 mA | x 1.25 | **86.13 mA** |

### Section C: Voltage Regulator Selection

| Power Rail | Regulator / Source Choice | Input Voltage | Output Voltage | Max Current Rating | Regulator Idle Current | Total Current Needed | Remaining Current Available |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **3.3V Rail** | `LD1117V33` Regulator | 5.0V | 3.3V | 800 mA | 10.0 mA | 0.44 mA | **+799.56 mA** |
| **5.0V Rail** | 5V Supply from Curiosity Nano / Hub | 5.0V | 5.0V | 500 mA | 0 mA | 96.57 mA (86.13 + 0.44 + 10.0 mA) | **+403.43 mA** |

### Section D: External Power Source Check

| External Power Source | Connected Rails | Voltage | Max Source Current | Total System Current Needed | Remaining Current Available |
| --- | --- | --- | --- | --- | --- |
| **Main Hub / Curiosity Nano 5V Supply (`SWI12-9-N-P5` Wall Adapter)** | 5.0V Rail & 3.3V Regulator | 5.0V / 9.0V | 500 mA (Nano) / 1300 mA (Wall) | **96.57 mA** | **+403.43 mA** (Positive / Safe) |
| **9V Battery (`EN22` 500 mAh with 5V Regulator)** | 5.0V Rail & 3.3V Regulator | 9.0V | 500 mA | **104.57 mA** (includes 8 mA for 5V regulator) | **+395.43 mA** (Positive / Safe) |

### Section E: Battery Life Calculation

If running off a 500 mAh 9V battery with everything running at max current plus the 25% safety margin:

- **Battery Capacity:** 500 mAh
- **Total Max Current:** 104.57 mA
- **Worst-Case Battery Life:** `500 mAh / 104.57 mA` = **4.78 hours**


## Pin Table

| MCU Pin | Mode | Direction | Connected To | Description |
| --- | --- | --- | --- | --- |
| `RA0` | `ADC` | Input | X-Axis Peak Detector | Reads X-axis slip signal |
| `RA6` | `ADC` | Input | Y-Axis Peak Detector | Reads Y-axis slip signal |
| `RA5` | `ADC` | Input | Z-Axis Peak Detector | Reads Z-axis slip signal |
| `RA2` | `DAC` | Output | Connector 1 (Pin 7) | Sends analog slip signal to Main Hub (Board 1) |
| `RA1` | `Digital I/O` | Output | Connector 1 (Pin 1) | Digital feedback to Main Hub (Board 1) |
| `RC4` | `Digital I/O` | Input | Connector 1 (Pin 2) | Digital input from Main Hub (Board 1) |
| `RB1` | `Digital I/O` | Unused | None | Extra pin on block diagram |
| `RB2` | `Digital I/O` | Unused | None | Extra pin on block diagram |
| `RD1` | `Digital I/O` | Unused | None | Extra pin on block diagram |
| `RC7` | `Digital I/O` | Unused | None | Extra pin on block diagram |

> ## Component Selection Rationale Overview

To meet the requirements of creating this PCB Node, I selected the ADXL335BCPZ accelerometer, TLV9062 op-amps, and a 9V alkaline battery with a barrel jack connector. 
The ADXL335BCPZ beat out its competitors due to being the most reasonably priced and readily available. 
The TLV9062 was selected since it runs directly off the board's 5V rail. While a wall adapter is an option, having all five of our team members' boards plugged into the wall at the same time would be difficult to manage. 
Instead, a 9V battery with a barrel jack connector feeds into our onboard L7805 regulator to provide a steady 5V for the system while allowing mobility. 
From this 5V rail, a dedicated LD1117V33 voltage regulator steps the power down to safely supply 3.3V to the accelerometer.   