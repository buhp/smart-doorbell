# ESP32-C6 Smart Doorbell System

This project converts your existing doorbell into a smart doorbell integrated with Home Assistant, allowing you to:
- Receive notifications when someone presses the doorbell button
- Control when the ringer can actually ring through Home Assistant automations
- Monitor doorbell activity remotely

## Overview

The system consists of:
1. **Input**: Binary sensor detecting doorbell button press (normally open contact)
2. **Output**: Electronically controlled switch for the doorbell ringer circuit
3. **Integration**: ESPHome-based Home Assistant integration
4. **Power**: 5V USB or battery powered

## Features

- **Button Detection**: Detects doorbell button presses via GPIO input
- **Ring Control**: Controls doorbell ringer via relay/MOSFET output
- **Home Assistant Integration**: Automatic discovery via ESPHome
- **Automation Ready**: Trigger ringer only when permitted (e.g., disable at night, when baby sleeping, etc.)
- **Notifications**: Receive instant notifications when doorbell is pressed

## Quick Start

1. See [PARTS_LIST.md](PARTS_LIST.md) for required components
2. Review [SCHEMATIC.md](SCHEMATIC.md) for wiring diagrams
3. Follow [ASSEMBLY.md](ASSEMBLY.md) for step-by-step assembly
4. Upload [esphome-doorbell.yaml](esphome-doorbell.yaml) configuration
5. Add Home Assistant automation from [homeassistant-automation.yaml](homeassistant-automation.yaml)

## Documentation

- [Parts List](PARTS_LIST.md) - All required components
- [Schematic](SCHEMATIC.md) - Circuit diagrams and connections
- [Assembly Guide](ASSEMBLY.md) - Step-by-step wiring instructions
- [ESPHome Configuration](esphome-doorbell.yaml) - Firmware configuration
- [Home Assistant Setup](homeassistant-automation.yaml) - Automation examples

## Safety Notes

⚠️ **Important Safety Information:**
- This project uses low voltage (5V max)
- Ensure all connections are properly insulated
- Do not connect to high voltage doorbell systems without proper isolation
- If your existing doorbell uses AC voltage, consult an electrician

## Technical Specs

- **Microcontroller**: ESP32-C6 DevKit
- **Input Voltage**: 5V (USB) or 3.7V LiPo battery with charging circuit
- **Button Input**: Digital GPIO with pull-up resistor
- **Ringer Output**: Relay or MOSFET controlled
- **WiFi**: 2.4GHz (ESP32-C6)
- **Integration**: ESPHome + Home Assistant
