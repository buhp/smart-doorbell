# Parts List (Bill of Materials)

## Required Components

### Main Components

| Component | Quantity | Description | Example/Notes |
|-----------|----------|-------------|---------------|
| ESP32-C6 DevKit | 1 | Main microcontroller | ESP32-C6-DevKitC-1 or similar |
| 5V Relay Module | 1 | For controlling ringer circuit | 5V single channel relay (optocoupler isolated) |
| Doorbell Ringer | 1 | Your existing ringer | 4.5V buzzer/ringer |
| Doorbell Button | 1 | Your existing button | Normally open pushbutton |

### Electronic Components

| Component | Quantity | Value/Type | Purpose |
|-----------|----------|------------|---------|
| Resistor | 1 | 10kΩ | Pull-up for button input |
| Resistor | 1 | 1kΩ | Current limiting for relay control |
| Resistor | 1 | 330Ω | LED current limiting (optional) |
| LED | 1 | Any color | Status indicator (optional) |
| Diode | 1 | 1N4007 | Flyback protection for relay (if not built-in) |
| Capacitor | 1 | 100nF ceramic | Button debouncing (optional) |

### Power Supply Options

Choose **ONE** of the following:

#### Option A: USB Powered (Recommended)
| Component | Quantity | Description |
|-----------|----------|-------------|
| 5V USB Power Adapter | 1 | Min 1A output (phone charger) |
| Micro-USB or USB-C Cable | 1 | Depending on your ESP32-C6 board |

#### Option B: Battery Powered
| Component | Quantity | Description |
|-----------|----------|-------------|
| 18650 Li-ion Battery | 1 | 3.7V, min 2000mAh |
| TP4056 Charging Module | 1 | With protection circuit |
| Switch | 1 | Power on/off switch |
| Battery Holder | 1 | For 18650 battery |
| Boost Converter | 1 | 3.7V to 5V (optional, if relay needs 5V stable) |

### Connection Materials

| Component | Quantity | Description |
|-----------|----------|-------------|
| Jumper Wires | 10-15 | Male-to-female, various lengths |
| Wire | 2-5m | 22-24 AWG stranded wire |
| Terminal Blocks | 2-3 | 2-position screw terminals |
| Breadboard or Perfboard | 1 | For prototyping/permanent build |
| Enclosure | 1 | Plastic project box (optional) |
| Heat Shrink Tubing | Various | For insulation |

## Alternative Components

### Alternative to Relay: MOSFET Control

If you prefer a solid-state solution instead of a relay:

| Component | Quantity | Value/Type | Purpose |
|-----------|----------|------------|---------|
| N-Channel MOSFET | 1 | IRLZ44N or similar | Switching ringer circuit |
| Resistor | 1 | 10kΩ | Pull-down for MOSFET gate |
| Resistor | 1 | 220Ω | Gate current limiting |

**Note**: MOSFET option is quieter, has faster switching, and longer lifetime than relay.

## Where to Buy

- **ESP32-C6**: Espressif official store, Amazon, AliExpress
- **Relay Module**: Amazon, eBay, AliExpress, local electronics store
- **Resistors/Capacitors/Diodes**: Electronics component stores, Amazon kits
- **Wire & Connectors**: Hardware store, electronics store
- **Power Supply**: Phone charger (USB), electronics store

## Cost Estimate

- **Basic Build (USB powered)**: ~$15-25 USD
- **Battery Powered Build**: ~$25-40 USD
- **Enclosure & Extras**: ~$5-15 USD

**Total Project Cost**: $20-55 USD depending on options chosen
