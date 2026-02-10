# Assembly and Installation Guide

Step-by-step instructions for building your ESP32-C6 Smart Doorbell system.

## Prerequisites

- [ ] All parts from [PARTS_LIST.md](PARTS_LIST.md) gathered
- [ ] Basic soldering skills (if using perfboard)
- [ ] Wire strippers and cutters
- [ ] Multimeter for testing
- [ ] Computer with ESPHome installed
- [ ] Home Assistant instance running

## Safety First

⚠️ **Safety Warnings:**
- Always disconnect power before making connections
- Double-check polarity when connecting power
- Ensure all exposed connections are insulated
- Do not exceed voltage ratings
- Your existing doorbell should be low voltage (under 12V DC)

## Step-by-Step Assembly

### Step 1: Prepare Your Workspace

1. Clear a clean, well-lit workspace
2. Organize all components
3. Have your schematic ([SCHEMATIC.md](SCHEMATIC.md)) ready for reference
4. Label wires as you work to avoid confusion

### Step 2: Test ESP32-C6 Board

Before assembling, verify your ESP32-C6 is working:

1. Connect ESP32-C6 to computer via USB
2. Open ESPHome Dashboard (see Software Setup below)
3. Create a basic test configuration
4. Upload and verify it connects
5. **Only proceed if this works!**

### Step 3: Prepare the Button Input Circuit

#### Option A: Using Breadboard (for testing)

1. Insert ESP32-C6 into breadboard
2. Connect button to GPIO4:
   ```
   ESP32 GPIO4 ─── Button ─── GND
   ```
3. No external resistor needed (using internal pull-up)

#### Option B: Using Perfboard (permanent installation)

1. Plan component layout on perfboard
2. Solder ESP32-C6 headers to perfboard
3. Solder button wires:
   - One wire to GPIO4
   - Other wire to GND
4. Optional: Add 10kΩ pull-up resistor between GPIO4 and 3.3V
5. Optional: Add 100nF capacitor between GPIO4 and GND for debouncing

**Testing:**
- Power the ESP32
- Measure voltage at GPIO4: Should read ~3.3V
- Press button: Voltage should drop to 0V
- If this works, proceed!

### Step 4: Prepare the Relay/Ringer Circuit

#### Option A: Using 5V Relay Module (Recommended for beginners)

1. **Identify relay pins:**
   - VCC (power input)
   - GND (ground)
   - IN (signal input)
   - COM (common contact)
   - NO (normally open contact)
   - NC (normally closed contact - not used)

2. **Connect relay control:**
   ```
   ESP32 GPIO2 ─── 1kΩ resistor ─── Relay IN pin
   ESP32 5V ──────────────────────── Relay VCC
   ESP32 GND ─────────────────────── Relay GND
   ```

3. **Connect ringer to relay output:**
   ```
   Power (+) 5V ───── Relay COM
   Relay NO ─────── Ringer (+)
   Ringer (-) ────── GND
   ```

**Testing:**
- Power ESP32
- Measure voltage at IN pin: Should be 0V (relay off)
- You should NOT hear relay clicking yet
- Verify relay does not conduct between COM and NO (use multimeter continuity test)

#### Option B: Using MOSFET (Advanced, quieter operation)

1. **Prepare MOSFET circuit on breadboard/perfboard:**
   ```
   Components needed:
   - IRLZ44N N-channel MOSFET
   - 220Ω resistor (gate)
   - 10kΩ resistor (pull-down)
   - 1N4007 diode (flyback protection)
   ```

2. **Connect MOSFET:**
   ```
   ESP32 GPIO2 ─── 220Ω ─── MOSFET Gate
   MOSFET Gate ─── 10kΩ ─── GND (pull-down)
   MOSFET Drain ──────────── Ringer (+)
   MOSFET Source ─────────── GND
   Power 5V ─────────────────── Ringer (+) via diode
   ```

3. **Add flyback diode:**
   - Cathode (stripe) to Ringer (+)
   - Anode to GND

**Testing:**
- Measure Gate voltage: Should be 0V (MOSFET off)
- Measure Drain-Source: Should be open circuit
- Do NOT connect ringer yet for initial test

### Step 5: Connect Power Supply

#### Option A: USB Powered (Simplest)

1. Connect USB cable to ESP32-C6
2. Connect USB to 5V power adapter
3. Adapter should provide at least 1A current
4. That's it! USB provides both 5V and GND

#### Option B: Battery Powered

1. **Setup TP4056 charging module:**
   - Solder battery wires to OUT+ and OUT-
   - Add power switch in series with OUT+
   
2. **Optional boost converter (if using 5V relay):**
   ```
   Battery (+) ─── Switch ─── Boost IN+
   Battery (-) ──────────────── Boost IN-
   Boost OUT+ ────────────────── ESP32 5V
   Boost OUT- ────────────────── ESP32 GND
   Set boost output to 5.0V using potentiometer
   ```

3. **Without boost (using 3.3V relay):**
   ```
   Battery (+) ─── Switch ─── ESP32 3.3V pin
   Battery (-) ──────────────── ESP32 GND
   ```

**Testing:**
- Measure voltage at ESP32 5V/3.3V pin
- Should read 4.8-5.2V (USB) or 3.2-3.6V (battery)
- LED on ESP32 should light up

### Step 6: Wire Your Existing Doorbell Components

1. **Disconnect existing battery from ringer**
   - Note polarity markings on ringer
   - Keep the old battery holder as backup

2. **Connect ringer to your circuit:**
   ```
   For Relay:
   Ringer (+) ─── Relay NO
   Ringer (-) ─── GND
   Relay COM ─── Power (+) 5V or 4.5V
   
   For MOSFET:
   Ringer (+) ─── Power (+) 5V or 4.5V
   Ringer (-) ─── MOSFET Drain
   MOSFET Source ─── GND
   ```

3. **Connect doorbell button:**
   - One button wire to GPIO4
   - Other button wire to GND
   - Button likely has 2 wires coming from outside
   - Polarity doesn't matter for the button

**Testing:**
- Do NOT test ringer yet (wait for software)
- Verify button wiring with multimeter continuity test
- Button pressed = continuity between wires
- Button released = no continuity

### Step 7: Complete Wiring Checklist

Before powering on, verify these connections:

- [ ] ESP32 USB/Battery connected to power
- [ ] ESP32 GND connected to common ground
- [ ] Doorbell button wire 1 → GPIO4
- [ ] Doorbell button wire 2 → GND
- [ ] Relay/MOSFET control → GPIO2
- [ ] Relay/MOSFET power → 5V or 3.3V
- [ ] Relay/MOSFET ground → GND
- [ ] Ringer connected through relay/MOSFET
- [ ] All connections secure and insulated
- [ ] No shorts between power and ground

### Step 8: Software Setup

#### Install ESPHome

1. **On Windows:**
   ```powershell
   # Install Python if not already installed
   # Then install ESPHome
   pip install esphome
   ```

2. **Start ESPHome Dashboard:**
   ```powershell
   esphome dashboard config/
   ```
   This opens a web interface at http://localhost:6052

#### Configure Your Device

1. In ESPHome Dashboard, click **+ New Device**
2. Name it: `smart-doorbell`
3. Select device type: **ESP32-C6**
4. Copy the contents of [esphome-doorbell.yaml](esphome-doorbell.yaml) into the editor

5. **Create secrets file:**
   - Copy [secrets.yaml.example](secrets.yaml.example) to `secrets.yaml`
   - Edit with your WiFi credentials:
     ```yaml
     wifi_ssid: "YourWiFiName"
     wifi_password: "YourWiFiPassword"
     api_encryption_key: ""  # Will be auto-generated
     ota_password: "changeme123"
     web_username: "admin"
     web_password: "admin123"
     ```

6. **Install firmware:**
   - Click **Install** on your device
   - Choose **Plug into this computer**
   - Select the COM port for your ESP32-C6
   - Wait for upload to complete (3-5 minutes first time)

#### Verify Software

1. After upload, check the logs
2. You should see:
   ```
   [I][app:102] ESPHome version 2024.x.x compiled on ...
   [I][wifi:543] WiFi connected!
   [I][app:129] Smart Doorbell System Started
   ```
3. Device should appear in Home Assistant automatically

### Step 9: Testing

#### Test 1: Button Detection

1. Open ESPHome web interface: http://`device-ip-address`
2. Watch the logs
3. Press your doorbell button
4. You should see: `"Doorbell button pressed!"`
5. Check Home Assistant - `binary_sensor.doorbell_button` should turn ON

**If not working:**
- Check GPIO4 connection
- Verify button is connected correctly
- Test button with multimeter

#### Test 2: Ringer Control

1. In Home Assistant, go to Developer Tools → Services
2. Call service: `switch.turn_on`
   - Entity: `switch.doorbell_ringer`
3. **RINGER SHOULD RING!**
4. Ringer should auto-off after 10 seconds
5. Try `switch.turn_off` to stop it early

**If not working:**
- Check GPIO2 connection to relay IN
- Verify relay power connections (VCC, GND)
- Check ringer connections to relay COM/NO
- Test relay manually by connecting IN to VCC briefly

#### Test 3: Auto-Ring Feature

1. Ensure `switch.auto_ring_enabled` is ON in Home Assistant
2. Press physical doorbell button
3. Ringer should activate for 2 seconds automatically
4. You should receive notification (if automation setup)

**If not working:**
- Check that auto_ring_enabled switch is ON
- Review ESPHome logs for errors
- Verify button press is detected first (Test 1)
- Verify ringer can be controlled (Test 2)

### Step 10: Final Installation

1. **Secure components:**
   - Place ESP32 and relay in project enclosure
   - Drill holes for wire entry/exit
   - Use cable glands or rubber grommets
   - Leave USB port accessible for updates

2. **Mount enclosure:**
   - Near doorbell ringer for short wires
   - Near power outlet if USB powered
   - Protected from weather
   - Good WiFi signal location

3. **Weatherproofing doorbell button:**
   - If button is outside, ensure weatherproof enclosure
   - Use waterproof connectors
   - Apply silicone sealant if needed

4. **Cable management:**
   - Use cable ties to organize wires
   - Keep power wires separate from signal wires
   - Label all connections

5. **Final checkout:**
   - Test button press → notification
   - Test button press → ringer activation
   - Test manual ringer control from Home Assistant
   - Verify WiFi signal strength
   - Check power consumption

### Step 11: Home Assistant Setup

1. **Add device:**
   - Go to Settings → Devices & Services
   - ESPHome integration should auto-discover
   - Click Configure
   - Enter API encryption key from secrets.yaml

2. **Add automations:**
   - Copy examples from [homeassistant-automation.yaml](homeassistant-automation.yaml)
   - Customize to your needs
   - Test each automation

3. **Create dashboard:**
   - Add entities card
   - Add button card for quick control
   - Add history graph for doorbell activity

## Troubleshooting

### ESP32 won't connect to WiFi
- Double-check WiFi credentials in secrets.yaml
- Ensure 2.4GHz WiFi (ESP32-C6 doesn't support 5GHz)
- Check WiFi signal strength at installation location
- Try fallback AP mode: connect to "Smart-Doorbell Fallback" WiFi

### Button not detected
- Verify GPIO4 connection
- Check if pull-up is enabled in code
- Test button with multimeter
- Try swapping button wires
- Check for loose connections

### Ringer not working
- Verify relay clicks when activated (you should hear it)
- Check relay COM/NO connections
- Measure voltage at relay output when activated
- Verify ringer voltage requirements (should be 3-6V)
- Try different power source for ringer
- Check for blown fuse in ringer (if it has one)

### Relay keeps clicking/cycling
- Check power supply current capacity (need >500mA)
- Verify GPIO2 settings in code
- Check for loose connections
- May need larger power supply

### Home Assistant not discovering device
- Check both devices on same network
- Verify API key matches
- Check firewall settings
- Try manual integration via IP address
- Check ESPHome logs for connection attempts

### Battery drains too fast
- Reduce WiFi transmission power in ESPHome config
- Increase update intervals for sensors
- Use deep sleep mode (requires code changes)
- Check for excessive logging
- Verify no short circuits

### Web interface not accessible
- Verify device IP address (check router or ESPHome dashboard)
- Check firewall settings
- Ensure web_server is enabled in YAML
- Try http://smart-doorbell.local instead of IP

## Maintenance

### Regular Checks (Monthly)
- [ ] Test doorbell button functionality
- [ ] Verify ringer operation
- [ ] Check battery level (if battery powered)
- [ ] Review WiFi signal strength
- [ ] Update ESPHome firmware if available

### Battery Replacement (if applicable)
- Replace 18650 battery every 1-2 years
- Check for battery swelling
- Clean battery contacts
- Verify charging circuit still working

### Relay Replacement
- Relays have limited lifetime (~100,000 cycles)
- If clicking sounds weak, consider replacement
- MOSFET alternative has much longer life

## Enhancements

Consider these upgrades:

1. **Add status LED**: Visual indicator on GPIO6
2. **Add buzzer**: Audio confirmation of button press
3. **Add display**: OLED screen showing status
4. **Add camera**: ESP32-CAM for video doorbell
5. **Add PIR sensor**: Motion detection
6. **Add battery monitor**: Track battery percentage
7. **Add temperature sensor**: Environmental monitoring

## Support

If you encounter issues:
1. Check ESPHome logs first
2. Review Home Assistant logs
3. Test each component individually
4. Verify wiring against schematic
5. Consult ESPHome documentation: https://esphome.io
6. Home Assistant forum: https://community.home-assistant.io

## Success!

Once everything is working:
- Document your specific wiring for future reference
- Take photos of your build
- Share your success with the Home Assistant community!
- Customize automations to your lifestyle
- Enjoy your smart doorbell! 🎉
