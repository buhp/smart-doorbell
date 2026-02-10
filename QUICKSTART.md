# Quick Start Guide

Get your ESP32-C6 Smart Doorbell running in 30 minutes!

## What You'll Build

A smart doorbell that:
- ✅ Detects button presses
- ✅ Controls your existing doorbell ringer
- ✅ Sends notifications to your phone via Home Assistant
- ✅ Allows automation (e.g., disable at night)

## Shopping List (Minimum Required)

1. **ESP32-C6 DevKit** (~$8)
2. **5V Relay Module** (~$2)
3. **USB Power Adapter** 5V/1A (~$3)
4. **Jumper Wires** pack (~$3)
5. **10 resistors** (10kΩ, 1kΩ) (~$1)

**Total: ~$17**

Your existing doorbell button and ringer will work!

## Build Steps (Overview)

### 1. Buy Parts (30 min)
- Order from Amazon or local electronics store
- See [PARTS_LIST.md](PARTS_LIST.md) for details

### 2. Test ESP32 (15 min)
- Plug into computer
- Install ESPHome: `pip install esphome`
- Verify it powers on

### 3. Wire It Up (45 min)
```
Button → GPIO4 & GND
Relay Control → GPIO2
Relay Power → 5V & GND
Ringer → Relay output
```
See [SCHEMATIC.md](SCHEMATIC.md) for diagrams

### 4. Upload Software (20 min)
- Edit secrets.yaml with your WiFi
- Upload esphome-doorbell.yaml
- Device appears in Home Assistant automatically!

### 5. Test & Enjoy (10 min)
- Press button → notification
- Toggle ringer from Home Assistant
- Add automations

## Detailed Guides

1. **Parts**: [PARTS_LIST.md](PARTS_LIST.md)
2. **Wiring**: [SCHEMATIC.md](SCHEMATIC.md)
3. **Assembly**: [ASSEMBLY.md](ASSEMBLY.md)
4. **Software**: [esphome-doorbell.yaml](esphome-doorbell.yaml)
5. **Automations**: [homeassistant-automation.yaml](homeassistant-automation.yaml)

## Minimal Wiring Diagram

```
┌──────────────────┐
│  ESP32-C6        │
│                  │
│  GPIO4 ──────────┼─── Doorbell Button ─── GND
│                  │
│  GPIO2 ──[1kΩ]───┼─── Relay IN
│  5V ─────────────┼─── Relay VCC
│  GND ────────────┼─── Relay GND
│                  │
└──────────────────┘

Relay Module:
  COM ── (+) Power 5V
  NO ─── (+) Ringer
  Ringer (-) ── GND
```

## Power Options

**USB Powered (easiest)**: Just plug ESP32 into phone charger
**Battery Powered**: Add 18650 battery + TP4056 charging module

## First Test

After uploading firmware:

1. Open Home Assistant
2. Go to Settings → Devices
3. You'll see "ESPHome: smart-doorbell"
4. Click Configure
5. Press your doorbell button
6. You get a notification! 🎉

## Popular Automations

### 1. Disable at Night
```yaml
- Turn off "Auto Ring Enabled" at 10 PM
- Turn on "Auto Ring Enabled" at 7 AM
```

### 2. Baby Mode
```yaml
- When "Baby Sleeping" mode ON → Disable ringer
- Still get phone notification
```

### 3. Flash Lights
```yaml
- Press doorbell → Flash living room lights
- Great for hard of hearing
```

See [homeassistant-automation.yaml](homeassistant-automation.yaml) for complete examples!

## Troubleshooting Quick Fixes

| Problem | Quick Fix |
|---------|-----------|
| Won't connect to WiFi | Check secrets.yaml WiFi name/password |
| Button not detected | Verify GPIO4 wire connection |
| Ringer won't ring | Check relay wiring, verify 5V power |
| Not in Home Assistant | Check both on same WiFi network |

## Next Steps

1. Start with [PARTS_LIST.md](PARTS_LIST.md) to order components
2. While waiting for parts, read [ASSEMBLY.md](ASSEMBLY.md)
3. When parts arrive, follow [ASSEMBLY.md](ASSEMBLY.md) step-by-step
4. Enjoy your smart doorbell!

## Need Help?

- Full assembly guide: [ASSEMBLY.md](ASSEMBLY.md)
- Wiring diagrams: [SCHEMATIC.md](SCHEMATIC.md)
- ESPHome docs: https://esphome.io
- Home Assistant forum: https://community.home-assistant.io

## Safety Note

⚠️ This project uses low voltage (5V DC) only. If your existing doorbell uses AC power or high voltage, DO NOT use this project without proper isolation. Consult an electrician.

---

**Ready to start?** Head to [PARTS_LIST.md](PARTS_LIST.md) to see what to buy!
