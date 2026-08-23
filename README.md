# SMD-Avionics-PCB
# Sounding Rocket Avionics System

A complete avionics package for high-altitude sounding rockets, featuring dual redundancy in barometric sensing, real-time GPS/LoRa telemetry, and programmable pyro firing/servo control.

## Overview

This project implements a two-board avionics system for sounding rocket applications:

- **Flight Controller Board**: RP2350B-based main computer with integrated sensors and communication systems
- **Power Distribution Board**: Regulated power delivery with current monitoring and fault protection

The system is capable of logging sensor data to onboard storage, transmitting telemetry via LoRa, determining apogee via dual barometers, and firing pyro charges or deploying parachutes via servo control.

<img width="899" height="743" alt="Screenshot 2026-08-05 011245" src="https://github.com/user-attachments/assets/b02e081f-5210-4011-b059-667d20a6cea5" />


---

##  Hardware Architecture

### Flight Controller Board 

**Main Processor**
- Raspberry Pi RP2350B (QFN-80, dual-core ARM Cortex M33)
- 32 MHz crystal oscillator with load capacitors
- QSPI flash storage (W25Q128JVSIQ - 128Mb NOR flash)

**Inertial Measurement Unit (IMU)**
- LSM6DSO32 6-axis accelerometer + gyroscope (SPI interface)
- 3-axis acceleration and rotation rate measurement
- Critical for apogee detection via vertical acceleration

**Barometric Sensors (Dual Redundancy)**
- Primary: BMP390 (SPI interface) - 300-1100 hPa range
- Secondary: MS5611 (I2C interface) - High altitude capable
- Cross-checked for data integrity and fallback capability

**GNSS/Positioning**
- L89HAS90 multi-constellation receiver
  - GPS, GLONASS, BeiDou, Galileo, IRNSS, QZSS support
  - 3D fix output, UTC time sync
  - External antenna via UFL connector

**Telemetry & Long-Range Communication**
- SX1262 LoRa module (868 MHz ISM band)
- SPI interface to MCU
- Real-time data transmission during flight

**Pyro & Servo Control**
- 4x pyro firing channels (continuity-enabled)
  - TLP281-4 optocouplers for isolation
  - IRLR7843 MOSFETs for power switching
  - continuity-check LEDs
- 2x servo PWM output channels
- 10kΩ pull-up resistors on signal lines

**Onboard Storage**
- SD card socket (QSPI quad mode capable)
- Data logging to microSD for black-box recording
- Shared SPI bus with accelerometer

**Interface & Debug**
- USB Type-C connector (CX90M-16P) for programming
- SWD debug header (SM03B-SRSS)
- Active piezo buzzer for audible feedback (GPIO control)
- 2x ARM debug LEDs (APT2012MGC, R0603 current limiting)
- 3x test points for oscilloscope probing (TP1-TP3)

**I2C & SPI Buses**
- I2C0: INA260 current monitor, Accelerometer(10kΩ pullups)
- SPI0: Primary barometer
- SPI1: SD card, MS5611
- QSPI: Flash memory

---

### Power Distribution Board (Main_FC_power_circuitary)

**Primary Buck Converter**
- TPS5450DDAR 5A adjustable regulator
  - Input: +BATT (Li-Po battery recommended)
  - Output: +5V at up to 5A
  - PA4343.153NLT inductor
  - Soft-start to limit inrush current

**Secondary Buck Converter**
- TLV62569PDDCR high-efficiency step-down
  - Input: +5V from primary converter
  - Output: +3.3V or other regulated voltage
  - Optimized for efficency.

**Current Monitoring**
- INA260AIPWR integrated power monitor
  - Measures input current and voltage in real-time
  - I2C interface to flight controller
  - Essential for battery health monitoring and power budgeting
  - Programmable alerts via ALERT pin

**Headers & Distribution**
- Multiple headers for connecting payloads
- +5V rail
- +5High rail
- +3.3V rail  
- Ground distribution (critical for low-impedance GND return)
- XT60 connector for external power
- Debug LEDs (3x) indicating power rail status

---

##  Interface Pinouts & Connectors

### Flight Controller - Key Signals

**GPS/GNSS (L89HAS90)**
- UART: RXD, TXD (async serial 9600 baud typical)
- 1PPS output for timing sync
- External antenna (RP-SMA via UFL)

**LoRa (SX1262)**
- SPI: MOSI, MISO, SCK, CS
- Control: DIO1, DIO2, RST, BUSY
- GPIO: TXEN, RXEN (optional Tx/Rx control)

**Servo Outputs**
- 2x 3-pin JST-PH headers (servo 1, servo 2)
  - Pin 1: +5V
  - Pin 2: PWM signal
  - Pin 3: GND

**Pyro Channels**
- 4x WAGO 2-pin push connectors (pyro 1-4 shared across 2 connectors)
- Electrical continuity check via onboard LEDs
- Fires on MCU GPIO command

**Debug/Serial**
- USB Type-C for programming & serial console
- SWD header (SWCLK, SWD, GND, 3.3V)

**Power Input**
- XT60 main battery connector
- +BATT and GND distribution rails between the two boards
  
**JST VH**
- Connect Power Board to Flight Computer Board
- +5, +3.3, +5H Power Rails, and data from INA Current Sensor is sent over.

---

##  Electrical Specifications

| Parameter | Value | Notes |
|-----------|-------|-------|
| **Battery Input** | 3S-4S Li-Po (11.1V - 14.8V nominal) | Connector: XT60 |
| **Board to board connectors** | 10A max per pin | Connector: JST VH |
| **+5V Rail** | 5A max @ 5V | Primary buck converter |
| **+3.3V Rail** | 2A max @ 3.3V | Secondary buck converter |
| **+1.1V Rail** | Internal only | For Rp2350 internal circuitary |
| **Temperature Range** | -40°C to +85°C | Component rated range |
| **SD Card Voltage** | 3.3V | Shared with MCU I/O |
| **LoRa Frequency** | 868 MHz ISM | Region-dependent; requires license compliance |
| **USB Power** | 500 mA max | For programming only; battery required for flight |

---

##  Features & Capabilities

### Sensor Capabilities
-  Dual redundant barometric altitude measurement  
-  6-axis IMU (acceleration + rotation rates)  
-  Multi-constellation GNSS positioning  
-  Real-time telemetry downlink (LoRa)  
-  Onboard data logging to SD card  

### Control & Firing
-  4 programmable pyro firing channels  
-  2 servo control outputs (PWM)  
-  Continuity monitoring on each pyro channel  
-  Optocoupler isolation for pyro firing  

### Power Management
-  Real-time current monitoring  
-  Multiple independent regulated rails  
-  Battery voltage monitoring  

### Interface & Diagnostics
-  USB programming & serial console  
-  SWD JTAG debug port  
-  Onboard status LEDs  
-  Piezo buzzer feedback  
-  Test points for scope probing  

---

##  Bill of Materials Summary

### Flight Controller Board
- **Microcontroller**: 1x RP2350B QFN-80
- **Sensors**: 2x barometers, 1x IMU, 1x GPS, 1x LoRa module
- **Power**: 2x buck converters, 20+ filtering capacitors
- **Discrete**: 30+ resistors, 10x diodes, 2x MOSFETs, 2x optocouplers
- **Connectors**: USB-C, SWD header, servo headers, pyro WAGO connectors, SD card socket
- **Total Components**: ~150 parts

### Power Distribution Board
- **Regulators**: TPS5450, TLV62569, INA260 current monitor
- **Input Protection**: Schottky diodes, XT60 connector
- **Debug**: 3x status LEDs with current-limiting resistors
- **Connectors**: Battery input, XT60 main output, JST VH
- **Total Components**: ~40 parts

Full BOM available in PDF schematics (see `/BOM` section).

---

##  Component Datasheets

Key datasheets for reference:
- **RP2350**: https://datasheets.raspberrypi.com/rp2350/rp2350-datasheet.pdf
- **BMP390**: https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bmp390-ds002.pdf
- **MS5611**: https://www.te.com/commerce/DocumentDelivery/DDEController?action=showdoc&DocId=Data%20Sheet%7CMS5611-01BA03%7CB3%7Fpdf
- **LSM6DSO32**: https://www.st.com/content/st_com/en/products/motion-sensors/inemo-inertial-measurement-units/lsm6dso32.html
- **SX1262**: https://semtech.my.salesforce.com/sfc/p/#E0000000JelW/a/44000000MDqq/7r.gfK0v3yjqOhMzMxT06MbkAXVSNqg_OeJWJmU1uxQ
- **L89HAS90**: https://www.u-blox.com/en/product/l89-high-precision-gnss-module
