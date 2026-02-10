# Technical Reference Guide - ESP32-C6 Smart Doorbell

This document contains all technical specifications, pinouts, calculations, and reference information for the smart doorbell project.

## Table of Contents
1. [ESP32-C6 Specifications](#esp32-c6-specifications)
2. [Pin Configuration](#pin-configuration)
3. [Power Requirements](#power-requirements)
4. [Battery Specifications](#battery-specifications)
5. [Relay Module Specifications](#relay-module-specifications)
6. [Voltage Calculations](#voltage-calculations)
7. [Component Values Reference](#component-values-reference)
8. [ESPHome Configuration Reference](#esphome-configuration-reference)
9. [Home Assistant Entity Reference](#home-assistant-entity-reference)
10. [Troubleshooting Values](#troubleshooting-values)

---

## ESP32-C6 Specifications

### Microcontroller Details
- **Chip**: ESP32-C6 (Espressif Systems)
- **Core**: RISC-V 32-bit single-core processor
- **Clock Speed**: Up to 160 MHz
- **Flash**: 4MB (typical on DevKit)
- **SRAM**: 512 KB
- **WiFi**: 802.11b/g/n (2.4 GHz only)
- **Bluetooth**: BLE 5.0
- **Operating Voltage**: 3.0V - 3.6V
- **GPIO Count**: 22 programmable GPIOs
- **ADC**: 12-bit SAR ADC, 7 channels
- **PWM**: 6 channels
- **USB**: Built-in USB Serial/JTAG

### Important Character istics
- **Deep Sleep Current**: < 7 µA
- **Active WiFi Current**: ~80-120 mA (typical)
- **Modem Sleep**: ~15-20 mA
- **ADC Input Range**: 0 - 3.3V (with 11dB attenuation)
- **GPIO Max Current**: 40 mA per pin
- **Total GPIO Current**: 200 mA max combined

### Built-in Battery Charging
⚠️ **IMPORTANT**: The ESP32-C6 chip itself does NOT have a built-in battery charger.

Some confusion may arise from:
- ESP32-C6 DevKits that include a TP4056 on the board
- Other ESP32 boards (like some ESP32-S3 variants) that have charging circuits
- The USB-Serial functionality (which is not the same as battery charging)

**Recommendation**: Always use an external charging module (TP4056) for safety and proper battery management.

### Why Use TP4056 Instead of "Built-in" Charging
1. **Protection Circuit**: TP4056 modules include:
   - Overcharge protection (stops at 4.2V)
   - Over-discharge protection (cuts off at ~2.4V)
   - Short circuit protection
   - Thermal protection

2. **Proper Charge Algorithm**: 
   - Constant current / constant voltage (CC/CV) charging
   - Termination at proper current threshold
   - Trickle charge for deeply discharged cells

3. **Safety**:
   - Prevents LiPo fires from overcharging
   - Prevents battery damage from over-discharge
   - Temperature monitoring

4. **Cost**: TP4056 modules cost $1-2, excellent value for safety

**Conclusion**: ALWAYS use TP4056 or similar dedicated charging IC for LiPo batteries.

---

## Pin Configuration

### Assigned Pins (This Project)

| GPIO | Function | Direction | Details |
|------|----------|-----------|---------|
| GPIO4 | Button Input | Input | Pull-up enabled, active LOW |
| GPIO2 | Relay Control | Output | High = relay ON, Low = relay OFF |
| GPIO5 | Battery Voltage (ADC) | Input | Analog read, 0-3.3V range |
| GPIO6 | Status LED (optional) | Output | Future expansion |
| GPIO15 | Built-in LED | Output | Status indicator (board dependent) |

### Available for Future Expansion

| GPIO | Safe to Use? | Notes |
|------|--------------|-------|
| GPIO0 | ⚠️ Caution | Boot mode pin, use with pull-up |
| GPIO1 | ✅ Yes | General purpose |
| GPIO3 | ✅ Yes | General purpose |
| GPIO7 | ✅ Yes | Good for charging status sensor |
| GPIO10 | ✅ Yes | General purpose |
| GPIO11 | ✅ Yes | General purpose |
| GPIO18 | ❌ No | USB D- (if using USB) |
| GPIO19 | ❌ No | USB D+ (if using USB) |
| GPIO8 | ⚠️ Caution | Strapping pin |
| GPIO9 | ⚠️ Caution | Strapping pin |

### ADC Capable Pins
- GPIO0 (ADC1_CH0)
- GPIO1 (ADC1_CH1)
- GPIO2 (ADC1_CH2)
- GPIO3 (ADC1_CH3)
- GPIO4 (ADC1_CH4)
- **GPIO5 (ADC1_CH5)** ← Used for battery voltage
- GPIO6 (ADC1_CH6)

---

## Power Requirements

### Power Consumption Breakdown

| Component | Current Draw | Voltage | Notes |
|-----------|--------------|---------|-------|
| ESP32-C6 (WiFi active) | 80-120 mA | 3.3V | Peak during transmission |
| ESP32-C6 (idle) | 15-20 mA | 3.3V | WiFi connected |
| ESP32-C6 (deep sleep) | < 1 mA | 3.3V | WiFi disconnected |
| 5V Relay Module | 15-30 mA | 5V | When coil activated |
| Relay Coil (active) | 70-100 mA | 5V | During ringing |
| Status LED | 5-10 mA | 3.3V | If used |
| **Total (idle)** | **30-50 mA** | - | Typical operation |
| **Total (ringing)** | **180-250 mA** | - | Peak load |

### Power Supply Requirements

**USB Powered:**
- Minimum: 5V @ 500 mA (USB 2.0 standard)
- Recommended: 5V @ 1A (phone charger)
- Connector: USB-C or Micro-USB (depends on DevKit)

**Battery Powered:**
- LiPo voltage range: 3.0V - 4.2V
- Minimum capacity: 300 mAh (recycled vape)
- Recommended capacity: 500-2000 mAh
- Direct to 3.3V pin (no boost converter needed)

### Battery Runtime Calculations

**Formula**: Runtime (hours) = Battery Capacity (mAh) / Current Draw (mA)

| Battery | Always-On WiFi | WiFi Power Reduced | Deep Sleep (5 min wake) |
|---------|----------------|-------------------|-------------------------|
| 300 mAh | 6-10 hours | 12-15 hours | 3-5 days |
| 500 mAh | 10-17 hours | 20-25 hours | 5-8 days |
| 1000 mAh | 20-33 hours | 2-3 days | 1-2 weeks |
| 2000 mAh | 1.5-2.5 days | 4-6 days | 2-4 weeks |

*Actual runtime depends on WiFi signal strength, doorbell press frequency, and temperature*

---

## Battery Specifications

### LiPo Battery (Typical from Disposable Vape)

| Parameter | Value | Notes |
|-----------|-------|-------|
| Chemistry | LiPo (Lithium Polymer) | |
| Nominal Voltage | 3.7V | Average operating voltage |
| Full Charge | 4.2V | Maximum safe voltage |
| Empty (cutoff) | 3.0V | Minimum safe voltage |
| Capacity | 300-600 mAh | Varies by vape brand |
| Dimensions | Varies | Usually 10-15mm diameter |
| Weight | 5-10g | Typical |
| Charge Rate | 0.5C - 1C | Max 600 mA for 600mAh cell |
| Discharge Rate | 1C - 2C | Continuous |
| Cycle Life | 300-500 cycles | To 80% capacity |

### Voltage vs. State of Charge (Approximate)

| Voltage | Charge % | Battery State |
|---------|----------|---------------|
| 4.20V | 100% | Fully charged |
| 4.15V | 95% | Very high |
| 4.10V | 90% | High |
| 4.00V | 80% | Good |
| 3.90V | 70% | Good |
| 3.80V | 60% | Medium |
| 3.70V | 50% | Medium (nominal) |
| 3.60V | 40% | Getting low |
| 3.50V | 30% | Low |
| 3.40V | 20% | Very low |
| 3.30V | 10% | Critical - charge soon |
| 3.20V | 5% | Critical |
| 3.00V | 0% | Empty - cutoff |

**Note**: LiPo voltage curves are non-linear. The relationship above is approximate.

### TP4056 Charging Module Specifications

| Parameter | Value |
|-----------|-------|
| Input Voltage | 4.5V - 5.5V (USB) |
| Charge Voltage | 4.2V ± 1% |
| Charge Current | 1A (with 1.2kΩ Rprog) |
| Termination Current | 10% of charge current |
| Protection Voltage | 2.4V - 2.9V (under-voltage) |
| Protection Voltage | 4.25V - 4.35V (over-voltage) |
| Efficiency | ~85% |
| Operating Temp | 0°C to +85°C |

**LED Indicators:**
- Red LED: Charging in progress
- Blue/Green LED: Charging complete (or no battery)
- No LED: No power or fault

---

## Relay Module Specifications

### 5V Relay Module (Typical)

| Parameter | Value | Notes |
|-----------|-------|-------|
| Coil Voltage | 5V DC | Operating voltage |
| Coil Current | 70-100 mA | When energized |
| Control Input | 3.3V or 5V | Works with ESP32 |
| Input Current | 5-10 mA | Through optocoupler |
| Contact Rating | 10A @ 250VAC | Maximum switching |
| Contact Rating | 10A @ 30VDC | For our low voltage use |
| Contact Type | SPDT | Single Pole Double Throw |
| Switching Time | ~10 ms | On/Off time |
| Mechanical Life | 10,000,000 operations | Without load |
| Electrical Life | 100,000 operations | With rated load |
| Operating Temp | -30°C to +70°C | |

**Our Usage:**
- Switching: < 1A @ 5V DC (well within rating)
- Expected life: > 1,000,000 doorbell rings
- Signal isolation: Yes (optocoupler)

### 3.3V Relay Alternative (for Battery Use)

| Parameter | Value |
|-----------|-------|
| Coil Voltage | 3.3V DC |
| Coil Current | 60-90 mA |
| Everything else | Similar to 5V version |

**Advantage**: Can operate directly from LiPo battery voltage (3.0-4.2V)

---

## Voltage Calculations

### Voltage Divider for Battery Monitoring

**Circuit:**
```
Battery+ ──[R1=100kΩ]──┬──[R2=100kΩ]── GND
                       │
                    GPIO5
```

**Formulas:**
```
V_out = V_in × (R2 / (R1 + R2))

With R1 = R2 = 100kΩ:
V_GPIO5 = V_Battery × (100k / 200k)
V_GPIO5 = V_Battery × 0.5
V_GPIO5 = V_Battery / 2
```

**Voltage Table:**

| Battery Voltage | GPIO5 Reads | Charge % | Status |
|-----------------|-------------|----------|--------|
| 4.20V | 2.10V | 100% | Full |
| 4.10V | 2.05V | 90% | High |
| 4.00V | 2.00V | 80% | Good |
| 3.90V | 1.95V | 70% | Good |
| 3.80V | 1.90V | 60% | Medium |
| 3.70V | 1.85V | 50% | Medium |
| 3.60V | 1.80V | 40% | Getting Low |
| 3.50V | 1.75V | 30% | Low |
| 3.40V | 1.70V | 20% | Very Low |
| 3.30V | 1.65V | 10% | Critical |
| 3.00V | 1.50V | 0% | Empty |

**Why 100kΩ resistors?**
- High resistance = low current draw (~20 µA)
- Minimal battery drain
- Sufficient for ADC input impedance
- Common, inexpensive value

### ADC Attenuation Settings

ESP32-C6 ADC with different attenuation:

| Attenuation | Voltage Range | Use Case |
|-------------|---------------|----------|
| 0dB | 0 - 0.95V | Not suitable |
| 2.5dB | 0 - 1.34V | Not suitable |
| 6dB | 0 - 1.75V | Marginal |
| **11dB** | **0 - 3.3V** | **✓ Use this** |

**For our project**: 
- 11dB attenuation allows reading up to 3.3V
- GPIO5 will see max 2.1V (from 4.2V battery)
- Perfect fit with safety margin

---

## Component Values Reference

### Resistor Values and Purposes

| Value | Quantity | Purpose | Location |
|-------|----------|---------|----------|
| 10kΩ | 1 | Button pull-up (optional) | GPIO4 to 3.3V |
| 1kΩ | 1 | Relay control current limiting | GPIO2 to Relay IN |
| 330Ω | 1 | LED current limiting | GPIO6 to LED |
| 100kΩ | 2 | Battery voltage divider | Battery+ to GPIO5 to GND |
| 220Ω | 1 | MOSFET gate resistor (if MOSFET) | GPIO2 to MOSFET gate |
| 10kΩ | 1 | MOSFET pull-down (if MOSFET) | MOSFET gate to GND |

### Capacitor Values and Purposes

| Value | Quantity | Purpose | Location |
|-------|----------|---------|----------|
| 100nF | 1 | Button debouncing (optional) | GPIO4 to GND |
| 10µF | 1 | ADC stabilization (optional) | GPIO5 to GND |
| 100µF | 1 | Power supply smoothing (optional) | 5V to GND near ESP32 |

### Diode Specifications

| Type | Quantity | Purpose | Location |
|------|----------|---------|----------|
| 1N4007 | 1 | Relay flyback protection | Across relay coil |
| 1N4007 | 1 | MOSFET flyback (if used) | Across ringer |

**1N4007 Specs:**
- Max Forward Current: 1A
- Peak Reverse Voltage: 1000V
- Forward Voltage Drop: ~0.7V

---

## ESPHome Configuration Reference

### WiFi Power Settings

```yaml
wifi:
  output_power: 20dB    # Default, max range
  output_power: 8.5dB   # Reduced, saves power
```

**Power Consumption vs. TX Power:**
- 20dB (100mW): ~120 mA, best range
- 15dB (32mW): ~100 mA, good range
- 8.5dB (7mW): ~80 mA, reduced range

### Update Intervals

**Default (responsive):**
```yaml
update_interval: 30s   # Battery voltage
update_interval: 60s   # WiFi signal, uptime
```

**Power Saving (longer battery):**
```yaml
update_interval: 300s  # Battery voltage (5 min)
update_interval: 600s  # WiFi signal (10 min)
```

### Deep Sleep Configuration

**Not implemented by default** (doorbell needs to be always responsive)

For ultra-low power (advanced users):
```yaml
deep_sleep:
  run_duration: 30s      # Stay awake for 30s
  sleep_duration: 5min   # Sleep for 5 minutes
```

**Tradeoff**: Battery lasts weeks, but doorbell only checks every 5 minutes!

### ADC Configuration

```yaml
sensor:
  - platform: adc
    pin: GPIO5
    attenuation: 11dB           # 0-3.3V range
    update_interval: 30s        # How often to read
    filters:
      - multiply: 2.0           # Compensate for voltage divider
      - sliding_window_moving_average:
          window_size: 10       # Average over 10 samples
          send_every: 5         # Report every 5 averages
```

---

## Home Assistant Entity Reference

### Entities Created by ESPHome

**Binary Sensors:**
- `binary_sensor.doorbell_button` - Button press detection
- `binary_sensor.doorbell_status` - Online/offline status

**Switches:**
- `switch.doorbell_ringer` - Manual ringer control
- `switch.auto_ring_enabled` - Enable/disable auto-ring

**Sensors:**
- `sensor.doorbell_wifi_signal` - WiFi RSSI in dB
- `sensor.doorbell_uptime` - Device uptime in seconds
- `sensor.doorbell_press_count` - Count of button presses
- `sensor.battery_voltage` - Battery voltage in V
- `sensor.battery_level` - Battery percentage 0-100%

**Buttons:**
- `button.doorbell_restart` - Restart the device
- `button.test_ring` - Ring for 1 second test

**Text Sensors:**
- `text_sensor.doorbell_ip_address` - Device IP
- `text_sensor.doorbell_wifi_ssid` - Connected network
- `text_sensor.doorbell_esphome_version` - Firmware version

### Service Calls

**Ring doorbell with custom duration:**
```yaml
service: esphome.smart_doorbell_ring_doorbell
data:
  duration: 3000  # milliseconds
```

**Turn ringer on/off:**
```yaml
service: switch.turn_on
target:
  entity_id: switch.doorbell_ringer
```

**Enable/disable auto-ring:**
```yaml
service: switch.toggle
target:
  entity_id: switch.auto_ring_enabled
```

---

## Troubleshooting Values

### Expected Voltages (Multimeter Tests)

| Test Point | Expected | Issue if Different |
|------------|----------|-------------------|
| USB 5V to GND | 4.8 - 5.2V | Bad USB power |
| ESP32 3.3V to GND | 3.2 - 3.4V | Regulator issue |
| Battery+ to Battery- | 3.0 - 4.2V | Battery problem |
| GPIO4 (button up) | 3.3V | Pull-up not working |
| GPIO4 (button down) | 0V | Button or wiring issue |
| GPIO5 (battery monitor) | 1.5 - 2.1V | Voltage divider issue |
| GPIO2 (relay off) | 0V | Output stuck high |
| GPIO2 (relay on) | 3.3V | Output stuck low |
| Relay COM to NO (off) | Open | Relay stuck on |
| Relay COM to NO (on) | 0Ω | Relay not working |

### WiFi Signal Strength (dBm)

| RSSI | Quality | Status |
|------|---------|--------|
| -30 to -50 dBm | Excellent | Perfect |
| -50 to -60 dBm | Good | No issues |
| -60 to -70 dBm | Fair | Should work |
| -70 to -80 dBm | Poor | May disconnect |
| < -80 dBm | Very Poor | Unreliable |

### Current Draw Measurements

Measure with ammeter in series with power supply:

| Mode | Expected Current | Issue if Different |
|------|------------------|-------------------|
| Boot | 200-300 mA | Normal spike |
| WiFi connecting | 150-200 mA | Normal |
| Idle (WiFi on) | 30-50 mA | Check for issues if > 80 mA |
| Ringer active | 180-250 mA | Relay coil + ESP32 |
| Deep sleep | < 1 mA | Check if > 5 mA |

### Temperature Ranges

**Normal operating temperatures:**
- ESP32-C6: 30-50°C (86-122°F) under normal load
- TP4056 charging: 35-45°C while charging
- Battery: Should stay < 40°C

**If temperatures exceed:**
- ESP32 > 70°C: Reduce WiFi power or improve cooling
- TP4056 > 60°C: Check for short circuit
- Battery > 45°C: STOP USING - safety hazard!

---

## Quick Reference Tables

### GPIO Quick Reference (This Project)

| Pin | Function | Type | Value When |
|-----|----------|------|------------|
| GPIO4 | Button | Input | 3.3V (not pressed), 0V (pressed) |
| GPIO2 | Relay | Output | 3.3V (on), 0V (off) |
| GPIO5 | Battery ADC | Input | 1.5-2.1V (half battery voltage) |

### Component Quick Specs

| Component | Voltage | Current | Power |
|-----------|---------|---------|-------|
| ESP32-C6 | 3.3V | 80-120 mA | 0.26-0.40 W |
| Relay Module | 5V | 70-100 mA | 0.35-0.50 W |
| Status LED | 3.3V | 10 mA | 0.03 W |
| Total System | 3.3-5V | 180-250 mA | 0.64-1.25 W |

### Battery Chemistry Comparison

| Type | Voltage | Capacity | Cost | Safety | Recharge | Best For |
|------|---------|----------|------|--------|----------|----------|
| LiPo (vape) | 3.7V | 300-600mAh | Free | Medium | Yes | This project |
| 18650 Li-ion | 3.7V | 2000-3500mAh | $3-5 | Good | Yes | Long runtime |
| AAA NiMH × 3 | 3.6V | 800-1000mAh | $5-8 | Very Safe | Yes | Easy replacement |
| 9V Alkaline | 9V | 500mAh | $2 | Very Safe | No | With buck converter |

---

## Configuration Examples

### Minimal Configuration (USB Power Only)

```yaml
# No battery sensors
sensor:
  - platform: wifi_signal
    name: "WiFi Signal"
  - platform: uptime
    name: "Uptime"
```

### Full Configuration (Battery Powered)

```yaml
# Include all sensors
sensor:
  - platform: wifi_signal
    name: "WiFi Signal"
  - platform: uptime
    name: "Uptime"
  - platform: adc
    pin: GPIO5
    name: "Battery Voltage"
    # ... (battery monitoring)
  - platform: template
    name: "Battery Level"
    # ... (percentage calculation)
```

### Power Saving Configuration

```yaml
wifi:
  output_power: 8.5dB  # Reduce TX power

sensor:
  - platform: wifi_signal
    update_interval: 600s  # Update every 10 min
  - platform: adc
    update_interval: 120s  # Battery every 2 min
```

---

## Formulas and Calculations

### Battery Life Estimation

```
Runtime (hours) = Battery Capacity (mAh) / Average Current (mA)

Example with 500mAh battery:
- Always-on WiFi (50mA): 500 / 50 = 10 hours
- Power saving (30mA): 500 / 30 = 16.7 hours
- With sleep (5mA avg): 500 / 5 = 100 hours = 4.2 days
```

### Resistor Current Calculation

```
I = V / R

Voltage divider current draw:
I = 4.2V / (100kΩ + 100kΩ) = 4.2V / 200kΩ = 0.021 mA = 21 µA
```

### Relay Coil Power

```
P = V × I

5V Relay: P = 5V × 80mA = 0.4W
```

### LED Current Limiting Resistor

```
R = (V_supply - V_led) / I_led

For 3.3V supply, 2.0V LED, 10mA:
R = (3.3V - 2.0V) / 0.010A = 1.3V / 0.01A = 130Ω
Use 150Ω or 330Ω (standard values)
```

---

## Software Versions

**Project tested with:**
- ESPHome: 2024.x.x (any recent version)
- Home Assistant: 2024.x or later
- ESP-IDF Framework: Latest stable
- Python: 3.9 or later (for ESPHome)

---

## Standards and Safety

### Electrical Safety
- **Low Voltage**: All circuits operate at ≤5V DC
- **Current Limit**: ESP32 pins max 40 mA each
- **Isolation**: Relay provides isolation between control and load
- **Protection**: TP4056 includes overcharge, over-discharge, short circuit protection

### LiPo Battery Safety (per UN38.3)
- Store at 3.7-3.8V (40-60% charge) for long term
- Charge at 0.5C-1C rate maximum
- Never exceed 4.2V charging voltage
- Never discharge below 3.0V
- Operate between 0-45°C
- Store in fire-safe container

### EMC Considerations
- Keep antenna area clear (top of ESP32 board)
- Minimum 20mm from metal objects
- Use shielded USB cable if EMI issues
- Ferrite bead on power if noise issues

---

## Design Decisions Summary

**Why ESP32-C6?**
- Low cost (~$8)
- Built-in WiFi
- ESPHome support
- Low power modes
- Enough GPIOs

**Why TP4056 (not built-in charging)?**
- ESP32-C6 has NO built-in battery charging
- TP4056 provides comprehensive protection
- Standard, proven, safe solution
- Very low cost ($1-2)

**Why 100kΩ voltage divider?**
- Low current draw (21 µA)
- Standard resistor values
- Perfect ratio (divide by 2)
- Works with 11dB ADC attenuation

**Why GPIO4 for button?**
- Safe boot pin
- Has programmable pull-up
- No special boot functions
- ADC capable (future use)

**Why GPIO2 for relay?**
- Safe output pin
- No boot conflicts
- Can drive relay via 1kΩ resistor

**Why ESPHome (not Arduino)?**
- Zero coding required
- Automatic HA integration
- OTA updates built-in
- Extensive component library
- Active community

---

## Common Calculations Cheat Sheet

**Ohm's Law:**
- V = I × R
- I = V / R
- R = V / I

**Power:**
- P = V × I
- P = V² / R
- P = I² × R

**Battery Life:**
- Hours = Capacity(mAh) / Current(mA)
- Days = Hours / 24

**Voltage Divider:**
- V_out = V_in × (R2 / (R1 + R2))

**LED Resistor:**
- R = (V_supply - V_led) / I_desired

**Percentage from Voltage:**
- % = ((V_current - V_min) / (V_max - V_min)) × 100

---

## Pin Summary Diagram

```
ESP32-C6 DevKit Pinout (Used Pins)

                 ┌─────────────┐
                 │   ESP32-C6  │
                 │   DevKit    │
                 ├─────────────┤
    USB Power ───│ USB         │
                 │             │
     Button ─────│ GPIO4       │──── Input (pull-up)
                 │             │
     Relay ──────│ GPIO2       │──── Output (via 1kΩ)
                 │             │
   ADC (Batt)───│ GPIO5       │──── Analog In
                 │             │
   Status LED ──│ GPIO6       │──── Output (optional)
    (optional)  │             │
                 │ 3.3V    │──── To battery via switch
                 │ GND         │──── Common ground
                 └─────────────┘
```

---

This technical reference contains all the key values, calculations, and specifications for the ESP32-C6 Smart Doorbell project. Use it alongside the main documentation for detailed implementation guidance.
