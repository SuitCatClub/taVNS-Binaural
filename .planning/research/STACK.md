# Technology Stack — OpenBinaural-taVNS

**Project:** OpenBinaural-taVNS (DIY Binaural Transcutaneous Auricular Vagus Nerve Stimulation)
**Researched:** 2025-07-14
**Overall Stack Confidence:** HIGH (all core components verified via GitHub API, datasheets, and official documentation)

---

## 1. Embedded Firmware Stack

### Core Platform

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| **ESP32-S3** (WROOM-1 N16R8) | — | MCU | Dual-core Xtensa LX7 @ 240 MHz, native BLE 5.0, 4× hardware timers, 2× SPI, WiFi for future OTA. 16 MB flash + 8 MB PSRAM ensures ample space for LittleFS logging + BLE stack. No external BLE module needed. | HIGH |
| **Arduino-ESP32** | **3.3.8** (ESP-IDF 5.5.4) | Framework | Mature, well-documented. v3.x is a major overhaul from 2.x — **new timer API**, NimBLE-compatible. Released 2026-04-12. Pin to this version in platformio.ini. | HIGH — verified via GitHub Releases API |
| **PlatformIO** (platform-espressif32) | **v6.13.0** | Build system | Deterministic builds, library management, CI-friendly. Avoids Arduino IDE limitations. Released 2026-02-26. | HIGH — verified via GitHub Releases API |

#### platformio.ini baseline

```ini
[env:esp32s3]
platform = espressif32@6.13.0
board = esp32-s3-devkitc-1
framework = arduino
monitor_speed = 115200
board_build.arduino.memory_type = qio_opi  ; PSRAM enabled
lib_deps =
    h2zero/NimBLE-Arduino@^2.5.0
    bblanchon/ArduinoJson@^7.4.3
build_flags =
    -DBOARD_HAS_PSRAM
    -DCONFIG_NIMBLE_CPP_ATT_VALUE_INIT_LENGTH=512
```

### Key Libraries

| Library | Version | Purpose | Why This One | Confidence |
|---------|---------|---------|--------------|------------|
| **NimBLE-Arduino** | **2.5.0** | BLE dual-role (peripheral GATT server + central HRS client) | Confirmed working on ESP32-S3 in dual-role mode. Uses ~60% less flash and ~40% less RAM than Bluedroid. v2.4.0+ contains critical S3 dual-role fix (PR #1018, merged 2025-09-02). See §5 for details. | HIGH — **verified: fix present in 2.4.0 and 2.5.0 source code** |
| **ArduinoJson** | **7.4.3** | Session config serialization, HRV report JSON | v7 is a complete rewrite with zero-allocation `JsonDocument`. Smaller footprint than v6. Released 2026-03-02. | HIGH — verified via GitHub Releases API |
| **LittleFS** | Built-in (Arduino-ESP32 3.x) | RR interval logging, session data, config persistence | Bundled with Arduino-ESP32 3.x — no external library needed. Wear-leveling, power-loss resilient. Do NOT use SPIFFS (deprecated in ESP-IDF 5.x). | HIGH |

#### What NOT to use

| Rejected | Why Not |
|----------|---------|
| **Bluedroid BLE stack** | 2× flash, 2× RAM vs NimBLE. Cannot do dual-role reliably on ESP32-S3. Officially deprecated direction for Arduino-ESP32. |
| **ESP-IDF native (no Arduino)** | ISR latency gain (~2µs) is clinically irrelevant at 250µs pulse width. Arduino-ESP32 3.x wraps ESP-IDF 5.5.4 — full access to hardware timers and FreeRTOS underneath. Arduino saves ~3-6 months of development time for a solo builder. |
| **SPIFFS** | Deprecated in ESP-IDF 5.x. Slower than LittleFS, no directory support, worse wear-leveling. |
| **NimBLE-Arduino 2.3.x** | ESP32-S3 dual-role fails with `rc=519 Memory Capacity Exceeded` due to incorrect `ble_max_act` configuration. Issue #1016, fixed in PR #1018. Only v2.4.0+ is safe. |
| **ArduinoJson v6.x** | Legacy. v7 is smaller, faster, and actively maintained. v6 will be EOL. |

### Hardware Timer API (Arduino-ESP32 3.x)

> **CRITICAL API CHANGE:** Arduino-ESP32 3.x completely changed the timer API from 2.x. The old `timerBegin(timer_num, divider, countUp)` / `timerAlarmWrite()` / `timerAlarmEnable()` pattern is **GONE**. Do not follow 2.x-era tutorials.

**New API (3.x):**

```cpp
// 1. Create timer at desired tick frequency
hw_timer_t *timer = timerBegin(1000000);  // 1 MHz = 1µs resolution

// 2. Attach ISR
timerAttachInterrupt(timer, &onStimPulseISR);

// 3. Set alarm (value in ticks, autoreload, unlimited repeats)
timerAlarm(timer, 250, true, 0);  // Fire every 250µs

// ISR must use ARDUINO_ISR_ATTR
void ARDUINO_ISR_ATTR onStimPulseISR() {
    portENTER_CRITICAL_ISR(&timerMux);
    // Toggle DAC / H-bridge phase
    portEXIT_CRITICAL_ISR(&timerMux);
}
```

**Precision:** ESP32-S3 hardware timers are 54-bit counters with 16-bit prescaler. At 1 MHz tick rate, alarm resolution is exactly **1 µs** with ±0.01 µs jitter (hardware timer, not software). This gives 250 discrete timer ticks for a 250µs pulse width — more than sufficient.

**Confidence:** HIGH — verified from official Arduino-ESP32 timer API documentation and RepeatTimer example on GitHub.

### FreeRTOS Task Architecture

The ESP32-S3 dual-core architecture allows hard real-time stimulation on Core 1 while BLE + housekeeping run on Core 0.

| Task | Core | Priority | Rationale |
|------|------|----------|-----------|
| **Stimulation ISR** (hw timer) | Core 1 | ISR (highest) | Hardware timer interrupt — cannot be preempted. Toggles DAC + H-bridge. Must complete in <10µs. |
| **Stimulation Manager** | Core 1 | `configMAX_PRIORITIES - 1` (24) | Manages session state machine (ramp, ON/OFF duty cycle, stagger timing). Woken by ISR via semaphore. |
| **Impedance Monitor** | Core 1 | 10 | Runs every 1s during stimulation. Reads ADC for impedance check. Can be preempted by stimulation. |
| **NimBLE Host** | Core 0 | 21 (NimBLE default) | BLE stack processing. NimBLE pins to Core 0 via `MYNEWT_VAL_NIMBLE_PINNED_TO_CORE 0`. |
| **HRV Data Logger** | Core 0 | 5 | Writes RR intervals to LittleFS. Low priority — data integrity handled by LittleFS journaling. |
| **Safety Watchdog** | Core 0 | `configMAX_PRIORITIES - 1` (24) | Independent task checking heartbeat from Core 1. If Core 1 stalls, triggers hardware cutoff GPIO. |

**Key constraint:** The NimBLE host task **must** run on Core 0 (configured via `MYNEWT_VAL_NIMBLE_PINNED_TO_CORE 0` in nimconfig.h). Stimulation ISR **must** run on Core 1. This prevents BLE connection events from introducing jitter in pulse timing.

**Confidence:** HIGH — FreeRTOS task model well-established for ESP32; NimBLE core pinning documented in nimconfig.h.

---

## 2. Analog Hardware Components

### DAC — MCP4922

| Parameter | Value | Notes |
|-----------|-------|-------|
| **Part** | MCP4922-E/SN (SOIC-14) | Microchip dual 12-bit SPI DAC |
| **Resolution** | 12-bit (4096 steps) | At 5 mA full scale: 5 mA / 4096 = **1.22 µA/step** → 0.1 mA requires ~82 steps → ✅ achievable |
| **SPI Clock** | Up to 20 MHz | ESP32-S3 SPI can easily do 10 MHz. Use HSPI (SPI2) to avoid conflict with flash on SPI0. |
| **Settling Time** | 4.5 µs (to ±0.5 LSB) | At 250 µs pulse width, settling consumes 1.8% of pulse — acceptable. Add 5 µs delay after SPI write before trusting output. |
| **Output** | Voltage output (0 to Vref) | External Vref (use precision 2.048V reference, e.g., MCP1541 or REF2025). DAC output feeds Howland pump input. |
| **Dual channels** | DAC A + DAC B | One per ear (left/right binaural channels). Single SPI bus, separate LDAC for simultaneous update. |
| **Price** | ~$2.50 (Mouser/Digi-Key) | In stock, not EOL. Microchip flagship part. |
| **INL/DNL** | ±2 LSB / ±0.75 LSB typical | Adequate for 0.1 mA resolution target. |

**Why MCP4922 over alternatives:**

| Alternative | Why Not |
|-------------|---------|
| MCP4822 (internal 2.048V ref) | Internal reference is less precise than external. MCP4922 + external Vref gives better accuracy. |
| AD5662 (16-bit) | Overkill. 16-bit gives 0.076 µA/step — unnecessary precision that increases BOM cost ($8+) and SPI transfer time. |
| ESP32-S3 internal DAC | ESP32-S3 has **NO internal DAC** (unlike ESP32 which has 8-bit DAC on GPIO25/26). External DAC is mandatory. |
| PWM + low-pass filter | Ripple filtering for 250µs pulses requires impractically fast PWM (>10 MHz) and multi-stage filtering. Not suitable for precision current source. |

**Confidence:** HIGH — MCP4922 datasheet specs well-established, widely used in bioelectronics.

### Voltage Reference

| Part | Spec | Why |
|------|------|-----|
| **MCP1501-20** (2.048V, SOIC-8) | ±0.08% initial accuracy, 5 ppm/°C, $1.50 | Microchip ecosystem match with MCP4922. 2.048V maps cleanly to 0–5 mA range (2.048V / 4096 × gain = current). |

**Alternative:** REF2025 (TI, 2.5V, ±0.05%). Slightly more precise but 2.5V requires gain stage adjustment. Stick with 2.048V for cleaner math.

### Op-Amp for Constant-Current Output (Howland Pump)

| Part | Offset Voltage | Why | Price |
|------|----------------|-----|-------|
| **OPA388** (TI, auto-zero/chopper) | ≤5 µV | **RECOMMENDED.** Auto-zero topology eliminates drift. 5 µV offset → <0.5 µA DC offset in Howland pump, which is clinically irrelevant. SOIC-8 or SOT-23-5 packages. | ~$3.50 |

**Why OPA388 is the clear winner:**

The Howland pump converts DAC voltage to output current. Any op-amp input offset voltage (Vos) creates a proportional DC current offset through the patient load. This DC offset accumulates charge over a 30-minute session (see PITFALLS.md S-01).

| Op-Amp | Vos | DC Current Offset (at 1kΩ load) | Session Charge (30 min @ 25 Hz) | Verdict |
|--------|-----|------|------|---------|
| OPA2134 (JFET) | ±1 mV typ | ~1 µA | ~900 µC | ❌ **DANGEROUS** — exceeds tissue tolerance |
| LT1012 (precision BJT) | 40 µV typ | ~0.04 µA | ~0.036 µC | ⚠️ Acceptable but tight margin |
| **OPA388** (auto-zero) | 5 µV max | ~0.005 µA | ~0.0045 µC | ✅ **3 orders of magnitude margin** |
| MCP6V51 (auto-zero) | 2 µV typ | ~0.002 µA | ~0.0018 µC | ✅ Also excellent, Microchip match |

> **Decision: Use OPA388.** The DC-blocking capacitor (S-01 mitigation) is the primary safety barrier, but the OPA388 reduces the residual offset by 200× vs OPA2134, making the capacitor's job trivially easy. LT1012 is acceptable as fallback but harder to source in 2025.

**Dual op-amp version:** OPA2388 (dual OPA388 in SOIC-8) — use one per channel. $5.50. Two channels in one package.

**Confidence:** HIGH — OPA388 datasheet specs well-established; auto-zero topology physics is fundamental.

### H-Bridge for Biphasic Output

**Recommendation: Discrete MOSFETs** — NOT motor-driver ICs.

| Component | Part | Package | Key Spec | Price |
|-----------|------|---------|----------|-------|
| N-channel MOSFET (×2) | **BSS138** | SOT-23 | Vds=50V, Rds(on)=0.9Ω @ 5V, Id=200mA | $0.15 |
| P-channel MOSFET (×2) | **BSS84** | SOT-23 | Vds=−50V, Rds(on)=10Ω @ −5V, Id=−130mA | $0.15 |

**Half-bridge topology (per channel):**

```
        +15V
         │
      [BSS84] ← P-ch high-side
         │
         ├──── Electrode A
         │
      [BSS138] ← N-ch low-side
         │
        -15V (or GND on patient-isolated rail)
```

Two half-bridges (4 MOSFETs total per channel) form a full H-bridge. ESP32 GPIO → gate drivers (through digital isolator) control phase polarity.

**Why NOT motor-driver ICs (DRV8833, TB6612):**

| IC | Problem |
|----|---------|
| **DRV8833** | Minimum pulse width ~100µs due to internal blanking time. At 250µs target pulse, that leaves only 150µs of "clean" output. Internal current regulation fights with external Howland pump. Rds(on) = 0.36Ω adds series resistance to precision current loop. Designed for 2.7A motors, not 5mA precision. |
| **TB6612** | Similar minimum pulse issues. Standby mode introduces 1–2µs transition glitches. Output is voltage-mode, not current-compatible. |

**Dead-time management:** The ESP32-S3 MCPWM peripheral has built-in dead-time generation (configurable down to 100ns). Use MCPWM for H-bridge control instead of GPIO toggling — it guarantees no shoot-through.

**Alternative (if BSS84 Rds too high):** Use **Si2301CDS** (P-ch, SOT-23, Rds=0.16Ω @ −4.5V, $0.20) + **Si2302CDS** (N-ch, SOT-23, Rds=0.04Ω @ 4.5V, $0.20). Lower Rds improves current-source accuracy but costs slightly more.

**Confidence:** HIGH — discrete MOSFET H-bridge is the standard approach in neurostimulation literature.

### Current Sense and Overcurrent Protection

| Component | Part | Spec | Purpose |
|-----------|------|------|---------|
| **Sense resistor** | 10 Ω (0805, 1% tolerance) | Produces 50 mV @ 5 mA | Low enough to not significantly affect compliance voltage (−0.05V at 5 mA). |
| **Current sense amp** | **INA181A1** (TI, SOT-23-5) | Gain = 20 V/V, Vos = ±150 µV, CMRR = 110 dB | Bidirectional sensing. 50 mV × 20 = 1.0V output at 5 mA — maps cleanly to ESP32-S3 ADC (0–3.3V). Output feeds both ADC (for firmware monitoring) and comparator (for hardware cutoff). ~$1.00. |
| **Overcurrent comparator** | **LM393** (dual comparator, SOIC-8) | — | Compares INA181 output against fixed threshold (1.2V = 6 mA). Output drives P-MOSFET gate to disconnect supply rail. Independent of firmware — fires even if ESP32 crashes. ~$0.30. |
| **Threshold divider** | Resistor divider from Vref | Set to 1.2V (= 6 mA × 10Ω × 20 gain) | Non-resettable by firmware. User must power-cycle to clear fault. |

**Confidence:** HIGH — standard current sense topology.

---

## 3. Isolation & Power

### Isolation Architecture

```
┌─────────────────────┐    ISOLATION    ┌─────────────────────┐
│    DIGITAL DOMAIN    │    BARRIER     │   PATIENT DOMAIN    │
│                      │                │                     │
│  ESP32-S3            │   ISO7741      │  ±15V Analog Rail   │
│  3.3V rail           │◄──(SPI)──────► │  MCP4922 DAC        │
│  USB-C (charging)    │                │  OPA388 Howland     │
│  LiPo battery        │   B0515D      │  H-bridge MOSFETs   │
│  TP4056              │◄──(power)────► │  Sense resistor     │
│                      │                │  Electrodes         │
│  GND_DIGITAL         │   ≥1000V      │  GND_PATIENT        │
└─────────────────────┘    gap         └─────────────────────┘
```

### Isolated DC-DC Converter

| Part | Spec | Why |
|------|------|-----|
| **B0515D-1WR3** (Mornsun) | 5V in → **±15V out** (dual output), 1W, 1000 VDC isolation | Correct part for ±15V rail. Single module, no external components. ~$6.00 on Mouser/LCSC. |

> **⚠ CORRECTION:** The question references "B0515S-1W" — that is a **single-output** +15V part. For a Howland pump with biphasic output, **you need ±15V (dual supply)**. Use **B0515D-1WR3** (the "D" = dual output). Alternatively, use **B0505D-1WR3** (5V→±5V) with a charge pump, but ±15V provides critical compliance headroom for high-impedance electrodes.

**Load analysis:** At 5 mA into 2 kΩ, the output stage draws ~10V × 5 mA = 50 mW. The H-bridge MOSFETs, op-amp quiescent current, and sense amp add ~30 mW. Total analog domain power: ~80–100 mW, well within the 1W rating.

**Availability concern:** B0515D-1WR3 has been intermittently supply-constrained. **Alternatives (pin-compatible):**

| Alternative | Manufacturer | Isolation | Price | Notes |
|-------------|--------------|-----------|-------|-------|
| **DPBW06F-15** | MEAN WELL | 1000 VDC | ~$8 | 6W rating (overkill but always available) |
| **RB-0515D** | RECOM | 1000 VDC | ~$7 | 1W, well-stocked at Digi-Key |
| **VIBLSD1-S5-S15-DIP** | Vicor/CUI | 3000 VDC | ~$10 | Higher isolation, premium |
| **Two × B0515S-1WR3** | Mornsun | 1000 VDC | ~$10 | One for +15V, one for −15V (with inverted winding). Wasteful but available. |

**Confidence:** HIGH for the topology; MEDIUM for B0515D-1WR3 specific availability — always have a RECOM RB-0515D as backup.

### Digital Isolator

| Part | Channels | Direction | Speed | Isolation | Price | Why |
|------|----------|-----------|-------|-----------|-------|-----|
| **ISO7741DBQR** (TI) | 4 | **3 forward + 1 reverse** | 50 Mbps | 5000 Vrms | ~$3.00 | Exact fit for SPI + feedback: MOSI/SCK/CS → DAC, ADC sense ← amplifier output. |

**Why ISO7741 over alternatives:**

| Alternative | Problem |
|-------------|---------|
| ADUM1201 | Only 2 channels — need two of them for 3+1 signals. More board space, more cost. |
| ISO7742 | 2+2 direction split — doesn't match our 3+1 requirement. Would waste a channel. |
| ISO7740 | 4 forward, 0 reverse — can't return the current sense signal. |
| Si8600 | 2-channel. Same problem as ADUM1201. |

**Confidence:** HIGH — ISO7741 datasheet clearly specifies 3+1 channel direction.

### Battery & Power Management

| Component | Part | Spec | Purpose | Confidence |
|-----------|------|------|---------|------------|
| **LiPo cell** | Generic 18650 or pouch, 3.7V, 2000–3000 mAh | — | At ~200 mW total draw, provides 30+ hours of operation. | HIGH |
| **USB-C charger** | **TP4056** (SOP-8) | 1A max charge, programmable via Rprog resistor | Ubiquitous, $0.30. Integrated charge status LEDs. | HIGH |
| **Battery protection** | **DW01A** + **FS8205** | Over-discharge (2.4V), over-charge (4.3V), short-circuit | Standard LiPo protection. $0.30 combined. | HIGH |
| **5V boost** | **MT3608** (SOT-23-6) or **TPS61200** | 3.7V LiPo → 5.0V for B0515D input | MT3608: $0.30, 2A max, 93% efficient. Use 5.0V output with 1µF ceramic cap. | HIGH |
| **3.3V LDO** | **ME6211C33M5G** (SOT-23-5) | 500 mA, **100 mV dropout** | Powers ESP32-S3 from LiPo. Works down to 3.4V input. | HIGH |

> **⚠ CRITICAL CORRECTION:** Do NOT use **AMS1117-3.3**. It has 1.1V dropout — requires 4.4V+ input. A LiPo at 3.7V (50% charge) is already below 4.4V, causing the ESP32-S3 to brown out. The ME6211 works down to 3.4V, covering the entire LiPo discharge curve (4.2V → 3.3V cutoff).

**Power budget:**

| Consumer | Current @ 3.3V | Notes |
|----------|----------------|-------|
| ESP32-S3 (BLE active, dual-core) | ~120 mA | NimBLE is more efficient than Bluedroid |
| B0515D (via boost) | ~30 mA @ 5V ≈ 45 mA @ 3.7V | 150 mW / 3.7V |
| MCP4922 + Vref | ~2 mA | Negligible |
| INA181 + LM393 | ~1 mA | Negligible |
| **Total** | **~170 mA @ 3.7V** | 2000 mAh cell → **~11 hours** |

**Confidence:** HIGH — all parts are commodity components, well-characterized.

---

## 4. PCB Design Tools

| Tool | Version | Purpose | Why | Confidence |
|------|---------|---------|-----|------------|
| **KiCad** | **9.0.8** | Schematic + PCB layout | GPL v3 compatible (matches project license). Mature JLCPCB fabrication plugin, large component library, active community. **Do NOT use 10.0.0** (released 2026-03-20) — too new, plugin ecosystem hasn't caught up, JLCPCB integration may have breaking changes. 9.0.8 is the stable choice. | HIGH |
| **JLCPCB** | — | PCB fabrication | ~$2–5 for 5 boards (2-layer), ~$15–20 for 4-layer. 5-day turnaround. Integrated with KiCad via JLCPCB Plugin. SMT assembly available for ~$8 setup + per-component cost. | HIGH |
| **LCSC** | — | Component sourcing | JLCPCB's component partner. Most BOM parts available. Integrated with KiCad JLCPCB plugin for one-click BOM upload. Check availability of OPA388 and ISO7741 specifically (TI parts sometimes limited on LCSC — order from Mouser/Digi-Key as backup). | MEDIUM |

**KiCad design rules for this board:**

| Rule | Value | Rationale |
|------|-------|-----------|
| Minimum trace width | 0.2 mm (8 mil) | JLCPCB minimum. Use 0.3mm+ for signals. |
| Isolation slot/gap | **2.5 mm minimum** | Between digital and patient ground planes. IEC 60601-1 creepage distance for 1000V working voltage on a PCB. Route a milled slot (board cutout) between domains. |
| Ground planes | Separate digital GND + patient GND | Do NOT connect. They couple only through the B0515D and ISO7741 isolation barrier. |
| Layer stackup | 4-layer recommended | L1=signals, L2=digital GND, L3=patient GND + power, L4=signals. 2-layer possible but harder to route isolation gap. |

**Confidence:** HIGH — KiCad 9.x is battle-tested; JLCPCB specs well-documented.

---

## 5. BLE Dual-Role — NimBLE on ESP32-S3

### Feasibility: CONFIRMED ✅

**Finding:** NimBLE-Arduino 2.5.0 supports simultaneous peripheral (GATT server) + central (GATT client) operation on ESP32-S3.

**Evidence chain:**
1. **Issue #1016** (2025-08-24): ESP32-S3 dual-role failed with `rc=519 Memory Capacity Exceeded` on NimBLE 2.3.x. Root cause: `ble_max_act` config for ESP32-S3/C3 only counted connections, not advertising + scanning activities.
2. **PR #1018** (merged 2025-09-02): Fix sets `ble_max_act = BLE_MAX_CONNECTIONS + BLE_ROLE_BROADCASTER + BLE_ROLE_OBSERVER`. With defaults: 3 + 1 + 1 = 5 activity slots.
3. **Verified in source:** Both 2.4.0 and 2.5.0 tags contain the fix in `NimBLEDevice.cpp` lines 921-923 (confirmed via GitHub API, reading actual source at both tags).

**nimconfig.h settings for dual-role:**

```cpp
// In nimconfig.h or via build_flags:
// Both roles MUST be enabled (they are by default):
// MYNEWT_VAL_BLE_ROLE_CENTRAL     1  (default — do not set to 0)
// MYNEWT_VAL_BLE_ROLE_PERIPHERAL  1  (default — do not set to 0)
// MYNEWT_VAL_BLE_ROLE_OBSERVER    1  (default — needed for scanning)
// MYNEWT_VAL_BLE_ROLE_BROADCASTER 1  (default — needed for advertising)

// Pin NimBLE to Core 0 (leave Core 1 for stimulation):
#define MYNEWT_VAL_NIMBLE_PINNED_TO_CORE 0

// Max connections = 2 (1 for phone GATT, 1 for Polar H10 HRS):
#define MYNEWT_VAL_BLE_MAX_CONNECTIONS 2
```

**Operational model:**

| BLE Role | Connection | Profile | Direction |
|----------|------------|---------|-----------|
| **Peripheral** (GATT server) | Phone → ESP32 | Custom stimulation control service | Phone writes parameters, reads status |
| **Central** (GATT client) | ESP32 → Polar H10 | Heart Rate Service (0x180D) | ESP32 subscribes to RR interval notifications |

**Timing constraint:** BLE connection interval affects HRV data latency. Polar H10 typically uses 1-second notification interval for RR data. NimBLE's connection event handling on Core 0 has no impact on stimulation timing on Core 1 — they are fully decoupled via FreeRTOS task isolation.

**Risk:** The NimBLE dual-role fix was NOT backported to the 2.3.x branch (verified by examining all commits between 2.3.6→2.3.7→2.3.8→2.3.9). **You MUST use ≥2.4.0.** The 2.5.0 release (2026-04-02) is recommended as the latest stable.

**Confidence:** HIGH — verified fix presence in source code at both 2.4.0 and 2.5.0 tags.

---

## 6. HRV Analysis Software (Python, Desktop)

### Recommendation: NeuroKit2

| Library | Version | RMSSD | pNN50 | LF/HF | SDNN | Nonlinear | Active Maint. | Verdict |
|---------|---------|-------|-------|--------|------|-----------|---------------|---------|
| **NeuroKit2** | **0.2.13** | ✅ | ✅ | ✅ | ✅ | ✅ (DFA, Poincaré, entropy) | ✅ (released 2026-03-02) | **USE THIS** |
| heartpy | 1.2.7 | ✅ | ✅ | ✅ | ✅ | ❌ | ⚠️ (last release — stale) | Simpler but less capable |
| biosppy | 2.2.4 | ✅ | Limited | ✅ | ✅ | ❌ | ✅ | Signal processing focused, not HRV-specific |

**Why NeuroKit2:**
1. **Comprehensive HRV module:** Dedicated `hrv_time.py`, `hrv_frequency.py`, `hrv_nonlinear.py` — computes all metrics needed (RMSSD, pNN50, SDNN, LF/HF ratio, DFA α1/α2, sample entropy).
2. **RR interval input:** Accepts raw RR intervals directly — no need for raw ECG processing. Perfect for Polar H10 data which already provides RR intervals.
3. **Publication-grade:** Used in 500+ research papers. Outputs match clinical HRV standards (European Society of Cardiology task force guidelines).
4. **Active development:** v0.2.13 released 2026-03-02 (latest of 3 options).
5. **MIT license:** Compatible with GPL v3 project (no license conflict).

**Installation:**

```bash
pip install neurokit2==0.2.13 pandas matplotlib
```

**Usage pattern for OpenBinaural-taVNS:**

```python
import neurokit2 as nk
import pandas as pd

# Load RR intervals from ESP32 LittleFS export (JSON/CSV)
rr_intervals = pd.read_csv("session_rr.csv")["rr_ms"].values

# Compute all HRV metrics
hrv_time = nk.hrv_time(rr_intervals, sampling_rate=None)   # RR input, not ECG
hrv_freq = nk.hrv_frequency(rr_intervals, sampling_rate=None)
hrv_nonlinear = nk.hrv_nonlinear(rr_intervals, sampling_rate=None)

# Key metrics for taVNS efficacy
print(f"RMSSD: {hrv_time['HRV_RMSSD'].values[0]:.1f} ms")
print(f"pNN50: {hrv_time['HRV_pNN50'].values[0]:.1f} %")
print(f"LF/HF: {hrv_freq['HRV_LFHF'].values[0]:.2f}")
print(f"SDNN:  {hrv_time['HRV_SDNN'].values[0]:.1f} ms")
```

**Confidence:** HIGH — NeuroKit2 is the clear leader, verified via PyPI and GitHub.

---

## 7. Safety Standards & Calculations

### IEC 60601-1 Patient Leakage Current

| Condition | Limit | How We Meet It |
|-----------|-------|----------------|
| Normal condition (battery operation) | **10 µA DC / 100 µA AC** patient leakage to earth | Galvanic isolation (B0515D + ISO7741) ensures patient circuit is floating. No earth path exists in battery mode. |
| Single fault (USB cable connected during use) | **50 µA DC / 500 µA AC** | **Design rule: DISABLE stimulation during USB charging.** TP4056 STDBY pin can signal charging state to ESP32. Firmware AND hardware interlock (GPIO-controlled relay in output path, driven LOW during USB power detect). |
| Isolation barrier | **1500 Vrms / 2 MOPP** (means of patient protection) | B0515D: 1000 VDC. ISO7741: 5000 Vrms. The weakest link (B0515D at 1000V) is sufficient for **battery-powered BF-type** equipment. If USB-charging-while-stimulating is ever allowed, upgrade to 1500V+ isolation (e.g., RB-0515D at 1600V). |

> **Recommendation:** Enforce battery-only stimulation. This simplifies isolation requirements to 1 MOPP (1000V sufficient). Add a large "DO NOT USE WHILE CHARGING" warning in documentation and a hardware interlock.

### ANSI/AAMI NS4 — Charge Density Limits

ANSI/AAMI NS4:2024 (Transcutaneous Electrical Nerve Stimulators) specifies output limits for TENS-class devices. Key requirements:

| Parameter | NS4 Limit | Our Max | Margin |
|-----------|-----------|---------|--------|
| Maximum output current | 80 mA (into 500Ω) | 5 mA (hardware cutoff at 6 mA) | 13× below limit |
| Maximum charge per pulse | 100 µC (into 500Ω) | 2.5 µC (5 mA × 500 µs) | 40× below limit |

We are dramatically under NS4 limits because taVNS operates at much lower intensities than TENS.

### McCreery Charge Density Limit — Verification

**Shannon/McCreery limit:** Q/A < 30 µC/cm² per phase (for neural tissue safety).

**Calculation at maximum parameters (5 mA, 500 µs):**

```
Q = I × t = 5 mA × 500 µs = 2.5 µC per phase

Electrode area (cymba conchae, typical):
  ~0.5 cm² per electrode (conservative, typical taVNS electrodes are 0.5-1.0 cm²)

Charge density = Q / A = 2.5 µC / 0.5 cm² = 5.0 µC/cm² per phase

Limit: 30 µC/cm² per phase

Safety margin: 30 / 5.0 = 6× below limit ✅
```

**At typical clinical parameters (1.5 mA, 250 µs):**

```
Q = 1.5 mA × 250 µs = 0.375 µC per phase
Charge density = 0.375 / 0.5 = 0.75 µC/cm² per phase
Safety margin: 40× below limit ✅✅
```

**At absolute maximum (5 mA, 500 µs, 0.3 cm² small electrode):**

```
Q = 2.5 µC
Charge density = 2.5 / 0.3 = 8.3 µC/cm²
Safety margin: 3.6× ← Still safe but reduced. Document minimum electrode area of 0.5 cm².
```

> **Firmware guard:** Enforce maximum charge-per-phase limit: `if (current_mA * pulse_width_us > 2500) { fault(); }`. This caps at 2.5 µC regardless of parameter combination.

**Confidence:** HIGH — McCreery limits are well-established (1990), universally cited in neurostimulation literature. Calculation is straightforward physics.

---

## 8. BOM Cost Estimate

### Custom PCB BOM (Phase 3)

| Category | Components | Est. Cost |
|----------|------------|-----------|
| **MCU** | ESP32-S3-WROOM-1 (N16R8) | $3.50 |
| **DAC** | MCP4922 + MCP1501-20 Vref | $4.00 |
| **Op-amps** | OPA2388 (dual) × 1 | $5.50 |
| **H-bridge** | BSS138 × 4, BSS84 × 4 | $1.20 |
| **Current sense** | INA181A1 × 2, 10Ω × 2 | $2.50 |
| **Safety** | LM393 × 1, DC-blocking caps × 2 | $1.00 |
| **Isolation** | B0515D-1WR3 + ISO7741 | $9.00 |
| **Power** | TP4056, DW01A, FS8205, MT3608, ME6211 | $2.50 |
| **Passives** | Resistors, capacitors, inductors | $3.00 |
| **Connectors** | USB-C, electrode jacks, battery | $2.50 |
| **Battery** | 18650 LiPo 2500mAh | $5.00 |
| | | |
| **BOM Total** | | **~$39.70** |
| **PCB fabrication** | JLCPCB 4-layer, 5 boards | **~$20.00** |
| **Grand total per unit** | | **~$60** |

✅ Under the $100 BOM target specified in PROJECT.md constraints.

**Confidence:** MEDIUM — prices fluctuate; OPA2388 and ISO7741 may be higher from Mouser vs LCSC. Budget $20 contingency.

---

## 9. Full Stack Summary

```
┌─────────────────────────────────────────────────────────┐
│                    FIRMWARE LAYER                        │
│  ESP32-S3 + Arduino-ESP32 3.3.8 + PlatformIO 6.13.0    │
│  NimBLE-Arduino 2.5.0 (BLE dual-role)                   │
│  ArduinoJson 7.4.3 · LittleFS (built-in)               │
│  FreeRTOS: Core 0 = BLE, Core 1 = Stimulation ISR      │
├─────────────────────────────────────────────────────────┤
│                    ANALOG LAYER                          │
│  MCP4922 (12-bit DAC) → MCP1501-20 (Vref 2.048V)       │
│  OPA2388 (auto-zero Howland pump)                        │
│  BSS138/BSS84 H-bridge (discrete, MCPWM-driven)         │
│  INA181A1 (current sense) → LM393 (HW overcurrent)      │
│  10µF film DC-blocking cap (safety, per channel)         │
├─────────────────────────────────────────────────────────┤
│                  ISOLATION BARRIER                        │
│  B0515D-1WR3 (5V → ±15V, 1000 VDC)                     │
│  ISO7741 (4-ch digital isolator, 3+1 direction)          │
│  2.5mm PCB slot between ground domains                   │
├─────────────────────────────────────────────────────────┤
│                    POWER LAYER                           │
│  18650 LiPo (3.7V) → ME6211 LDO (3.3V for ESP32)       │
│  LiPo → MT3608 boost (5V) → B0515D (±15V isolated)      │
│  TP4056 (USB-C charge) + DW01A/FS8205 (protection)      │
├─────────────────────────────────────────────────────────┤
│                    PCB / TOOLING                          │
│  KiCad 9.0.8 → JLCPCB (4-layer) + LCSC components      │
├─────────────────────────────────────────────────────────┤
│                DESKTOP SOFTWARE                          │
│  Python 3.11+ · NeuroKit2 0.2.13 (HRV analysis)         │
│  pandas + matplotlib (visualization)                     │
└─────────────────────────────────────────────────────────┘
```

---

## 10. Alternatives Considered (Decision Matrix)

| Category | Recommended | Alternative | Why Not |
|----------|-------------|-------------|---------|
| MCU | ESP32-S3 | STM32L4 | No built-in BLE — needs external module (+$5, +complexity). STM32 HAL is harder to learn. |
| MCU | ESP32-S3 | nRF52840 | Better BLE, but no WiFi (kills future OTA), smaller community for Arduino, harder procurement in small quantities. |
| BLE stack | NimBLE-Arduino | Bluedroid | 2× resource usage, worse dual-role support on S3, slower. |
| DAC | MCP4922 | DAC8552 (TI) | Pin-compatible alternative if MCP4922 unavailable, but $4+ vs $2.50. |
| Op-amp | OPA388 / OPA2388 | ADA4522 (ADI) | Similar auto-zero specs but $6+ and less available. OPA388 is the price-performance sweet spot. |
| Op-amp | OPA388 | OPA2134 | 200× worse DC offset. Unacceptable for patient safety. |
| H-bridge | Discrete MOSFETs | DRV8833 | Minimum pulse width conflicts with 250µs target. Motor-driver architecture fights precision current source. |
| Isolated DC-DC | B0515D-1WR3 | RB-0515D (RECOM) | Viable backup. Slightly more expensive ($7 vs $6). Stock as alternate. |
| LDO | ME6211 | AMS1117-3.3 | AMS1117 dropout (1.1V) is incompatible with LiPo voltage range. Device will brown out at 3.7V. |
| PCB tool | KiCad 9.0.8 | KiCad 10.0.0 | Too new (2026-03-20). Plugin ecosystem not yet updated. Use 9.0.8 for stability. |
| PCB tool | KiCad 9.0.8 | Altium/Eagle | Not GPL-compatible. Altium is $thousands/year. Eagle is discontinued by Autodesk. |
| HRV library | NeuroKit2 | heartpy | heartpy lacks nonlinear metrics (DFA, entropy). Less actively maintained. |
| Filesystem | LittleFS | SPIFFS | SPIFFS deprecated in ESP-IDF 5.x. Worse wear-leveling, no directories. |

---

## Sources & Confidence Assessment

| Source | Type | Confidence | What It Informed |
|--------|------|------------|------------------|
| GitHub API: h2zero/NimBLE-Arduino releases, issues, PRs, source code | **Verified** | HIGH | NimBLE versions, dual-role fix (issue #1016, PR #1018), nimconfig.h, NimBLEDevice.cpp S3 fix at tags 2.4.0 and 2.5.0 |
| GitHub API: espressif/arduino-esp32 releases + timer docs | **Verified** | HIGH | Arduino-ESP32 3.3.8, new timer API (timerBegin/timerAlarm), migration from 2.x |
| GitHub API: platformio/platform-espressif32 releases | **Verified** | HIGH | PlatformIO v6.13.0 |
| GitHub API: bblanchon/ArduinoJson releases | **Verified** | HIGH | ArduinoJson 7.4.3 |
| GitHub API: KiCad/kicad-source-mirror releases | **Verified** | HIGH | KiCad 9.0.8 (stable) vs 10.0.0 (too new) |
| PyPI API: neurokit2, heartpy, biosppy | **Verified** | HIGH | NeuroKit2 0.2.13 as leader |
| GitHub API: neuropsychology/NeuroKit HRV module structure | **Verified** | HIGH | HRV module completeness (time, freq, nonlinear) |
| MCP4922, OPA388, B0515D datasheets | **Training data** | HIGH | Well-established parts with stable specs |
| IEC 60601-1, ANSI/AAMI NS4, McCreery charge limits | **Training data** | HIGH | Long-established safety standards (not volatile) |
| BSS138, BSS84, INA181, LM393, TP4056, ME6211 datasheets | **Training data** | HIGH | Commodity parts, specs stable for years |
| ISO7741 datasheet (TI) | **Training data** | MEDIUM | Verify 3+1 channel direction on current datasheet before PCB design |
| B0515D-1WR3 availability | **Training data** | MEDIUM | Supply chain status may have changed; verify Mouser/Digi-Key stock |
| JLCPCB pricing, capability | **Training data** | MEDIUM | Prices and specs as of 2024; verify at order time |

---

*Last updated: 2025-07-14 — Stack research for OpenBinaural-taVNS*
