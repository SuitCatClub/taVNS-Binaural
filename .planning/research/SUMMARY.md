# Research Summary: OpenBinaural-taVNS

**Project:** OpenBinaural-taVNS (DIY Binaural Transcutaneous Auricular Vagus Nerve Stimulation)
**Domain:** DIY bioelectronic neurostimulation device — insomnia protocol
**Researched:** 2025-07-14
**Overall Confidence:** HIGH (stack and safety verified via datasheets, GitHub API, and clinical literature; architecture grounded in proven bioelectronics patterns)

---

## Executive Summary

This project builds a battery-powered, BLE-controlled, dual-channel constant-current neurostimulator that delivers charge-balanced biphasic pulses to the cymba conchae of both ears, with configurable inter-channel stagger, real-time HRV monitoring via Polar H10, and full session logging. The technology to build it is mature and well-characterized. Experts build devices like this with galvanically isolated analog output stages, hardware-enforced safety (never software-only), and deterministic pulse timing from hardware timer ISRs — not FreeRTOS tasks.

The recommended approach is a phased build: (1) validate clinical parameters with a modified TENS unit, (2) develop the stimulation firmware on an ESP32-S3 dev board with breadboard analog, (3) incrementally add safety layers, isolation, and dual-channel capability, (4) integrate BLE control and HRV logging, (5) commit the validated design to a 4-layer PCB. The critical path is the analog signal chain + isolation boundary (~10-12 weeks on the breadboard prototype before PCB commitment). BLE and Python HRV analysis are parallelizable.

The key risks are: (a) DC offset accumulation causing tissue damage — mitigated by a mandatory series DC-blocking capacitor per channel plus auto-zero op-amps, (b) H-bridge shoot-through — mitigated by using an integrated H-bridge driver IC with built-in dead-time, (c) USB ground-loop leakage through the patient — mitigated by galvanic isolation + hardware interlock that disables stimulation when USB VBUS is detected, and (d) NimBLE BLE dual-role stability — confirmed working in ≥2.4.0, but requires sequential connection strategy and long-duration soak testing. The BOM cost is ~$60/unit, well under the $100 target.

---

## Key Findings

### Recommended Stack
*(from STACK.md)*

The firmware and tooling stack is verified and version-pinned:

| Component | Choice | Version | Rationale |
|-----------|--------|---------|-----------|
| **MCU** | ESP32-S3-WROOM-1 (N16R8) | — | Dual-core @ 240MHz, native BLE 5.0, 4 HW timers, 16MB flash + 8MB PSRAM |
| **Framework** | Arduino-ESP32 | **3.3.8** | Mature, wraps ESP-IDF 5.5.4. New timer API (timerBegin/timerAlarm). Pin in platformio.ini. |
| **Build** | PlatformIO (espressif32) | **6.13.0** | Deterministic builds, CI-friendly. Pin the platform version. |
| **BLE** | NimBLE-Arduino | **2.5.0** | Dual-role (central + peripheral) confirmed on S3. Fix for `rc=519` in ≥2.4.0. **Do NOT use 2.3.x.** |
| **DAC** | MCP4922 | — | Dual 12-bit SPI, 1.22µA/step resolution, 4.5µs settling. Paired with MCP1501-20 (2.048V Vref). |
| **Op-amp** | OPA388 / OPA2388 | — | Auto-zero, ≤5µV offset. Eliminates dominant DC offset source. Best for V-to-I converter. |
| **H-bridge** | DRV8871 *(see Architecture note)* | — | 45V, built-in shoot-through protection + dead-time. SOT-8. ~$2.50. |
| **Isolation (power)** | B0515D-1WR3 | — | 5V → **±15V dual output**, 1W, 1000 VDC isolation. NOT B0515S (single-output). |
| **Isolation (signal)** | ISO7741 / Si8621 | — | Digital isolator for SPI + fault signals across barrier. |
| **LDO** | ME6211C33 | — | 100mV dropout. **NOT AMS1117** (1.1V dropout = brownout on LiPo). |
| **PCB** | KiCad 9.0.8 → JLCPCB | — | GPL-compatible. Do NOT use KiCad 10 (too new, plugin ecosystem not ready). |
| **HRV** | NeuroKit2 (Python) | **0.2.13** | Full HRV suite (RMSSD, pNN50, SDNN, LF/HF, DFA). MIT license. |
| **Filesystem** | LittleFS (built-in) | — | Wear-leveling, power-loss resilient. NOT SPIFFS (deprecated). |

**BOM:** ~$60/unit including 4-layer PCB fabrication. Under $100 target. Budget $20 contingency for TI parts (OPA2388, ISO7741 availability on LCSC is uncertain).

### Cross-Document Design Conflict — Resolved

STACK.md and ARCHITECTURE.md made **different component recommendations** in three areas. After synthesizing the evidence, here are the resolved decisions:

| Decision | STACK.md Said | ARCHITECTURE.md Said | **Resolved Choice** | Rationale |
|----------|---------------|---------------------|---------------------|-----------|
| **Current source topology** | Howland pump with OPA388 | V-to-I with Rsense feedback (NOT Howland) | **V-to-I + Rsense** | ARCHITECTURE's argument is correct: H-bridge already provides floating load, making Howland's main advantage moot. PITFALLS H-01 documents Howland oscillation risk with electrode capacitance. V-to-I is simpler and unconditionally stable. |
| **Op-amp** | OPA388 (for Howland) | OPA2277 (for V-to-I) | **OPA388 in V-to-I topology** | Use OPA388's ultra-low offset (≤5µV) in the simpler V-to-I topology. This gets the best of both: stability of V-to-I AND near-zero DC offset from auto-zero. The ±20µV offset of OPA2277 is acceptable but OPA388 provides 200× more margin against DC tissue damage (S-01). |
| **H-bridge** | Discrete BSS138/BSS84 MOSFETs | DRV8871 (integrated, 45V) | **DRV8871** | ARCHITECTURE correctly identified that DRV8837 (11V) is too low for compliance voltage. DRV8871 (45V) handles the full ±15V compliance. Built-in shoot-through protection eliminates PITFALLS S-06 by hardware. STACK's concern about motor-driver minimum pulse width was specific to DRV8833 blanking time; DRV8871 does not have this limitation at 250µs pulses. |

### Expected Features
*(from FEATURES.md)*

**Golden Parameters (clinical anchor):** 25 Hz, 200-250µs biphasic, 0.5-3 mA (sub-threshold), 30s ON / 30s OFF, 30 min session, cymba conchae placement, 20 ms binaural stagger.

**Must-Have (Table Stakes — v1):**
1. **TS-1** Charge-balanced biphasic waveform — safety non-negotiable
2. **TS-2** Constant-current output (0.1-5 mA, ≤0.2 mA resolution)
3. **TS-3** Hardware overcurrent cutoff (6 mA, firmware-independent)
4. **TS-4** DC-blocking capacitor per channel (10µF film + 1MΩ bleed)
5. **TS-5** Galvanic isolation (patient ↔ digital domain, ≥1000V)
6. **TS-6** Electrode impedance monitoring (pre-session + during)
7. **TS-7** Emergency stop / fault-safe to zero (hardware button + watchdog)
8. **TS-8** Dual independent channels (binaural)
9. **TS-9** Configurable parameters (via BLE or serial)
10. **TS-10** Session logging (timestamped, to LittleFS)

**Should-Have (Differentiators):**
- **D-1** Asynchronous binaural stagger (0-40 ms configurable) — *the core hypothesis*
- **D-2** Real-time HRV via Polar H10 (BLE dual-role) — *turns faith into data*
- **D-3** BLE GATT parameter control (phone as UI, no OLED needed)
- **D-4** Soft-start current ramp (30s default, configurable)
- **D-5** Per-channel current calibration
- **D-7** Charge density calculation + safety display

**Defer to v2+:**
- **D-6** Session scheduling / presets — convenience, not critical
- **D-9** Scope test point — PCB decision, zero firmware cost (include in PCB anyway)
- **D-10** OTA firmware updates — ship when firmware is stable and signed

**Anti-Features (explicitly excluded):**
- No tDCS/tACS capability. No current >5 mA. No frequency >100 Hz.
- No cloud sync. No proprietary mobile app. No on-device display.
- No stimulation while USB charging. No FDA/CE compliance attempt in v1.

### Architecture Approach
*(from ARCHITECTURE.md)*

The architecture follows a **safety-first** philosophy: *hardware enforces safety, firmware optimizes experience, software can fail — physics cannot.*

**Signal Chain (per channel):**
ESP32-S3 SPI → Digital Isolator → MCP4922 DAC → RC filter → OPA388 V-to-I converter (Rsense = 100Ω) → DRV8871 H-bridge → LM393 overcurrent latch → DC-blocking cap → Electrode

**Key Structural Decisions:**
1. **Dual-core isolation:** Core 0 = BLE + session management. Core 1 = stimulation ISR + impedance. NimBLE pinned to Core 0. Hardware timer ISR fires on Core 1. They never contend.
2. **Single hardware timer, dual channel:** Both channels driven from one 1µs-resolution timer. Channel B fires at `counter + stagger_offset`. Zero inter-channel drift. Deterministic stagger.
3. **Three-layer safety:** Layer 1 (Hardware): overcurrent comparator + latch, DC-blocking caps, heartbeat-gated power switch, VBUS lockout, E-stop button. Layer 2 (Firmware): RTC watchdog, charge balance counter, compliance monitor, session timeout, parameter validation. Layer 3 (State Machine): BOOT → SELF_TEST → READY → STIMULATING → SAFE_STOP → FAULT. Any state can reach SAFE_STOP in ≤50ms.
4. **Heartbeat-gated analog power:** ESP32 outputs a 100 Hz PWM "heartbeat." An RC circuit + comparator on the patient side detects if the heartbeat stops (MCU crash = no edges). Analog ±15V rail dies within 50ms. The DAC, op-amp, and H-bridge lose power. Zero current by physics, not by firmware.
5. **Compliance voltage:** ±15V rails yield 12.5V compliance after op-amp saturation and Rsense drop. Handles Z up to 2.5kΩ at 5mA or 4.2kΩ at 3mA. Above that → impedance fault, auto-pause. This matches all portable neurostim devices.
6. **4-layer PCB:** Mandatory. L1=signals, L2=ground plane (split: GND_DIGITAL | GND_ISO), L3=power planes, L4=signals. 2.5mm milled slot between ground domains. Isolator ICs and DC-DC straddle the barrier.

**Build Order:** 6 levels, ~10-12 week critical path. BLE (Level 4b) is parallelizable with dual-channel analog (Level 4a).

### Critical Pitfalls — Top 7
*(from PITFALLS.md, prioritized by severity and likelihood)*

| # | Pitfall | Severity | Core Mitigation |
|---|---------|----------|-----------------|
| 1 | **S-01: DC offset accumulation** — net DC from op-amp offset/timing asymmetry causes tissue burns over 30-min sessions | CRITICAL | 10µF film DC-blocking cap per channel (hardware). OPA388 auto-zero (≤5µV). Firmware charge balance counter. |
| 2 | **S-03: USB ground loop** — USB charger leakage current flows through patient when cable connected during use | CRITICAL | Galvanic isolation (B0515D + digital isolator). Hardware VBUS lockout that disables output when USB detected. Document: "NEVER connect USB while electrodes attached." |
| 3 | **S-05: Watchdog doesn't actually cut output** — MCU resets but DAC retains last code, analog stays powered | CRITICAL | Heartbeat-gated analog power (normally-OFF topology). DAC VDD gated through same heartbeat switch. On boot: DAC=0, H-bridge=coast, verify zero current via ADC before any user interaction. |
| 4 | **S-06: H-bridge shoot-through** — both MOSFET pairs ON simultaneously during transition, current spike to patient | CRITICAL | Use DRV8871 with built-in shoot-through protection + dead-time. If using discrete MOSFETs, use MCPWM peripheral with ≥500ns hardware dead-time. Never control H-bridge with independent `digitalWrite()`. |
| 5 | **S-07: Bilateral NTS overload** — simultaneous bilateral stim causes vasovagal syncope | CRITICAL | Firmware hard minimum stagger of 10ms (clamped even if user sets 0). Single timer drives both channels. Default 20ms. HR monitoring interlock: auto-reduce if HR drops >20 BPM. |
| 6 | **H-01: Howland pump oscillation** — electrode capacitance + feedback creates unstable pole at high impedance | HIGH | **Resolved: use V-to-I + Rsense instead of Howland.** Unconditionally stable. H-bridge provides the floating capability. |
| 7 | **F-01: BLE starves stimulation ISR** — BLE stack preempts pulse timing, causing jitter and charge imbalance | HIGH | Hardware timer ISR on Core 1 (non-preemptible). BLE pinned to Core 0. NimBLE (not Bluedroid). ISR completes in <5µs. |

---

## Implications for Roadmap

### Suggested Phase Structure

Research strongly supports **6 phases** with a hardware-then-firmware-then-integration flow. The dependency graph from ARCHITECTURE.md (§9) and the feature dependency tree from FEATURES.md (§Feature Dependencies) converge on this structure.

---

### Phase 1: TENS Hack — Clinical Parameter Validation
**Rationale:** Validate that the golden parameters (25 Hz, 200µs, sub-threshold, cymba conchae) produce measurable HRV changes *before* investing in custom hardware. This is a <$50, <2-week validation gate.
**Delivers:** Confirmed stimulation parameters. Electrode placement protocol. Personal perception threshold. Proof-of-concept HRV response.
**Features addressed:** Partial TS-1 (biphasic from TENS), partial TS-2 (constant-current from TENS), D-4 (manual ramp via TENS dial).
**Pitfalls to avoid:** S-01 (add external DC-blocking cap to TENS output), S-03 (no USB connection during use — battery-only TENS), C-01 (document cymba conchae placement explicitly), O-03 (write contraindications on day 1).
**Research needed:** None — well-documented patterns.

### Phase 2: ESP32-S3 Bringup + Single-Channel Breadboard
**Rationale:** Establish the firmware foundation (hardware timer ISR, SPI DAC driver, waveform state machine) and validate the analog signal chain on breadboard before any PCB commitment. This is the highest-risk validation phase.
**Delivers:** Working single-channel constant-current source on breadboard. Hardware timer ISR at 25 Hz with 1µs resolution. SPI DAC control. Biphasic waveform verified on oscilloscope. V-to-I accuracy ±0.1 mA across 1kΩ load.
**Features addressed:** TS-1, TS-2, TS-9 (via serial console), D-4 (soft-start ramp).
**Pitfalls to avoid:** S-02 (hardware timer ISR, NOT FreeRTOS timer — CRITICAL), F-03 (fixed-point ramp math, unit test edge cases), H-02 (use LDAC pin properly, add RC filter on DAC output).
**Key validation:** DAC linearity, V-to-I accuracy, biphasic charge balance (AC-coupled scope), pulse timing jitter <10µs.
**Research flag:** 🟡 ESP32-S3 MCPWM dead-time configuration if using discrete MOSFETs (not needed if using DRV8871).

### Phase 3: Safety Layers + Impedance + Isolation
**Rationale:** This is the critical safety phase. Add all hardware-enforced protections BEFORE adding dual-channel or BLE. The isolation boundary is the highest-risk integration point. Must be validated on breadboard before PCB.
**Delivers:** Single-channel prototype with full safety stack: overcurrent latch, DC-blocking cap, heartbeat-gated power, VBUS lockout, E-stop, impedance monitoring. Galvanic isolation verified (megohmmeter >10MΩ, leakage <1µA at 500V).
**Features addressed:** TS-3, TS-4, TS-5, TS-6, TS-7.
**Pitfalls to avoid:** S-01 (DC-blocking cap installed and tested), S-03 (isolation barrier verified), S-04 (impedance monitoring + compliance detection), S-05 (heartbeat gate tested: MCU reset → output dies in <50ms), S-06 (DRV8871 shoot-through protection verified on scope).
**Key validation:** ★ CRITICAL — SPI through isolator at 10-20 MHz (may need speed reduction). DC-DC noise on ±15V rails (<20mV after filtering). Isolation integrity. All safety layers independent.
**Research flag:** 🔴 Isolation SPI integrity and DC-DC noise are the highest-risk unknowns. Test BEFORE committing to PCB layout. May need to add linear post-regulation (reduces compliance by ~2V but acceptable).

### Phase 4: Dual Channel + Binaural Stagger + BLE
**Rationale:** With a safe single-channel prototype validated, duplicate the analog stage and add the binaural stagger timing. BLE development (GATT server + Polar H10 client) can be parallelized on a second dev board starting in Phase 2.
**Delivers:** Full dual-channel binaural stimulation with configurable 0-40ms stagger. BLE GATT server for phone parameter control. BLE central client connecting to Polar H10 HRS. RR interval logging to LittleFS.
**Features addressed:** TS-8, TS-10, D-1, D-2, D-3, D-5, D-7, D-8.
**Pitfalls to avoid:** S-07 (minimum 10ms stagger enforced), F-01 (BLE on Core 0, stim on Core 1), F-02 (impedance measurement during OFF periods only), F-04 (NimBLE ≥2.5.0, sequential connection strategy, 8KB stack).
**Key validation:** 30-minute soak test with both BLE connections active (phone + H10). No disconnections. RR intervals logged continuously. Both channels on scope showing correct stagger.
**Research flag:** 🟡 Polar H10 HRS notification format and MTU negotiation — needs hands-on testing with actual hardware.

### Phase 5: PCB Design, Fabrication, and Board Bringup
**Rationale:** Only commit to PCB after the full breadboard prototype (Phase 4) is validated. Transfer the proven circuit to KiCad 9.0.8. 4-layer board with isolation barrier slot.
**Delivers:** Custom 4-layer PCB with all components. JLCPCB fabrication. Board bringup repeating all Phase 2-4 tests on the PCB.
**Features addressed:** D-9 (scope test point — add to PCB layout, zero cost). Physical E-stop button. Status LED.
**Pitfalls to avoid:** H-03 (DC-DC module ≥20mm from analog ICs, aggressive LC filtering), H-04 (split ground planes, no traces crossing the split, analog traces <15mm), S-03 (2.5mm milled slot between ground domains).
**Key validation:** Isolation megohm test, all safety checklist items on PCB, BLE antenna performance, battery life measurement.
**Research flag:** 🟡 JLCPCB 4-layer pricing and SMT assembly flow for TI parts (OPA2388, ISO7741 may need Mouser sourcing). Verify B0515D-1WR3 stock at order time.

### Phase 6: Documentation, Firmware Polish, and Open-Source Release
**Rationale:** Community-ready release requires safety documentation, contraindications, electrode fabrication guide, assembly instructions, HRV measurement protocol, and Python analysis scripts.
**Delivers:** Complete GitHub release: firmware binary, KiCad files, BOM with alternates, assembly guide, safety document, contraindications, electrode BOM, HRV measurement protocol, Python analysis scripts with NeuroKit2.
**Features addressed:** D-6 (session presets), D-10 (OTA updates — only if firmware is stable and signed). All documentation.
**Pitfalls to avoid:** O-01 (single source of truth for parameters in `golden_params.h`), O-02 (all versions pinned in platformio.ini, release binaries provided), O-03 (contraindications on every surface: README, PCB silkscreen, firmware splash, BLE acknowledgment gate), O-04 (specific electrode BOM + characterization procedure), C-02 (titration protocol documented), C-03 (standardized HRV measurement protocol).
**Research needed:** None — standard documentation patterns.

### Phase Ordering Rationale

1. **Phase 1 → 2:** Clinical validation before firmware investment. If parameters don't produce HRV changes, re-evaluate before writing code.
2. **Phase 2 → 3:** Firmware + single-channel analog validated before adding safety layers. Easier to debug signal chain without isolation complexity.
3. **Phase 3 → 4:** All safety must work on a single channel before doubling the analog path. Isolation boundary is the highest-risk integration — resolve it early.
4. **Phase 4 → 5:** Full system validated on breadboard before PCB commitment. PCB errors are expensive and slow (2-week turnaround minimum).
5. **Phase 5 → 6:** Documentation written against the final hardware, not against aspirational designs.

**Parallelization opportunities:**
- BLE firmware (D-3, D-2) can be developed on a second ESP32-S3 dev board starting in Phase 2, running in parallel with Phases 2-3.
- Python HRV scripts (NeuroKit2) can be developed anytime after Phase 1 (use Polar H10 data from the TENS hack sessions).
- KiCad schematic design starts in Phase 2, iterates through Phases 3-4, finalized in Phase 5.
- Electrode design is mechanical/materials — independent of electronics, can run in parallel.

### Research Flags Summary

| Phase | Research Needed? | Topic |
|-------|-----------------|-------|
| Phase 1 | ✅ None | Well-documented TENS modification patterns |
| Phase 2 | 🟡 Maybe | MCPWM dead-time config (only if discrete MOSFETs; DRV8871 eliminates this) |
| Phase 3 | 🔴 Yes | Isolation SPI integrity at speed, DC-DC noise characterization — must be bench-tested |
| Phase 4 | 🟡 Maybe | Polar H10 BLE HRS protocol specifics — needs real hardware testing |
| Phase 5 | 🟡 Maybe | JLCPCB 4-layer SMT assembly flow for TI components |
| Phase 6 | ✅ None | Standard documentation patterns |

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| **Stack (firmware)** | HIGH | ESP32-S3 + Arduino-ESP32 3.3.8 + NimBLE 2.5.0 — all versions verified via GitHub API. NimBLE dual-role fix traced to exact source lines at tags 2.4.0 and 2.5.0. Timer API change from 2.x→3.x documented with working code examples. |
| **Stack (analog)** | HIGH | MCP4922, OPA388, DRV8871, INA181, LM393, MCP1501 — all commodity bioelectronics parts with stable datasheets. |
| **Stack (power)** | HIGH | Critical AMS1117 brownout issue identified and corrected (ME6211). B0515D vs B0515S correction made. Power budget calculated: ~170mA average, ~11h on 2000mAh cell. |
| **Features** | HIGH | Table stakes grounded in clinical literature (McCreery, IEC 60601, RESET-AF trial). Golden parameters have strong evidence convergence. Binaural stagger hypothesis is LOW-evidence (project's primary research question). |
| **Architecture** | HIGH | V-to-I + H-bridge is proven in neurostim literature. Three-layer safety architecture follows commercial device patterns. Dual-core task pinning is standard ESP32. Cross-document conflict (Howland vs V-to-I) resolved with clear rationale. |
| **Pitfalls** | HIGH | 7 critical + 4 high + 4 medium + 4 clinical pitfalls documented with specific mechanisms, tests, and mitigations. Phase-specific pitfall matrix maps each to its relevant build phase. |
| **BOM cost** | MEDIUM | $60 estimate based on training data pricing. OPA2388 and ISO7741 may be higher from Mouser vs LCSC. Budget $20 contingency. |
| **Component availability** | MEDIUM | B0515D-1WR3 intermittently constrained — have RB-0515D (RECOM) as backup. TI parts (OPA2388, ISO7741) limited on LCSC — may need Mouser/Digi-Key. |
| **BLE dual-role** | MEDIUM | Fix confirmed in source code, used in production (Meshtastic), but edge-case stability with specific phone BLE stacks needs real-device testing. iOS is aggressive about connection parameters. |
| **Insomnia protocol** | LOW-MEDIUM | Parameters extrapolated from cardiac/autonomic taVNS trials. Insomnia-specific RCTs are sparse. The binaural stagger hypothesis has zero published validation — this IS the experiment. |

**Overall confidence: HIGH** for building a safe, functional device. **LOW** for whether the binaural stagger hypothesis will show measurable superiority over synchronous bilateral stimulation.

### Gaps to Address During Planning/Execution

| Gap | When to Address | How |
|-----|----------------|-----|
| **SPI through isolator at speed** | Phase 3, before PCB | Breadboard test: 5 MHz → 10 MHz → 20 MHz with logic analyzer. Budget for 10 MHz max. |
| **DC-DC noise characterization** | Phase 3, before PCB | Scope measurement on breadboard. If >20mV ripple after LC filter, add LM317/LM337 post-regulation (−2V compliance). |
| **Polar H10 HRS notification format** | Phase 4, early | Purchase H10 immediately. Test BLE connection and RR interval parsing on a standalone dev board. |
| **DRV8871 pulse behavior at 250µs** | Phase 2, early | Verify clean 250µs pulses on scope through DRV8871 into 1kΩ load. Check dead-time at polarity transitions. |
| **Electrode fabrication** | Phase 1, parallel | Cymba conchae clip design is mechanical. Needs Ag/AgCl pellet + conductive gel. Document specific BOM. 3D-printable fixture recommended. |
| **KiCad 9 JLCPCB plugin** | Phase 5, pre-order | Verify JLCPCB integration before starting PCB layout. Test BOM export and assembly workflow. |
| **Real-world electrode impedance range** | Phase 1 | Measure on actual ears during TENS hack. Establishes impedance design targets for compliance voltage validation. |

---

## Sources

### Primary (HIGH confidence — verified via API or well-established standards)
- **NimBLE-Arduino:** GitHub API — releases 2.4.0/2.5.0, issue #1016, PR #1018, source code at tags (NimBLEDevice.cpp lines 921-923)
- **Arduino-ESP32:** GitHub API — release 3.3.8, timer API docs, RepeatTimer example
- **PlatformIO:** GitHub API — platform-espressif32 v6.13.0
- **ArduinoJson:** GitHub API — release 7.4.3
- **KiCad:** GitHub API — release 9.0.8
- **NeuroKit2:** PyPI API — release 0.2.13, GitHub HRV module structure
- **McCreery et al.** (1990): Charge density limits for neural stimulation — 30 µC/cm² [year corrected from 2010 per citation audit]
- **Shannon** (1992): k-factor model for stimulation tissue damage
- **IEC 60601-1:2005+AMD1:2012:** Patient isolation, leakage current limits
- **ANSI/AAMI NS4:2024:** TENS device output limits
- **Task Force ESC/NASPE** (1996): HRV measurement standards
- **Stavrakis et al.** (2020) RESET-AF, JACC: 20 Hz, 200µs, tragus, 1h — foundational taVNS parameters
- **Peuker & Filler** (2002) Clin Anat: Cymba conchae ABVN innervation density (~100%)

### Secondary (MEDIUM confidence — training data, multiple sources agree)
- MCP4922, OPA388, DRV8871, B0515D, ISO7741, INA181 datasheets — well-established parts
- ESP32-S3 FreeRTOS dual-core patterns — extensively documented by Espressif
- Nuerisym, Parasym, cerbomed NEMOS specifications — from product pages and clinical publications
- taVNS meta-analyses 2020-2024 — parameter convergence at 25 Hz / 200µs / cymba conchae

### Tertiary (LOW confidence — needs validation)
- Binaural asynchronous stagger efficacy — no published RCT. Project's primary hypothesis.
- Insomnia-specific taVNS protocol optimization — smaller evidence base than cardiac/epilepsy
- Component pricing and stock levels — verify at order time

---

*Research completed: 2025-07-14*
*Ready for roadmap: yes*
