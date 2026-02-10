# Battery Monitoring Guide - Disposable Vape LiPo

This guide shows how to use a LiPo battery from a disposable e-cigarette/vape with TP4056 charger and monitor battery level in Home Assistant.

## Why Recycle Disposable Vape Batteries?

Disposable vapes contain perfectly good LiPo batteries (typically 300-600mAh) that are wastefully discarded. Perfect for low-power IoT projects like this doorbell!

## Battery Specifications

**Typical Disposable Vape Battery:**
- Voltage: 3.7V nominal
- Capacity: 300-600mAh (varies by brand)
- Chemistry: LiPo (Lithium Polymer)
- Size: Small, cylindrical or rectangular
- Voltage Range: 3.0V (empty) to 4.2V (full charge)

**Expected Runtime:**
- 500mAh battery with this doorbell: ~24-48 hours continuous
- With deep sleep optimization: ~1-2 weeks
- Actual runtime depends on WiFi usage and doorbell presses

## Required Components

### Hardware
- [ ] LiPo battery from disposable vape (extracted safely)
- [ ] TP4056 charging module with protection
- [ ] 2× 100kΩ resistors (voltage divider)
- [ ] 1× 10µF capacitor (optional, for stability)
- [ ] Power switch (optional)
- [ ] Wires and connectors

### Safety Equipment
- [ ] Fire-safe container for charging
- [ ] Multimeter for testing
- [ ] Electrical tape or heat shrink
- [ ] Proper insulation for exposed connections

## Safety Warnings

⚠️ **CRITICAL SAFETY INFORMATION:**

1. **Extraction**: 
   - Discharge vape battery to ~3.5V before extraction
   - Never short circuit or puncture the battery
   - Work in well-ventilated area
   - Have fire extinguisher nearby

2. **Charging**:
   - Always use TP4056 with built-in protection
   - Never leave charging unattended
   - Charge on non-flammable surface
   - Stop using if battery swells or gets hot

3. **Voltage Limits**:
   - Never discharge below 3.0V (damages battery)
   - Never charge above 4.2V (fire risk)
   - TP4056 handles this automatically

4. **Physical Damage**:
   - Inspect for damage before use
   - Discard if puffy, dented, or leaking
   - Properly insulate all connections

## Circuit Diagram

### Complete Battery-Powered Setup with Monitoring

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  LiPo Battery (from disposable vape)                        │
│  ┌──────────────┐                                          │
│  │   ≈≈≈≈≈≈≈    │  3.7V, 300-600mAh                        │
│  │   (_ _)      │                                           │
│  └──┬────────┬──┘                                          │
│     │(+)  (-) │                                             │
│     │        │                                              │
│  ┌──┴────────┴──────────────┐                              │
│  │  TP4056 Charger Module   │ ← Micro USB (charging only)  │
│  │  ┌────┐   ┌────┐         │                              │
│  │  │ B+ │   │ B- │         │                              │
│  │  └─┬──┘   └─┬──┘         │                              │
│  │    │   CHG  │   STDBY    │ ← Status LEDs                │
│  │  ┌─┴──────┬─┴──┐         │                              │
│  │  │OUT+    │OUT-│         │                              │
│  └──┴─┬──────┴─┬──┴─────────┘                              │
│       │        │                                            │
│    [SWITCH]    │                                            │
│       │        │                                            │
│       │        │                                            │
│   ┌───┴────────┴────────────────────────────────┐          │
│   │                                              │          │
│   │     Voltage Divider (Battery Monitoring)    │          │
│   │                                              │          │
│   │        100kΩ                                 │          │
│   ├───────[R1]─────┬──────→ ESP32 GPIO5 (ADC)  │          │
│   │                │                             │          │
│   │              100kΩ                           │          │
│   │               [R2]                           │          │
│   │                │                             │          │
│   │               GND ←────────────────────────┐│          │
│   │                                            ││          │
│   │                                            ││          │
│   └──→ ESP32-C6 3.3V Pin                      ││          │
│                                                ││          │
│       ESP32-C6 GND ←───────────────────────────┘│          │
│                                                 │          │
└─────────────────────────────────────────────────┘          │
                                                             
Optional 10µF capacitor between GPIO5 and GND for stability
```

## Step-by-Step Setup

### Step 1: Extract Battery Safely

1. **Discharge the vape** to ~3.5V (use it normally or resistor discharge)
2. **Carefully open** the vape housing (usually clips or screws)
3. **Identify battery terminals** (red = positive, black = negative)
4. **Desolder or cut wires** carefully - avoid shorting!
5. **Test voltage** with multimeter: should be 3.0-4.2V
6. **Inspect battery** for any damage, swelling, or leaks
7. **Label polarity** with tape or marker

### Step 2: Connect to TP4056

```
Battery Polarity Check:
- Red wire or marked end = Positive (+)
- Black wire or unmarked = Negative (-)

TP4056 Connection:
Battery (+) → TP4056 B+ pad
Battery (-) → TP4056 B- pad

Secure with solder, connector, or terminal block
Add heat shrink or electrical tape for insulation
```

### Step 3: Add Voltage Divider for Monitoring

```
Materials:
- 2× 100kΩ resistors (1/4W or better)
- Small piece of perfboard or solder directly
- Wire to ESP32 GPIO5

Assembly:
1. Solder first 100kΩ resistor to TP4056 OUT+ (or switch output)
2. Solder second 100kΩ resistor to first resistor
3. Connect junction point to GPIO5
4. Connect end of second resistor to GND
5. Optional: Add 10µF capacitor between GPIO5 and GND

Visual:
TP4056 OUT+ ──[100kΩ]──┬──[100kΩ]── GND
                        │
                    GPIO5 ── ESP32
                        │
                   [10µF cap]
                        │
                       GND
```

### Step 4: Connect Power to ESP32

```
TP4056 OUT+ ─── Switch ─── ESP32 3.3V pin
TP4056 OUT- ────────────── ESP32 GND pin

Note: 
- Use 3.3V pin, NOT 5V pin
- 3.7V LiPo is compatible with ESP32 3.3V rail
- Use 3.3V relay module, not 5V
```

### Step 5: Configure ESPHome

The battery monitoring is already configured in [esphome-doorbell.yaml](esphome-doorbell.yaml):

**Key sections:**
```yaml
sensor:
  # Battery voltage via ADC
  - platform: adc
    pin: GPIO5
    name: "Battery Voltage"
    attenuation: 11dB
    filters:
      - multiply: 2.0  # Voltage divider compensation
    
  # Battery percentage
  - platform: template
    name: "Battery Level"
    lambda: |-
      float voltage = id(battery_voltage).state;
      float percentage = ((voltage - 3.0) / (4.2 - 3.0)) * 100.0;
      return percentage;
```

**If you don't have battery**, comment out the battery sensors or they'll show incorrect readings.

### Step 6: Calibrate ADC (Optional)

For better accuracy:

1. Measure actual battery voltage with multimeter
2. Compare to ESP32 reading in Home Assistant
3. Adjust calibration in YAML:
   ```yaml
   filters:
     - multiply: 2.0
     - offset: 0.0  # Adjust if needed (+ or -)
   ```

## Home Assistant Integration

### Entities Created

After flashing, you'll see:

**Battery Sensors:**
- `sensor.battery_voltage` - Raw battery voltage (V)
- `sensor.battery_level` - Battery percentage (%)

**Dashboard Card Example:**

```yaml
type: entities
title: Doorbell Battery
entities:
  - entity: sensor.battery_voltage
    name: Voltage
    icon: mdi:lightning-bolt
  - entity: sensor.battery_level
    name: Charge Level
  - type: attribute
    entity: sensor.battery_level
    attribute: state_class
show_header_toggle: false
```

### Battery Level Gauge

```yaml
type: gauge
entity: sensor.battery_level
name: Battery
min: 0
max: 100
severity:
  green: 50
  yellow: 20
  red: 0
```

### Low Battery Automation

```yaml
automation:
  - alias: "Doorbell Low Battery Alert"
    trigger:
      - platform: numeric_state
        entity_id: sensor.battery_level
        below: 20
    action:
      - service: notify.mobile_app_your_phone
        data:
          title: "⚠️ Low Battery"
          message: "Doorbell battery at {{ states('sensor.battery_level') }}%"
          data:
            priority: high
```

### Battery Charging Status

The TP4056 has LED indicators:
- **Red LED**: Charging
- **Blue/Green LED**: Fully charged
- **No LED**: Not charging or no power

You can add an optocoupler to monitor charging status on another GPIO if desired.

## Testing

### Voltage Tests

Use multimeter to verify:

| Test Point | Expected Voltage | Notes |
|------------|------------------|-------|
| TP4056 OUT+ to OUT- | 3.0-4.2V | Battery voltage |
| GPIO5 to GND | 1.5-2.1V | Half of battery voltage |
| ESP32 3.3V pin | 3.0-3.6V | From battery via switch |

### Software Tests

1. **Upload firmware** with battery monitoring enabled
2. **Check ESPHome logs**: Should see voltage readings
3. **Open Home Assistant**: Battery entities should appear
4. **Verify percentage**: 
   - 4.2V → ~100%
   - 3.7V → ~58%
   - 3.3V → ~25%
   - 3.0V → ~0%

### Load Test

1. Fully charge battery (4.2V)
2. Disconnect USB from TP4056
3. Turn on device
4. Monitor battery level in Home Assistant
5. Should slowly decrease over hours
6. Test doorbell functionality at various charge levels

## Battery Life Optimization

### Current Consumption

**Active WiFi mode**: ~80-120mA
**Sleep mode**: ~10-20mA (still connected)
**Deep sleep**: <1mA (disconnected)

### Improving Runtime

1. **Reduce WiFi transmit power** (in YAML):
   ```yaml
   wifi:
     output_power: 8.5dB  # Default is 20dB
   ```

2. **Increase update intervals**:
   ```yaml
   sensor:
     - platform: wifi_signal
       update_interval: 300s  # Instead of 60s
   ```

3. **Implement deep sleep** (advanced):
   ```yaml
   deep_sleep:
     run_duration: 30s
     sleep_duration: 5min
   ```
   Note: Deep sleep disconnects WiFi, so doorbell won't be instantly responsive

### Expected Battery Life

| Battery | Mode | Expected Runtime |
|---------|------|------------------|
| 300mAh | Always-on WiFi | 24-36 hours |
| 500mAh | Always-on WiFi | 2-3 days |
| 500mAh | WiFi power reduced | 3-5 days |
| 500mAh | Deep sleep (5min) | 1-2 weeks |
| 2000mAh (18650) | Always-on WiFi | 1-2 weeks |

## Troubleshooting

### Battery Not Charging

- Check USB cable and adapter (5V needed)
- Verify battery connections to TP4056
- Check TP4056 protection circuit not triggered
- Test without battery load

### Wrong Voltage Readings

- Verify voltage divider resistors (both 100kΩ)
- Check GPIO5 connection
- Calibrate ADC in YAML
- Test with multimeter

### Battery Drains Too Fast

- Check for high WiFi transmission power
- Reduce sensor update rates
- Verify no short circuits
- Check relay not stuck on
- Monitor for excessive doorbell presses

### Home Assistant Shows Wrong Percentage

- Adjust voltage range in template sensor
- LiPo range should be 3.0V-4.2V
- Calibrate using multimeter readings
- Update lambda calculation if needed

### Battery Gets Hot

- **STOP USING IMMEDIATELY**
- Disconnect battery
- Check for short circuits
- Verify TP4056 current rating
- Battery may be damaged - replace it

## Wiring Checklist

Before first power-on:

- [ ] Battery polarity correct (+ to B+, - to B-)
- [ ] TP4056 OUT+ goes through switch to ESP32 3.3V
- [ ] TP4056 OUT- goes to ESP32 GND
- [ ] Voltage divider: 100kΩ from OUT+ to junction
- [ ] Voltage divider: 100kΩ from junction to GND
- [ ] Junction point connected to GPIO5
- [ ] Optional 10µF cap between GPIO5 and GND
- [ ] All connections insulated and secure
- [ ] No exposed battery terminals
- [ ] Switch accessible for power control

## Charging

### Initial Charge

1. Connect Micro USB to TP4056
2. Red LED should illuminate (charging)
3. Leave for 1-3 hours depending on battery size
4. Blue/Green LED indicates full charge (4.2V)
5. Can use device while charging (like a phone)

### Regular Charging

- Charge when battery drops below 20% (3.2V)
- Don't let it fully discharge to 0%
- Occasional full charge is good (0% to 100%)
- Avoid charging to 100% every time for longer lifespan

### Charging Safety

- Charge in fire-safe location
- Don't charge overnight unattended (first few times)
- Monitor battery temperature (should stay cool)
- Stop if battery swells or gets hot
- Use quality USB adapter (min 500mA)

## Enclosure Considerations

With LiPo battery:

1. **Ventilation**: Add small holes for air circulation
2. **Access**: Easy access to USB charging port
3. **Switch**: External power switch for convenience  
4. **Mounting**: Secure battery so it doesn't move
5. **Insulation**: Ensure no conductive surfaces touch battery
6. **Viewing**: Transparent window to see TP4056 LEDs (optional)

## Advanced: Add Charging Status Sensor

To monitor if battery is charging:

### Hardware Addition
```
TP4056 CHRG pin ──[10kΩ]── GPIO7
                     │
                    GND
```

### ESPHome Addition
```yaml
binary_sensor:
  - platform: gpio
    pin: 
      number: GPIO7
      inverted: true
    name: "Battery Charging"
    device_class: battery_charging
```

This will show in Home Assistant whether battery is currently charging.

## Disposal

When battery reaches end of life:

1. **Discharge** to ~3.0V
2. **Tape terminals** to prevent shorts
3. **Bring to e-waste recycling**
4. **Never throw in regular trash**
5. **Many electronics stores accept batteries**

## Summary

Using a disposable vape battery with TP4056 and battery monitoring:

✅ **Pros:**
- Recycles waste batteries
- Free or very cheap battery source
- Works great for low-power IoT
- Full Home Assistant integration
- Monitor battery level remotely

⚠️ **Cons:**
- Smaller capacity (300-600mAh vs 2000mAh for 18650)
- Requires safe extraction and handling
- Shorter runtime between charges
- More frequent charging needed

💡 **Best for:**
- Projects near charging location
- Learning battery management
- Environmental recycling
- Short-term portable operation

For longer runtime, consider an 18650 battery (2000-3500mAh) with the same TP4056 setup!

---

**Safety First!** If you're uncomfortable working with LiPo batteries, stick with USB power or use a pre-made battery pack.
