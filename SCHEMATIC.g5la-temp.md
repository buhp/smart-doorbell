# Circuit Schematic and Wiring Diagrams

This document provides detailed schematics for the ESP32-C6 Smart Doorbell system.

## System Block Diagram

```
                    ┌─────────────────────────────────┐
                    │      ESP32-C6 DevKit            │
                    │                                 │
 Doorbell Button    │  GPIO4 (Input)   GPIO2 (Output)│    Doorbell Ringer
     [Button]───────┼─────┐                  ┌───────┼────────[Relay]────┐
        │           │     │                  │       │                   │
        ╘═══════════╪═ GND│                  │       │              ┌────┴────┐
                    │                        │       │              │  Ringer │
      10kΩ Pull-up  │                        │       │              │  4.5V   │
         to 3.3V ───┼─────┘                  │       │              └────┬────┘
                    │                        │       │                   │
                    │                   [1kΩ]│       │                   │
                    │                        │       │                   │
                    │    5V/GND    USB/Battery│       │                   │
                    │      │ │         │      │       │                   │
                    └──────┼─┼─────────┼──────┘       │                   │
                           │ │         │              │                   │
                           │ │      ┌──┴──┐           │                   │
                           │ │      │Power│           │                   │
                           │ │      │ 5V  │           │                   │
                           │ │      └─────┘           │                   │
                           │ └──────────────────GND───┼───────────────────┘
                           └────────────────────5V────┘
```

## Detailed Schematic - Option 1: Relay Control

### Pin Connections

| Component | ESP32-C6 Pin | Notes |
|-----------|--------------|-------|
| Doorbell Button | GPIO4 | Input with internal pull-up |
| Doorbell Button GND | GND | Common ground |
| Relay Driver Base | GPIO2 | Through 1k-2.2k resistor |
| Relay Coil + | 5V | From USB power (G5LA-1 5VDC) |
| Relay Coil - | Transistor collector | BC547B low-side switch |
| Flyback Diode | Across relay coil | 1N4148 or 1N400x, band to 5V |
| Status LED (optional) | GPIO6 | Through 330Ω resistor |

### Button Input Circuit

```
                     3.3V (or use internal pull-up)
                      │
                    [10kΩ]
                      │
    Button     ┌──────┴──────┐
  [Contact]────┤ GPIO4       │
      │        │  ESP32-C6   │
     GND       └─────────────┘

Optional Debounce:
                     3.3V
                      │
                    [10kΩ]
                      │
    Button     ┌──────┴──────┬────[100nF]────GND
  [Contact]────┤ GPIO4       │
      │        │  ESP32-C6   │
     GND       └─────────────┘
```

### Relay Output Circuit (G5LA-1 5VDC + BC547B)

```
   ESP32-C6                                      Relay Coil (G5LA-1 5VDC)

   GPIO2 ──[1k-2.2k]──┬── Base
                     │
                   [BC547B]
                     │ C
   GND ──────────────┴ E
                            +5V (USB)
                               │
                               ├───────+|───┐
                               │   Flyback  │
                               │   Diode    │
                               │  1N4148/   │
                               │  1N400x    │
                               │            │
                            [Relay Coil]   │
                               │            │
                               └────────────┘

   Relay Contacts (dry contact to ringer)
   COM ──────────────── Power + (4.5V)
   NO  ──────────────── Ringer +
   Ringer - ─────────── Power - (4.5V)

Note: The relay coil must be powered from 5V USB. ESP32 GND and 5V GND must be common.
```

### Complete Wiring - USB Powered with Relay

```
┌─────────────────────────────────────────────────────────────────────┐
│  ESP32-C6 DevKit                                                    │
│                                                                     │
│  [USB-C] ──── 5V Power Supply                                      │
│                                                                     │
│  GPIO4 ─────────┬──────────────[10kΩ]──────── 3.3V                │
│                 │                                                   │
│                 └──────────────[Button]──────── GND                │
│                                                                     │
│  GPIO2 ─────────[1k-2.2k]── BC547B Base                           │
│                                                                     │
│  5V ──────────────────────── Relay Coil (+)                        │
│                                                                     │
│  GND ─────────┬──────────── BC547B Emitter                          │
│               │                                                     │
│               └──────────── Button Common                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

    Relay Contacts (G5LA-1)                 Doorbell Ringer

    COM ──────── Power (+) 4.5V
    NO  ──────── Ringer (+)
    Ringer (-) ─ Power (-) 4.5V

    Flyback diode across relay coil (not across ringer)
```

## Detailed Schematic - Option 2: MOSFET Control (Advanced)

### MOSFET Output Circuit

```
   ESP32-C6                    MOSFET Circuit                Ringer

   GPIO2 ────[220Ω]──── Gate                        (+) ─────┐
                              │                              │
                        [IRLZ44N]                            │  [Ringer]
                        N-MOSFET                             │   4.5V
                              │                              │
                         Drain├───── 5V (or 4.5V)           │
   GND ─────[10kΩ]──── Source├───── GND                    │
                              │                              │
                              └──────────────────────────────┘

Note: Add flyback diode (1N4007) across ringer if it's an inductive load
```

### MOSFET Circuit with Flyback Protection

```
                         5V or 4.5V
                              │
                    ┌─────────┴─────────┐
                    │                   │
                 [Ringer]          [Flyback]
                  Coil              Diode
                    │              1N4007
                    │            (Cathode to +)
   ESP32           ┌┴┐              │
   GPIO2 ──[220Ω]──│G│              │
                   │ │ IRLZ44N      │
   5V ─────────────│D│              │
                   └┬┘              │
   GND ─[10kΩ]─────│S├──────────────┘
                   └─┴───────── GND
```

## Power Supply Options

### Option A: USB Powered (Recommended)

```
   [USB 5V Adapter]
         │
    ┌────┴─────────────────┐
    │  [USB-C/Micro-USB]   │
    │   ESP32-C6 DevKit    │
    │                      │
    │  5V Pin ──────────── Relay VCC
    │  GND Pin ─────────── Common GND
    └──────────────────────┘
```

### Option B: Battery Powered

```
   [18650 Battery 3.7V]
         │
    ┌────┴─────────────┐
    │  TP4056 Module   │
    │  (Charging)      │
    │  OUT+ ──┬────────┼──── Switch ──┬──── Boost Converter ──── 5V out
    │  OUT- ──┼────────┼──────────────┤                           │
    └─────────┘        │              │                           │
                       │              └──────────── GND ──────────┤
                       │                                          │
                       └──────────────────────────────────────────┘
                                 To ESP32-C6 + Relay

Optional: If your relay can work at 3.7V, you can skip the boost converter
```

### Battery-Powered Simplified (No Boost)

```
   [LiPo Battery 3.7V]  (e.g., from disposable vape)
         │
    ┌────┴─────────────┐           Many relays work
    │  TP4056 Module   │           at 3.3V-5V range
    │  (Charging)      │
    │  OUT+ ──── Switch ──── ESP32-C6 3.3V/5V Pin
    │  OUT- ──────────────── GND
    └──────────────────┘

Note: ESP32-C6 can run from 3.0V-3.6V on 3V3 pin
      Some 5V relays may not trigger reliably at 3.7V
      Use a 3.3V relay module for battery operation
      LiPo from disposable vapes typically 300-600mAh
```

### Battery Voltage Monitoring Circuit

To monitor battery level in Home Assistant, add a voltage divider:

```
   Battery+  (TP4056 OUT+)
      │
    [100kΩ]  R1
      │
      ├────────── ESP32 GPIO5 (ADC)
      │
    [100kΩ]  R2
      │
     GND

Voltage at GPIO5 = Battery Voltage × (R2 / (R1 + R2))
With equal resistors: GPIO5 = Battery Voltage / 2

LiPo voltage range: 3.0V (empty) to 4.2V (full)
GPIO5 will see: 1.5V (empty) to 2.1V (full)

ESP32-C6 ADC safe range: 0-3.3V ✓
```

### Complete Battery-Powered Circuit with Monitoring

```
  ┌─────────────────────────────────────────────────┐
  │                                                 │
  │  LiPo Battery (3.7V, from disposable vape)      │
  │  ┌────────────┐                                 │
  │  │ + ════ -   │  300-600mAh typical              │
  │  └──┬──────┬──┘                                 │
  │     │      │                                     │
  │  ┌──┴──────┴────────┐                           │
  │  │  TP4056 Charger   │                           │
  │  │  ┌──┐  ┌──┐      │ ← Micro USB (charging)    │
  │  │  │B+│  │B-│       │                           │
  │  └───┬──────┬────────┘                           │
  │      │      │                                     │
  │   OUT+    OUT-                                   │
  │      │      │                                     │
  │      ├──────┼────── 100kΩ ──┬── GPIO5 (ADC)      │
  │      │      │                │                    │
  │      │      │          100kΩ│                    │
  │      │      │                │                    │
  │      │      └────────────────┴── GND             │
  │      │                                            │
  │   Switch                                          │
  │      │                                            │
  │      ├─────────────────────────── ESP32 3V3 Pin  │
  │      │                                            │
  │      └─────────────────────────── Relay VCC      │
  │                                    (if 3.3V)     │
  └────────────────────────────────────────────────────┘
```

## GPIO Pin Selection Rationale

| GPIO | Function | Reason |
|------|----------|--------|
| GPIO4 | Button Input | Safe boot pin, good for input |
| GPIO2 | Relay/MOSFET Control | Safe output pin, no boot conflicts |
| GPIO6 | Status LED (optional) | Safe output pin |

**Avoid these ESP32-C6 pins:**
- GPIO8, GPIO9: Strapping pins
- GPIO15: UART/Boot mode
- GPIO18, GPIO19: USB pins

## Protection Recommendations

1. **Input Protection**: The button input is protected by the internal pull-up resistor
2. **Output Protection**:
   - Relay modules typically include optocoupler isolation
   - MOSFET circuits should include a gate resistor (220Ω) and pull-down (10kΩ)
3. **Ringer Protection**:
   - Add flyback diode (1N4007) if ringer is inductive (coil-based)
   - Cathode to positive side, anode to ground
4. **Power Protection**:
   - Use quality USB power adapter (min 1A)
   - Battery systems should use TP4056 with built-in protection

## Testing Points

Before final assembly, verify these voltages:

| Test Point | Expected Voltage | Notes |
|------------|------------------|-------|
| ESP32 5V pin | 4.8-5.2V | When powered via USB |
| ESP32 3.3V pin | 3.2-3.4V | Internal regulator output |
| GPIO4 (button not pressed) | 3.3V | Pull-up active |
| GPIO4 (button pressed) | 0V | Connected to GND |
| GPIO2 (relay off) | 0V | Output low |
| GPIO2 (relay on) | 3.3V | Output high |
| Relay COM-NO (relay off) | Open | No continuity |
| Relay COM-NO (relay on) | Closed | Continuity |

## Salvaged Relay Test Circuit (Before Wiring to Doorbell)

Use this quick test to verify a salvaged G5LA-1 5VDC relay clicks and switches.

```
   5V (USB) ────────────────┐
                            │
                         [Relay Coil]
                            │
                            ├───────────────┐
                            │               │
                         [Flyback]         [LED + 1k]
                          Diode              (test load)
                       1N4148/1N400x          │
                            │               COM
                            │                │
                            └────── NO ──────┘
                                    │
                                   GND

   Manual trigger (coil): briefly connect coil (-) to GND
```

Expected:
- Relay clicks when coil is grounded.
- LED turns on when relay clicks (COM-NO closes).

If no click: confirm 5V supply, coil continuity, and diode orientation (band to 5V).

## Next Steps

Proceed to [ASSEMBLY.md](ASSEMBLY.md) for step-by-step wiring instructions.
