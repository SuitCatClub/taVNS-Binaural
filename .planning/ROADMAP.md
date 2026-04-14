# Roadmap: OpenBinaural-taVNS v1.0

## Overview

This roadmap delivers an open-source binaural taVNS device from clinical parameter validation through production PCB and community release. The build follows a dual-track strategy: first validating stimulation parameters on a modified TENS unit (Phase 1), then building the custom ESP32-S3 firmware and analog signal chain incrementally on breadboard (Phases 2–7), committing the proven design to a 4-layer PCB (Phase 8), and publishing everything for the community (Phase 9). Safety hardware and firmware are front-loaded — no patient-facing output exists until all protection layers are validated. 41 v1 requirements across 9 phases.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: TENS Hack — Clinical Parameter Validation** - Modify commercial TENS unit, build electrode adapter, validate golden parameters produce HRV response
- [ ] **Phase 2: Waveform Engine — Biphasic Generation & Core Architecture** - ESP32-S3 generates precise charge-balanced biphasic waveform via hardware timer ISR on breadboard
- [ ] **Phase 3: Firmware Safety — Bounds, Watchdog & Soft-Start** - Firmware enforces all safety invariants: parameter bounds, watchdog, fault-safe state, soft-start ramp
- [ ] **Phase 4: Safety Hardware — Isolation Barrier & Protection Circuits** - Hardware-enforced safety layers (isolation, overcurrent, DC-blocking, E-stop, USB interlock) validated on breadboard
- [ ] **Phase 5: Dual-Channel — Binaural Stagger & Impedance Sensing** - Full binaural stimulation with configurable inter-channel stagger and real-time impedance monitoring
- [ ] **Phase 6: BLE Interface — GATT Server & Polar H10 HRV** - Phone controls stimulation via BLE; Polar H10 provides real-time HRV data during sessions
- [ ] **Phase 7: Session Management — Presets, Logging & Data Export** - Clinical presets stored on device; session data logged and exportable for analysis
- [ ] **Phase 8: Custom PCB — KiCad Schematic, Layout & Board Bringup** - Production-quality 4-layer PCB designed, fabricated, assembled, and validated
- [ ] **Phase 9: Documentation & Open-Source Release** - Complete build guides, safety docs, clinical references, and GitHub release under GPL v3

## Phase Details

### Phase 1: TENS Hack — Clinical Parameter Validation
**Goal**: Validate that golden stimulation parameters (25Hz, 200µs, sub-threshold current, cymba conchae) produce measurable HRV changes on the builder before investing in custom hardware
**Depends on**: Nothing (first phase)
**Requirements**: HW-01, HW-02, HW-03
**Success Criteria** (what must be TRUE):
  1. Modified TENS unit delivers constant-current biphasic pulses at 25Hz/200µs in the 0.5–5mA range, verified on oscilloscope across 1kΩ dummy load with ≤0.2mA resolution
  2. DC-blocking capacitors (10µF film) installed on each output; DC offset across dummy load measures <1mV after 10 minutes of continuous stimulation at max intensity
  3. Cymba conchae electrode adapter achieves stable contact impedance <5kΩ on dry skin, confirmed by TENS unit test pulse measurement
  4. Pre/post-session HRV comparison (Polar H10 chest strap) shows measurable parasympathetic shift (RMSSD change documented) for at least one golden parameter combination
**Plans**: TBD

### Phase 2: Waveform Engine — Biphasic Generation & Core Architecture
**Goal**: ESP32-S3 generates a precise, charge-balanced biphasic waveform on a breadboard analog stage via hardware timer ISR, with dual-core FreeRTOS architecture established
**Depends on**: Phase 1
**Requirements**: FW-01, FW-02, FW-09, SAFE-01
**Success Criteria** (what must be TRUE):
  1. Hardware timer ISR on Core 1 generates biphasic square waveform at configurable frequency (1–100Hz) with 1µs pulse-width resolution, verified on oscilloscope — state machine cycles through IDLE → PHASE_POS → GAP → PHASE_NEG → REST
  2. MCP4922 DAC driven via SPI outputs correct current setpoints; V-to-I stage (OPA388 + 100Ω Rsense) delivers 0.1–5.0mA into 1kΩ dummy load with ±0.1mA accuracy across full range
  3. Charge balance verified: AC-coupled oscilloscope shows zero baseline drift over 60 seconds at maximum stimulation intensity; positive and negative phase areas match within ±2%
  4. FreeRTOS dual-core architecture operational — stimulation ISR pinned to Core 1, NimBLE stack reserved for Core 0, no timing jitter >10µs under load (verified with logic analyzer)
**Plans**: TBD

### Phase 3: Firmware Safety — Bounds, Watchdog & Soft-Start
**Goal**: Firmware enforces all safety invariants and gracefully handles faults — no unsafe parameter combination can reach the output stage, and any firmware failure results in zero current
**Depends on**: Phase 2
**Requirements**: FW-03, SAFE-06, SAFE-08, SAFE-09
**Success Criteria** (what must be TRUE):
  1. Parameter validation rejects any input exceeding safety bounds (current >5mA, charge/phase >25µC, frequency >100Hz, frequency × pulse_width × 2 ≥ 1.0) and logs the rejection — tested via serial console with boundary and out-of-range values
  2. Hardware watchdog triggers output shutdown within 100ms of firmware freeze, verified by injecting `while(true)` into the main loop — device requires power cycle to resume
  3. Soft-start ramp delivers linear current increase from 0→target over configurable period (default 30s) with symmetric ramp-down at session end, verified on oscilloscope as smooth envelope over pulse train
  4. Device powers up in fault-safe state with zero output on all channels; stimulation requires explicit start command — verified by power cycling during active stimulation and confirming zero current on restart
**Plans**: TBD

### Phase 4: Safety Hardware — Isolation Barrier & Protection Circuits
**Goal**: Hardware-enforced safety layers protect the user independent of firmware state — any single failure (firmware crash, component fault, user error) results in zero current to the patient
**Depends on**: Phase 3
**Requirements**: SAFE-02, SAFE-03, SAFE-04, SAFE-05, SAFE-10
**Success Criteria** (what must be TRUE):
  1. Galvanic isolation verified: megohmmeter reads >10MΩ between digital ground and patient ground at 500V test voltage; leakage current <1µA — Si8621/Si8622 + B0515D-1WR3 crossing the barrier
  2. Hardware overcurrent comparator (LM393 + sense resistor) disconnects output within 10µs when current exceeds 6mA, completely independent of firmware — requires user power cycle to reset the latch
  3. USB VBUS detection hardware-disables stimulation output via AND gate logic; zero current flows to electrodes while USB cable is connected, even if firmware explicitly commands stimulation
  4. Emergency stop button (normally-open, pull-up to 3.3V, wired directly to output enable MOSFET gate) cuts output to zero within 1ms regardless of firmware state — not software-mediated
  5. DC-blocking capacitors (10µF film, non-electrolytic) installed on each electrode output with 1MΩ bleed resistor; verified zero DC offset under all fault injection scenarios including firmware crash with DAC stuck at max code
**Plans**: TBD

### Phase 5: Dual-Channel — Binaural Stagger & Impedance Sensing
**Goal**: Full binaural stimulation with two independent channels, configurable inter-channel stagger, real-time impedance monitoring, and charge density safety calculations
**Depends on**: Phase 4
**Requirements**: FW-04, FW-05, FW-06, FW-07
**Success Criteria** (what must be TRUE):
  1. Two independent channels deliver different current levels simultaneously (e.g., L=1.5mA, R=2.0mA) from a single hardware timer — verified on dual-channel oscilloscope with separate dummy loads
  2. Channel B fires with configurable offset after Channel A (0–40ms range, default 20ms); stagger is deterministic and drift-free over 30-minute sessions, visible on oscilloscope as consistent inter-channel delay
  3. Pre-session impedance check injects 100µA test pulse per channel and classifies electrode contact: good (500Ω–5kΩ), warn (>5kΩ), abort (>10kΩ or <100Ω); during-session compliance voltage monitoring auto-pauses if hitting rail
  4. Real-time charge density calculation computes µC/cm² using configurable electrode area; warns via serial at >50% McCreery limit (15 µC/cm²) and hard-blocks stimulation at >80% (24 µC/cm²)
**Plans**: TBD

### Phase 6: BLE Interface — GATT Server & Polar H10 HRV
**Goal**: Phone controls all stimulation parameters via BLE GATT; ESP32-S3 simultaneously connects to Polar H10 for real-time HRV monitoring; OTA firmware updates supported
**Depends on**: Phase 5
**Requirements**: BLE-01, BLE-02, BLE-03, BLE-04
**Success Criteria** (what must be TRUE):
  1. BLE GATT server exposes all stimulation parameters (current L/R, frequency, pulse width, stagger, duty cycle, session duration, channel enable, preset ID) and accepts commands (start/stop/pause/emergency_stop) from nRF Connect or any generic BLE app
  2. ESP32-S3 simultaneously maintains BLE peripheral (phone control) and BLE central (Polar H10 HRS) connections via NimBLE dual-role; RR intervals logged to LittleFS continuously throughout a 30-minute session without disconnection
  3. Post-session HRV report computed on-device (RMSSD, pNN50, LF/HF ratio, SDNN) for three windows (5-min pre-session baseline, stimulation period, 5-min recovery) and transmitted via BLE notify — clinically meaningful threshold: RMSSD increase >20% flags parasympathetic response
  4. OTA firmware update via BLE DFU uses dual flash partitions; firmware signature verified before applying; automatic rollback if new firmware fails watchdog within 30 seconds of first boot; stimulation must be stopped before OTA begins
**Plans**: TBD

### Phase 7: Session Management — Presets, Logging & Data Export
**Goal**: All 6 clinical presets stored and selectable on device; every session fully logged; data exportable for desktop HRV analysis
**Depends on**: Phase 6
**Requirements**: FW-08, DATA-01, DATA-02
**Success Criteria** (what must be TRUE):
  1. All 6 firmware presets selectable via BLE preset_id characteristic: Insomnia-25Hz (25Hz/200µs/2mA/30s-30s/30min/20ms stagger), Insomnia-20Hz (20Hz variant), RESET-AF (20Hz/continuous/60min/unilateral left), Anti-Inflammatory (25Hz/250µs/1.5mA/60min), Dysautonomia (25Hz/0.5→1mA ramp/theoretical-only warning), and Exploration (user-defined)
  2. Each session logs to LittleFS: BLE-synced timestamp, all stimulation parameters, impedance at start/end (L/R), HRV summary (RMSSD pre/during/post), and any fault events — data persists across power cycles
  3. Session data exportable as CSV and JSON via USB serial dump or BLE file transfer; Python companion script (NeuroKit2 0.2.13) generates per-session HRV visualization and multi-session trend analysis from exported data
  4. Dysautonomia/hEDS preset displays mandatory "THEORETICAL ONLY — zero published clinical evidence" warning via BLE notify and requires explicit user acknowledgment before session starts
**Plans**: TBD

### Phase 8: Custom PCB — KiCad Schematic, Layout & Board Bringup
**Goal**: Production-quality 4-layer PCB captures the entire breadboard-validated design, fabricated at JLCPCB, assembled, and passes all validation tests from Phases 2–5
**Depends on**: Phase 7
**Requirements**: HW-04, HW-05, HW-06, HW-07, HW-08, HW-09, HW-10, HW-11
**Success Criteria** (what must be TRUE):
  1. KiCad 9.x schematic captures the complete validated circuit: ESP32-S3-WROOM-1 (N16R8), MCP4922 dual DAC, OPA388 V-to-I stages (100Ω Rsense), H-bridge per channel, B0515D-1WR3 isolated ±15V supply, Si8621/Si8622 digital isolators, LM393 overcurrent latch, ME6211 LDO, TP4056+DW01A charging, and all safety interlocks
  2. 4-layer PCB layout passes DRC with 2.5mm milled isolation slot between digital and patient ground domains, split ground planes with no traces crossing the split, and analog signal traces <15mm from DAC to V-to-I input
  3. Assembled board passes all Phase 2–5 validation tests on custom hardware: waveform accuracy, charge balance, isolation integrity (>10MΩ at 500V), overcurrent cutoff, DC-blocking, impedance sensing, dual-channel stagger, E-stop, USB interlock
  4. Buffered oscilloscope test point header exposes output waveform across known load resistor — accessible without opening enclosure; community builder can verify waveform with standard scope probe
  5. JLCPCB-ready Gerbers, BOM with Mouser/Digi-Key part numbers, and complete KiCad project files published to repository under GPL v3
**Plans**: TBD

### Phase 9: Documentation & Open-Source Release
**Goal**: Any technically-capable person can build, verify, and safely use the OpenBinaural-taVNS device from published documentation alone — complete with clinical references, safety warnings, and reproducible build instructions
**Depends on**: Phase 8
**Requirements**: SAFE-07, DOC-01, DOC-02, DOC-03, DOC-04, DOC-05
**Success Criteria** (what must be TRUE):
  1. "NOT A MEDICAL DEVICE" disclaimer verified present on: PCB silkscreen, BLE device name string, firmware serial startup banner, README.md header, and every documentation page — comprehensive audit checklist completed (SAFE-07)
  2. Clinical evidence summary published with golden parameters table, per-parameter evidence level, citations (2020–2026 literature), and neurobiological rationale for asynchronous binaural stimulation including explicit acknowledgment that binaural stagger is an unvalidated hypothesis
  3. Safety & contraindications document published with absolute contraindications (cardiac pacemaker/ICD, active epilepsy, pregnancy), relative contraindications (cervical vagotomy, severe arrhythmia, digoxin), and clear "DO NOT USE IF" language
  4. Complete assembly guide enables a technically-capable builder to construct both the TENS hack modification and the custom PCB build using only the documentation, component-level photos/diagrams, and standard tools (soldering iron, multimeter, oscilloscope)
  5. HRV verification protocol provides step-by-step procedure: Polar H10 setup, 5-min baseline, stimulation session, 5-min recovery, data export, NeuroKit2 analysis, and interpretation guide with expected vs. non-response criteria
  6. GitHub release published with versioned firmware binary (.bin), KiCad project files, Gerbers, BOM (Mouser + Digi-Key part numbers with alternates), Python analysis scripts, and all documentation — tagged and licensed GPL v3
**Plans**: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 9

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. TENS Hack — Clinical Parameter Validation | 0/TBD | Not started | - |
| 2. Waveform Engine — Biphasic Generation & Core Architecture | 0/TBD | Not started | - |
| 3. Firmware Safety — Bounds, Watchdog & Soft-Start | 0/TBD | Not started | - |
| 4. Safety Hardware — Isolation Barrier & Protection Circuits | 0/TBD | Not started | - |
| 5. Dual-Channel — Binaural Stagger & Impedance Sensing | 0/TBD | Not started | - |
| 6. BLE Interface — GATT Server & Polar H10 HRV | 0/TBD | Not started | - |
| 7. Session Management — Presets, Logging & Data Export | 0/TBD | Not started | - |
| 8. Custom PCB — KiCad Schematic, Layout & Board Bringup | 0/TBD | Not started | - |
| 9. Documentation & Open-Source Release | 0/TBD | Not started | - |

## Requirement Coverage

| Req ID | Requirement Summary | Phase |
|--------|---------------------|-------|
| SAFE-01 | Charge-balanced biphasic waveform (firmware + hardware) | Phase 2 |
| SAFE-02 | Series DC-blocking capacitor per electrode output | Phase 4 |
| SAFE-03 | Hardware overcurrent cutoff at 6mA (LM393 latch) | Phase 4 |
| SAFE-04 | Galvanic isolation (B0515D-1WR3 + Si8621/Si8622) | Phase 4 |
| SAFE-05 | USB charging interlock — no stim while USB connected | Phase 4 |
| SAFE-06 | Firmware parameter safety bounds (non-overridable) | Phase 3 |
| SAFE-07 | "NOT A MEDICAL DEVICE" disclaimer on all surfaces | Phase 9 |
| SAFE-08 | Hardware watchdog timer (100ms) | Phase 3 |
| SAFE-09 | Fault-safe default state (zero output on any fault) | Phase 3 |
| SAFE-10 | Emergency stop physical button (not software-mediated) | Phase 4 |
| HW-01 | TENS modification for constant-current 0.5–5mA output | Phase 1 |
| HW-02 | DC-blocking + galvanic isolation on TENS modification | Phase 1 |
| HW-03 | Cymba conchae electrode adapter (<5kΩ impedance) | Phase 1 |
| HW-04 | ESP32-S3 MCU (dual-core 240MHz, BLE 5.0) | Phase 8 |
| HW-05 | MCP4922 dual 12-bit SPI DAC | Phase 8 |
| HW-06 | Dual V-to-I output stages (OPA388 + Rsense feedback) | Phase 8 |
| HW-07 | H-bridge per channel for biphasic polarity reversal | Phase 8 |
| HW-08 | Isolated power: B0515D-1WR3 → ±15V analog rail | Phase 8 |
| HW-09 | LiPo management: TP4056 + ME6211 LDO (NOT AMS1117) | Phase 8 |
| HW-10 | Oscilloscope test point (buffered, external access) | Phase 8 |
| HW-11 | KiCad 9.x schematic + PCB layout (GPL v3, JLCPCB-ready) | Phase 8 |
| FW-01 | Charge-balanced biphasic waveform via hardware timer ISR | Phase 2 |
| FW-02 | Configurable parameters per channel (freq, PW, current, duty, duration) | Phase 2 |
| FW-03 | Soft-start current ramp (0→target, configurable period) | Phase 3 |
| FW-04 | Binaural asynchronous stagger (0–40ms, default 20ms) | Phase 5 |
| FW-05 | Independent per-channel current (L and R set separately) | Phase 5 |
| FW-06 | Real-time electrode impedance sensing (pre + during session) | Phase 5 |
| FW-07 | Real-time charge density calculation + safety limits | Phase 5 |
| FW-08 | Session presets (6 presets stored in ESP32 NVS) | Phase 7 |
| FW-09 | FreeRTOS dual-core architecture (Core 1=stim, Core 0=BLE) | Phase 2 |
| BLE-01 | BLE GATT peripheral server (stim control characteristics) | Phase 6 |
| BLE-02 | BLE central client for Polar H10 HRS (RR interval logging) | Phase 6 |
| BLE-03 | Post-session HRV report (RMSSD, pNN50, LF/HF, SDNN) | Phase 6 |
| BLE-04 | OTA firmware update via BLE DFU (dual partition + rollback) | Phase 6 |
| DATA-01 | Session logging to LittleFS (params, impedance, HRV, faults) | Phase 7 |
| DATA-02 | Open data export (CSV/JSON + Python NeuroKit2 script) | Phase 7 |
| DOC-01 | Clinical evidence summary + golden parameters table | Phase 9 |
| DOC-02 | Safety & contraindications document | Phase 9 |
| DOC-03 | Complete assembly guide (TENS hack + custom PCB) | Phase 9 |
| DOC-04 | HRV verification protocol (baseline/session/recovery) | Phase 9 |
| DOC-05 | GitHub release (firmware, KiCad, BOM, Gerbers, docs) | Phase 9 |

**Coverage: 41/41 v1 requirements mapped ✓ — No orphans, no duplicates.**
