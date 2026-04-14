# v1 Requirements — OpenBinaural-taVNS

> **Scope:** Personal insomnia self-experimentation + open-source community reference design.
> **Build path:** TENS hack (MVP validation) → Custom ESP32-S3 PCB (full reference design).
> **Safety constraints are non-negotiable and cannot be relaxed under any circumstance.**

---

## v1 Requirements

### SAFETY — Hardware (non-negotiable)

- [ ] **SAFE-01**: Charge-balanced biphasic square waveform — net DC current delivered to tissue = 0µC per cycle, enforced in both firmware (symmetric timing) and hardware (DC-blocking capacitors)
- [ ] **SAFE-02**: Series DC-blocking capacitor (10µF film, non-electrolytic) on each electrode output — physically prevents DC reaching tissue under any firmware fault condition
- [ ] **SAFE-03**: Hardware overcurrent cutoff at 6mA — comparator (LM393) monitors sense resistor, drives MOSFET to disconnect output independent of firmware; resettable by user only (power cycle)
- [ ] **SAFE-04**: Galvanic isolation — full electrical separation between digital ground (ESP32/USB) and patient ground; isolated DC-DC (B0515D-1WR3, ±15V) + digital isolators (Si8621/Si8622, 1500VDC reinforced) crossing barrier
- [ ] **SAFE-05**: USB charging interlock — stimulation output hardware-disabled when USB power detected; prevents mains ground loop bypass of isolation barrier
- [ ] **SAFE-06**: Firmware enforces parameter safety bounds (non-overridable): current ≤5mA, charge per phase ≤25µC (I × pulse_width), frequency × pulse_width × 2 < 1.0 (no pulse overlap), frequency ≤100Hz
- [ ] **SAFE-07**: "NOT A MEDICAL DEVICE" disclaimer printed on PCB silkscreen, firmware BLE device name, and all documentation

### SAFETY — Firmware (non-negotiable)

- [ ] **SAFE-08**: Hardware watchdog timer — if firmware fails to pet watchdog every 100ms, output stage hardware-disabled; requires user power cycle to reset
- [ ] **SAFE-09**: Fault-safe default state — output stage requires active assertion to deliver current; any power loss, reset, or unhandled exception results in zero output
- [ ] **SAFE-10**: Emergency stop — physical button directly connected to output enable MOSFET gate (not software-mediated); halts stimulation instantly regardless of firmware state

### HARDWARE — TENS Hack (Phase 1 MVP)

- [ ] **HW-01**: Commercial TENS unit modified to deliver constant-current output in 0.5–5mA range with ≤0.2mA resolution (precision V-to-I modification of existing output stage)
- [ ] **HW-02**: DC-blocking capacitors (10µF film) and galvanic isolation added to TENS modification to prevent charge accumulation
- [ ] **HW-03**: Auricular electrode adapter fabricated for cymba conchae placement; validated for contact impedance <5kΩ on dry skin

### HARDWARE — Custom PCB (Phase 3)

- [ ] **HW-04**: ESP32-S3 microcontroller (dual-core 240MHz, BLE 5.0) as main MCU
- [ ] **HW-05**: MCP4922 dual 12-bit SPI DAC — independent waveform setpoint per channel (1.22µA/step resolution at 5mA full scale — 82× finer than 0.1mA requirement)
- [ ] **HW-06**: Dual independent V-to-I constant-current output stages (op-amp + sense resistor feedback per channel) using OPA388 auto-zero op-amp; not shared/switched
- [ ] **HW-07**: Discrete MOSFET H-bridge per channel (BSS138 N-ch + BSS84 P-ch) for biphasic polarity reversal with dead-time control via MCPWM peripheral
- [ ] **HW-08**: Isolated power: LiPo 3.7V → 5V boost → B0515D-1WR3 isolated DC-DC → ±15V analog rail (patient side, floating); post-regulated to ±12V for op-amp supply
- [ ] **HW-09**: LiPo battery management: TP4056 USB-C charging + DW01A/FS8205 protection; ME6211 100mV-dropout LDO for ESP32-S3 3.3V (NOT AMS1117 — incompatible with LiPo direct input)
- [ ] **HW-10**: Oscilloscope test point — buffered header exposing output waveform across known load resistor; enables community waveform verification without opening device
- [ ] **HW-11**: Complete KiCad 9.x schematic + PCB layout published under GPL v3; JLCPCB-ready Gerbers included

### FIRMWARE — Stimulation Engine (Phase 2)

- [ ] **FW-01**: Charge-balanced biphasic square waveform generated via hardware timer (ESP32-S3 timerBegin/timerAlarm API, Arduino-ESP32 ≥3.3.8); 1µs pulse-width resolution; state machine: IDLE → RAMP_UP → [PHASE_POS → GAP → PHASE_NEG → REST] × N → RAMP_DOWN → IDLE
- [ ] **FW-02**: Configurable parameters per channel: frequency (1–100Hz), pulse width (50–1000µs), current (0.1–5.0mA in 0.1mA steps), duty cycle (ON time 1–60s, OFF time 1–60s), session duration (1–120min)
- [ ] **FW-03**: Soft-start current ramp — linear interpolation 0 → target over configurable period (default 30s); symmetric ramp-down at session end; ~3µA/pulse increment at 25Hz/30s/2mA
- [ ] **FW-04**: Binaural asynchronous stagger — Channel B fires with configurable offset after Channel A (0–40ms range, default 20ms = half-period at 25Hz); 0ms = synchronous mode (replicates commercial bilateral devices)
- [ ] **FW-05**: Independent per-channel current — left and right ear current set independently via BLE (e.g. L=1.5mA, R=2.0mA) to account for bilateral impedance asymmetry
- [ ] **FW-06**: Real-time electrode impedance sensing — pre-session: inject 100µA test pulse, measure voltage (good: 500Ω–5kΩ, warn: >5kΩ, abort: >10kΩ or <100Ω); during session: monitor compliance voltage, auto-pause if hitting rail
- [ ] **FW-07**: Real-time charge density calculation — compute charge/phase (I × pulse_width in µC) and charge density (µC/cm², using configurable electrode area parameter); warn at >50% McCreery limit, hard-block at >80%; display via BLE notify
- [ ] **FW-08**: Session presets stored in ESP32 NVS — minimum presets:
  - **"Insomnia Standard"**: 25Hz, 200µs, 2.0mA, 30s ON/30s OFF, 30min, 20ms stagger
  - **"RESET-AF Replication"**: 20Hz, 200µs, threshold (1.5mA default), continuous, 60min, synchronous
  - **"Anti-Inflammatory / Chronic Pain"**: 10Hz, 250µs, 1.5mA, 30s ON/30s OFF, 60min, 20ms stagger — targets vagal anti-inflammatory reflex (α7 nAChR pathway)
  - **"Dysautonomia / hEDS"**: 25Hz, 200µs, 1.0mA, 30s ON/30s OFF, 30min, 20ms stagger — targets sympathetic overdrive in POTS/hEDS; ⚠️ experimental, low clinical evidence, labeled accordingly
  - **"Exploration"**: all parameters user-defined
- [ ] **FW-09**: FreeRTOS task architecture — Core 1: stimulation ISR (highest priority, hardware timer) + impedance monitor; Core 0: BLE stack (NimBLE ≥2.4.0) + session manager + flash logger; stimulation ISR must never be preempted by BLE

### FIRMWARE — BLE Interface (Phase 4)

- [ ] **BLE-01**: BLE GATT peripheral server exposing stimulation control characteristics: current_L/R (uint16 ×100 = 0.01mA resolution), frequency_Hz (uint8), pulse_width_us (uint16), stagger_ms (uint8), duty_on_s/off_s (uint8), session_min (uint8), channel_enable (bitfield), preset_id (uint8), command (start/stop/pause/emergency_stop), status (notify: running/paused/fault/complete), impedance_L/R (notify, uint16 Ω), charge_density (notify, float µC/cm²)
- [ ] **BLE-02**: BLE central client connects to Polar H10 chest strap (BLE HRS profile, Heart Rate Measurement characteristic); receives RR intervals and logs to LittleFS during session; NimBLE dual-role (peripheral + central simultaneously) confirmed working on ESP32-S3 with NimBLE ≥2.4.0
- [ ] **BLE-03**: Post-session HRV report computed on-device and transmitted via BLE notify: RMSSD, pNN50, LF/HF ratio, SDNN — computed for pre-session baseline (5min), stimulation window, and recovery (5min); clinically meaningful threshold: RMSSD increase >20% = parasympathetic response confirmed
- [ ] **BLE-04**: OTA firmware update via BLE DFU — dual OTA flash partitions; firmware signature verification before applying; automatic rollback if new firmware fails watchdog on first boot; stimulation must be stopped before OTA can begin

### DATA & DOCUMENTATION

- [ ] **DATA-01**: Session logging to LittleFS — per session: timestamp (BLE-synced from phone), parameters (all), impedance at start/end (L/R), HRV summary (RMSSD pre/during/post), fault events log
- [ ] **DATA-02**: Open data export — CSV and JSON via USB serial dump or BLE file transfer; Python companion script (NeuroKit2 0.2.13) for post-session HRV visualization and multi-session analysis
- [ ] **DOC-01**: Clinical evidence summary — "Golden Parameters" table with citations (2020–2026 literature), evidence level per parameter, neurobiological rationale for asynchronous binaural stimulation
- [ ] **DOC-02**: Safety & Contraindications document — absolute contraindications: cardiac pacemaker/ICD, active epilepsy, pregnancy; relative: cervical vagotomy, severe cardiac arrhythmia; "NOT A MEDICAL DEVICE" statement
- [ ] **DOC-03**: Complete assembly guide — TENS hack walkthrough + custom PCB build walkthrough with photos/diagrams; reproducible by technically-capable builder without lab equipment
- [ ] **DOC-04**: HRV verification protocol — baseline measurement, session protocol, recovery period, interpreting results, expected vs. non-response criteria
- [ ] **DOC-05**: GitHub release — versioned firmware (.bin), KiCad files, BOM (Mouser + Digi-Key part numbers), Gerbers, and all documentation

---

## v2 Requirements (Deferred)

- Native iOS/Android app — generic BLE apps sufficient for v1; app is future milestone
- Multi-user session profiles — single-user self-experimentation is the v1 target
- Real-time waveform streaming to phone — high BLE bandwidth, complexity not justified for v1
- EMG/EEG co-monitoring — requires additional hardware; out of scope for v1
- CE/FDA medical device certification — explicitly deferred; requires ISO 13485 QMS

---

## Out of Scope

- Invasive VNS — different modality, surgical implant
- Transcranial stimulation (tDCS/tACS) — different target, different safety profile; explicitly blocked at hardware level
- Transcervical (neck) taVNS — auricular only; electrode design and impedance thresholds are ear-specific
- Current above 5mA — hardware cutoff at 6mA, no bypass
- Cloud data sync — all data local; open format lets users sync to their own tools
- Automatic threshold detection — unreliable; manual titration only

---

## Traceability

| Req ID | Phase | Roadmap Phase |
|--------|-------|---------------|
| SAFE-01 to SAFE-10 | All | Phase 1 (TENS) + Phase 2 (Firmware) + Phase 3 (PCB) |
| HW-01 to HW-03 | TENS Hack | Phase 1 |
| HW-04 to HW-11 | Custom PCB | Phase 3 |
| FW-01 to FW-09 | Stimulation Engine | Phase 2 |
| BLE-01 to BLE-04 | BLE Interface | Phase 4 |
| DATA-01 to DATA-02 | Data | Phase 4 |
| DOC-01 to DOC-05 | Documentation | Phase 5 |

---

*Generated: 2026-04-14 | Research confidence: HIGH (stack/architecture/safety), MEDIUM (clinical protocols), LOW (asynchronous binaural hypothesis — primary experimental goal)*
