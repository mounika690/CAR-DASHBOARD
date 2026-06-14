# Car Black Box System using STM32 & CAN Bus

A real-time Car Black Box system built using STM32F429ZI6 microcontroller
and CAN Bus communication protocol. The system monitors and logs critical
vehicle parameters like RPM, Speed, Gear Position, Indicator, Temperature,
and Timestamp across multiple ECU nodes over a CAN Bus network.

## Project Overview

This project simulates an automotive black box system where multiple ECUs
communicate over a CAN Bus network. ECU2 and ECU3 transmit vehicle data,
and ECU1 acts as the central receiver that decodes each CAN message by
its unique CAN ID and displays all parameters via UART serial monitor.

## Project Structure

STM32-CAN-Bus-Car-BlackBox/
│
├── CAN_PRO_ECU1/          # Central Receiver - Decodes and logs all CAN data via UART
├── CAN_PRO_ECU2/          # Transmitter - Speed, Gear, Temperature
└── CAN_PRO_ECU3/          # Transmitter - RPM, Timestamp, Indicator

## ECU Roles

| ECU  | Role                       | Parameters                        | CAN IDs              |
|------|----------------------------|-----------------------------------|----------------------|
| ECU1 | Receiver / BlackBox Logger | Receives and displays all data    | -                    |
| ECU2 | Transmitter                | Speed (ADC), Gear (GPIO), Temp (ADC) | 0x106, 0x107, 0x108 |
| ECU3 | Transmitter                | RPM (ADC), Time (RTC), Indicator (GPIO) | 0x103, 0x104, 0x105 |

## CAN Message Format

| CAN ID | Parameter   | Format         | Sender | Source         |
|--------|-------------|----------------|--------|----------------|
| 0x103  | Engine RPM  | 4-digit value  | ECU3   | ADC1 Channel 0 |
| 0x104  | Timestamp   | HHMMSS         | ECU3   | RTC Module     |
| 0x105  | Indicator   | NT/LT/RT/HZ    | ECU3   | GPIO PA5/PA6/PA7 |
| 0x106  | Vehicle Speed | 4-digit value | ECU2   | ADC1 Channel 0 |
| 0x107  | Gear Position | G1-G4/ON/GR/GN/CO | ECU2 | GPIO PA5/PA6/PA7 |
| 0x108  | Temperature | 4-digit value (°C) | ECU2 | ADC1 Channel 1 |

## Indicator States (ECU3)

| State   | Value Sent | Button Pin |
|---------|------------|------------|
| Neutral | NT         | None pressed |
| Left Turn | LT       | PA5        |
| Right Turn | RT      | PA6        |
| Hazard  | HZ         | PA7        |

## Gear States (ECU2)

| State   | Value Sent | Description  |
|---------|------------|--------------|
| ON      | ON         | Engine ON    |
| GR      | GR         | Reverse      |
| GN      | GN         | Neutral      |
| G1      | G1         | Gear 1       |
| G2      | G2         | Gear 2       |
| G3      | G3         | Gear 3       |
| G4      | G4         | Gear 4       |
| CO      | CO         | Clutch Override |

## Technologies Used

- Microcontroller  : STM32F429ZI (ARM Cortex-M4)
- IDE              : STM32CubeIDE
- Language         : Embedded C
- Protocol         : CAN Bus (Controller Area Network)
- Peripherals      : CAN1, ADC1, USART1, RTC, GPIO
- HAL Library      : STM32 HAL Drivers
- CAN Transceiver  : MCP2551 / TJA1050
- Baud Rate        : 500 Kbps
- UART Baud Rate   : 115200

## Hardware Requirements

- 3x STM32F429ZI Nucleo Boards
- 3x MCP2551 / TJA1050 CAN Transceiver Module
- 2x Potentiometer (RPM and Speed simulation via ADC)
- 1x Temperature Sensor / Potentiometer (Temperature simulation via ADC)
- 6x Push Buttons (Gear Up / Gear Down / Clutch on ECU2, Left / Right / Hazard on ECU3)
- 2x 120Ω Termination Resistors (at both ends of CAN bus)
- Jumper Wires

## Hardware Connections

CAN Bus Wiring:
ECU1 CAN_H ──── ECU2 CAN_H ──── ECU3 CAN_H
ECU1 CAN_L ──── ECU2 CAN_L ──── ECU3 CAN_L
[120Ω]                            [120Ω]

ECU2 Pin Connections:
| Pin      | Function                        |
|----------|---------------------------------|
| PA0      | Speed Potentiometer (ADC CH0)   |
| PA1      | Temperature Sensor (ADC CH1)    |
| PA5      | Gear Up Button                  |
| PA6      | Gear Down Button                |
| PA7      | Clutch Button                   |

ECU3 Pin Connections:
| Pin      | Function                        |
|----------|---------------------------------|
| PA0      | RPM Potentiometer (ADC CH0)     |
| PA5      | Left Indicator Button           |
| PA6      | Right Indicator Button          |
| PA7      | Hazard Button                   |
| RTC      | Timestamp Generation            |

## How to Run

1. Clone this repository
   git clone https://github.com/yourusername/STM32-CAN-Bus-Car-BlackBox.git

2. Open STM32CubeIDE

3. Import all 3 projects
   File → Open Projects from File System → Select each ECU folder

4. Build each project
   Right click project → Build Project

5. Flash each board via ST-Link
   Right click → Run As → STM32 Application
   Flash ECU1 first, then ECU2, then ECU3

6. Open Serial Monitor on ECU1 at 115200 baud to view live data

## Sample Output (ECU1 UART Monitor)

RPM  : 3200
TIME : 151045
IND  : LT
SPD  : 0065
GEAR : G2
TEMP : 0085

## Important Notes

- All 3 ECUs must share common GND
- CAN baud rate must be same across all ECUs
- 120Ω termination resistors required at both ends of CAN bus
- Flash ECU1 first before ECU2 and ECU3
- Each ECU requires its own ST-Link for flashing
- TEMP value range is 0 to 150 degree C mapped from 12-bit ADC
- RPM value is calculated as (ADC / 40.95) x 60
- Speed value range is 0 to 99 kmph mapped from 12-bit ADC
