# Feature Landscape: OpenBinaural-taVNS

**Domain:** DIY Binaural Transcutaneous Auricular Vagus Nerve Stimulation — Insomnia Protocol
**Researched:** 2025-07-11
**Primary Sources:** Training data from clinical literature (RESET-AF, taVNS meta-analyses 2020–2024, Nuerisym/Parasym/cerbomed NEMOS documentation, IEC 60601 safety standards, McCreery charge limits)
**Confidence Note:** No live web search was available. All findings derive from training data (cutoff ~2024). Confidence levels are assigned accordingly. Claims marked HIGH are well-established in peer-reviewed literature; MEDIUM reflects consensus across multiple training sources; LOW flags areas where my knowledge may be stale or thin.

---

## Golden Parameters for Insomnia Protocol

> **This section is the clinical anchor for every feature decision below.**

| Parameter | Recommended Value | Evidence Level | Source |
|-----------|------------------|----------------|--------|
| **Frequency** | **25 Hz** | HIGH — RCT-supported | RESET-AF (Stavrakis et al., JACC 2020) used 20 Hz; majority of 2020–2024 taVNS trials converged on 25 Hz. Nuerisym and cerbomed NEMOS both default to 25 Hz. |
| **Pulse width** | **200–250 µs** (charge-balanced biphasic) | HIGH — RCT-supported | RESET-AF: 200 µs. Most insomnia/autonomic trials: 200–250 µs. Wider pulses (>500 µs) recruit pain fibers — avoid. |
| **Current intensity** | **0.5–3 mA** (titrate to perception threshold, below pain) | HIGH — Consensus across trials | Sub-pain threshold maximizes afferent engagement without nociceptor recruitment. RESET-AF titrated to threshold. Nuerisym recommends 0.5–3 mA range. |
| **Duty cycle** | **30s ON / 30s OFF** | MEDIUM — Most common protocol | Standard across RESET-AF, TREAT-AF (Stavrakis et al., 2023), and most autonomic trials. Some depression protocols use continuous — not recommended for insomnia. |
| **Session duration** | **30–60 minutes** | MEDIUM — Varied across studies | 60 min in RESET-AF. 30 min sufficient for acute HRV effect in multiple pilot studies. Recommend 30 min for nightly use, 60 min option available. |
| **Session timing** | **30–60 min before sleep** | LOW — Theoretical + case reports | Parasympathetic enhancement is acute (onset within 5–15 min). Evening use aligns with chronobiology. Nuerisym case reports suggest bedtime use for insomnia. |
| **Electrode site** | **Cymba conchae** (both ears) | HIGH — Anatomical evidence | Peuker & Filler (2002): cymba conchae has highest ABVN innervation density (~100%). Tragus is ~45% ABVN. |
| **Waveform** | **Biphasic charge-balanced square wave** | HIGH — Safety standard | Eliminates net DC current. All commercial devices use this. Asymmetric biphasic acceptable if charge-balanced. |
| **Binaural stagger** | **20 ms inter-channel offset** (Channel B fires 20 ms after Channel A) | LOW — Theoretical | Not validated in any published RCT. Rationale: prevents simultaneous bilateral NTS saturation; mirrors natural vagal asymmetry. This is the project's primary hypothesis to test. |
| **Current ramp** | **0 → target over 30 seconds** | MEDIUM — Clinical practice standard | All commercial devices implement soft-start. Prevents startle response. cerbomed NEMOS ramps over ~20–30s. |

**Critical note on "Golden Parameters":** The 25 Hz / 200 µs / 0.5–3 mA / 30s ON-OFF / cymba conchae combination has the strongest evidence convergence across cardiac and autonomic taVNS literature. For insomnia *specifically*, the evidence base is smaller (mostly pilot studies and case series, not large RCTs). The parameters are extrapolated from autonomic/cardiac trials because the mechanism (parasympathetic enhancement via vagal afferents → NTS → dorsal motor nucleus) is the same.

---

## Table Stakes

Features users and researchers **expect**. Missing any of these = device is unsafe, non-reproducible, or unusable. These are non-negotiable.

### TS-1: Charge-Balanced Biphasic Waveform
| Attribute | Value |
|-----------|-------|
| **Why Expected** | Net DC current causes tissue damage (electrolysis, pH shift, electrode corrosion). Every clinical taVNS device uses charge-balanced output. This is the #1 safety requirement. |
| **Complexity** | Medium — Requires symmetric biphasic waveform generation with matched anodic/cathodic phases. H-bridge or dual-supply output stage. DC-blocking capacitor as hardware backup. |
| **Evidence** | HIGH — McCreery et al. (2010), Shannon (1992) charge density limits. IEC 60601-2-10. |
| **Dependencies** | Requires: TS-5 (galvanic isolation), TS-4 (DC-blocking hardware) |
| **Implementation Notes** | Symmetric biphasic (equal amplitude, equal duration, opposite polarity) is simplest and safest. Asymmetric biphasic (different amplitudes, adjusted durations for equal charge) is acceptable but adds complexity with no clinical benefit for this application. |

### TS-2: Constant-Current Output (0.1–5 mA range, ≤0.2 mA resolution)
| Attribute | Value |
|-----------|-------|
| **Why Expected** | Voltage-mode stimulation is unpredictable — output current varies with electrode impedance (skin moisture, contact pressure). All clinical taVNS devices are constant-current. Researchers need known current delivery for reproducibility. |
| **Complexity** | Medium-High — Howland current pump or improved Howland with H-bridge for biphasic. Requires adequate compliance voltage (>V_electrode = I × Z_load; at 5 mA × 2 kΩ = 10V minimum). |
| **Evidence** | HIGH — Universal standard in neurostimulation. Parasym, Nuerisym, NEMOS all use constant-current. |
| **Dependencies** | Requires: sufficient power supply headroom (±12–15V isolated), precision DAC (MCP4922 12-bit provides 1.2 µA steps at 5 mA full-scale — well within 0.1 mA resolution). |
| **Implementation Notes** | 0.1 mA resolution is achievable with 12-bit DAC. 0.5–3 mA is the clinical operating range; 5 mA maximum provides headroom for research exploration while remaining safe (charge density at 5 mA × 500 µs = 2.5 µC per phase, well under McCreery 30 µC/cm² limit for typical electrode areas). |

### TS-3: Hardware Overcurrent Protection (6 mA hard cutoff)
| Attribute | Value |
|-----------|-------|
| **Why Expected** | Software bugs cannot be the last line of defense for a device touching human tissue. A hardware comparator + MOSFET cutoff that kills output if current exceeds 6 mA is non-negotiable. Commercial devices all have hardware-level overcurrent. |
| **Complexity** | Low — Simple comparator (LM393 or similar) monitoring sense resistor, drives MOSFET gate to disconnect output. |
| **Evidence** | HIGH — IEC 60601-1 requires fault-safe design. McCreery limits require bounded current delivery. |
| **Dependencies** | Independent of firmware — must function even if MCU crashes. |
| **Implementation Notes** | 6 mA threshold provides margin above 5 mA clinical max. Sense resistor in output path (e.g., 10 Ω — 50 mV at 5 mA, 60 mV trip point). Resettable only by user action (power cycle or button), not automatic. |

### TS-4: DC-Blocking / Charge Accumulation Prevention
| Attribute | Value |
|-----------|-------|
| **Why Expected** | Even charge-balanced waveforms can develop DC offset from component tolerances, timing asymmetries, or firmware bugs. A series DC-blocking capacitor prevents net charge delivery to tissue under all fault conditions. |
| **Complexity** | Low — Series capacitor (10–47 µF electrolytics or film caps) in output path. Must handle peak voltage. |
| **Evidence** | HIGH — Standard practice in all clinical neurostimulation devices. |
| **Dependencies** | None — passive safety component. |
| **Implementation Notes** | Capacitor value must be large enough to pass the lowest-frequency stimulation without significant voltage droop. At 25 Hz, 200 µs pulse, 30s ON cycle: C > I × t / ΔV. 10 µF is conservative. Use bipolar film capacitor if possible (no polarity concern with biphasic). |

### TS-5: Galvanic Isolation (Patient Circuit ↔ Digital Ground)
| Attribute | Value |
|-----------|-------|
| **Why Expected** | USB-connected or charging device must not allow digital/mains ground faults to reach patient. Even in battery-only operation, isolation prevents ground loops and ensures the patient circuit is floating. |
| **Complexity** | Medium — Isolated DC-DC converter (B0515S-1WR3 or similar) + digital isolator (ADUM1200/Si8600) for SPI/control signals crossing isolation barrier. |
| **Evidence** | HIGH — IEC 60601-1 patient isolation requirements. All commercial devices isolate patient circuit. |
| **Dependencies** | Drives PCB layout partitioning (digital ground plane vs. patient ground plane). Must be designed from schematic stage. |
| **Implementation Notes** | 1500V isolation is minimum. B0515S-1W provides 1000 VDC — adequate for battery-powered device but verify datasheet. Consider reinforced isolation if device could be used while charging via USB (1500V+ required). Simplest safe rule: **disable stimulation during USB charging**. |

### TS-6: Electrode Impedance Monitoring
| Attribute | Value |
|-----------|-------|
| **Why Expected** | Detects: (1) electrode detachment (impedance → ∞, voltage rail saturation), (2) short circuit (impedance → 0, overcurrent), (3) poor contact (high impedance, uncomfortable stimulation). Users need feedback on electrode placement quality. |
| **Complexity** | Medium — Inject known small current, measure voltage across electrodes before session starts. During-session monitoring can use existing stimulation pulses. |
| **Evidence** | HIGH — cerbomed NEMOS has impedance check. Nuerisym has contact quality indicator. Standard feature in clinical devices. |
| **Dependencies** | Requires: ADC channel on MCU, calibrated measurement circuit. |
| **Implementation Notes** | Pre-session: inject 100 µA test pulse, measure voltage. Good contact: 500 Ω–5 kΩ. Alert if >10 kΩ (poor contact) or <100 Ω (short/metallic contact). During session: monitor compliance voltage — if hitting rail, impedance too high, auto-pause. |
| **Thresholds** | Abort: >10 kΩ or <100 Ω. Warn: >5 kΩ. Good: 500 Ω–5 kΩ. |

### TS-7: Emergency Stop / Fault-Safe to Zero
| Attribute | Value |
|-----------|-------|
| **Why Expected** | User must be able to instantly stop stimulation. Device must safe-fail to 0 mA on any detected fault (watchdog timeout, impedance out of range, firmware crash, power brown-out). |
| **Complexity** | Low-Medium — Physical button (hardware interrupt) + software watchdog timer + hardware watchdog. Output enable controlled by AND of all safety conditions. |
| **Evidence** | HIGH — Universal requirement. All commercial devices have immediate stop. |
| **Dependencies** | Integrates with TS-3 (overcurrent), TS-6 (impedance), firmware watchdog. |
| **Implementation Notes** | Watchdog timer: if firmware doesn't pet the watchdog every 100 ms, output stage is hardware-disabled. Physical button: direct GPIO to output enable MOSFET, not software-mediated. Fail-safe default: output OFF — output stage requires active assertion to deliver current. |

### TS-8: Dual Independent Channels (Binaural Capability)
| Attribute | Value |
|-----------|-------|
| **Why Expected** | The entire project premise is binaural stimulation. Two independent constant-current output stages, each with its own safety chain (overcurrent, DC block, impedance sensing). |
| **Complexity** | High — Doubles the analog output path. MCP4922 dual DAC helps, but need two independent current sources, two impedance sense channels, two DC-blocking caps, two overcurrent comparators. |
| **Evidence** | MEDIUM — Nuerisym is the only commercial bilateral device. Most literature is unilateral. Bilateral is the project's differentiating hypothesis. |
| **Dependencies** | Requires: dual-channel DAC (MCP4922), dual Howland current pump / V-to-I stage, dual safety chain. |
| **Implementation Notes** | Channels must be fully independent — separate current sources, not a switched single source. This allows asynchronous timing. Shared isolated power supply is acceptable (both channels share the ±15V rail). |

### TS-9: Configurable Stimulation Parameters
| Attribute | Value |
|-----------|-------|
| **Why Expected** | Researchers need to adjust frequency, pulse width, current, and duty cycle to replicate different protocols. Locked-down parameters are a dealbreaker for the research/biohacker audience. |
| **Complexity** | Low — Firmware parameter storage. BLE GATT characteristics for each parameter. |
| **Evidence** | HIGH — Differentiates from consumer-locked devices. The DIY/research community values configurability above all else. |
| **Dependencies** | Requires: BLE GATT server (D-3), parameter validation in firmware. |
| **Parameter Ranges** | Frequency: 1–100 Hz. Pulse width: 50–1000 µs. Current: 0.1–5.0 mA (per channel). Duty cycle: continuous to 50% (configurable ON/OFF times). Session duration: 1–120 min. |
| **Safety Bounds** | Firmware must enforce: charge per phase ≤ 25 µC (I × pulse_width). Current ≤ 5 mA. Frequency × pulse_width × 2 < 1 (no pulse overlap). These bounds are non-overridable. |

### TS-10: Session Logging (Local, Timestamped)
| Attribute | Value |
|-----------|-------|
| **Why Expected** | Researchers need records. Self-experimenters need to track what they did. Without logging, results are anecdotal and non-reproducible. |
| **Complexity** | Low — Log to ESP32-S3 flash (SPIFFS/LittleFS): session start/end time, parameters used, impedance readings, any fault events. |
| **Evidence** | MEDIUM — Parasym has app-based logging. Nuerisym logs via app. NEMOS has session counters. |
| **Dependencies** | Requires: RTC or NTP time source (BLE time sync from phone is simplest). Flash filesystem. |
| **Implementation Notes** | JSON or CSV format. Each session: timestamp, parameters (freq, pw, current_L, current_R, stagger_ms, duty_on, duty_off), impedance at start/end (L/R), HRV summary if available, fault events. Downloadable via BLE or USB serial. |

---

## Differentiators

Features that set this device apart from commercial offerings and other DIY builds. Not expected, but high-value for the target audience.

### D-1: Asynchronous Binaural Stagger (Pulse-Level Inter-Channel Offset)
| Attribute | Value |
|-----------|-------|
| **Value Proposition** | No commercial device offers configurable pulse-level asynchronous bilateral stimulation. Nuerisym is bilateral but synchronous (both channels fire simultaneously). The hypothesis: staggered pulses prevent simultaneous NTS saturation and reduce afferent adaptation, potentially increasing vagal tone more than synchronous bilateral. |
| **Complexity** | Medium — Timer-interrupt-driven waveform generation. Channel A fires at t=0, Channel B fires at t=offset (default 20 ms). Requires independent timer channels or phase-offset ISR scheduling on ESP32-S3 hardware timers. |
| **Evidence Level** | LOW — No published RCT validates asynchronous vs. synchronous bilateral taVNS. Rationale is theoretical (NTS convergence physiology + adaptation literature). This is the primary experimental hypothesis of the project. |
| **Dependencies** | Requires: TS-8 (dual channels), precise timer control (ESP32-S3 has 4 hardware timers — sufficient). |
| **Configuration** | Stagger offset: 0–40 ms (at 25 Hz = 40 ms period, max meaningful offset is ~half-period = 20 ms). 0 ms = synchronous mode (replicates Nuerisym). Configurable via BLE. |
| **Why This Matters** | This is the *reason the project exists*. If synchronous bilateral is equivalent, a Nuerisym is the better choice. The open design lets the community test the hypothesis with HRV data. |

### D-2: Real-Time HRV Monitoring via Polar H10 Integration
| Attribute | Value |
|-----------|-------|
| **Value Proposition** | No sub-$500 taVNS device includes objective efficacy measurement. Commercial devices are "stimulate and hope." This device connects to a Polar H10 chest strap (BLE HRS profile), captures RR intervals, and computes HRV metrics pre/during/post stimulation — turning every session into a mini-experiment. |
| **Complexity** | High — ESP32-S3 must act as BLE central (connecting to Polar H10) AND BLE peripheral (exposing GATT to phone) simultaneously. Dual BLE role is supported on ESP32-S3 but adds firmware complexity. |
| **Evidence Level** | HIGH (for HRV as vagal biomarker) — RMSSD and HF power are established indices of cardiac vagal tone (Task Force of the ESC/NASPE, 1996; Laborde et al., 2017). |
| **Dependencies** | Requires: BLE dual-role firmware, RR interval storage, HRV computation (can be on-device or offloaded to Python script). |
| **Key HRV Metrics** | **Primary:** RMSSD (root mean square of successive RR differences) — gold standard short-term vagal index. **Secondary:** pNN50 (% of successive RR intervals >50ms apart), HF power (0.15–0.40 Hz band), LF/HF ratio. **Tertiary:** SDNN (overall variability). |
| **Measurement Protocol** | 5-min baseline (pre-stim) → 30-min stimulation → 5-min recovery. Compare RMSSD pre vs. during vs. post. Clinically meaningful change: >20% increase in RMSSD suggests parasympathetic enhancement. |
| **Why This Matters** | Transforms the device from "faith-based stimulation" to "measured-response neurostimulation." The biohacker community will adopt this specifically because it provides objective feedback. |

### D-3: BLE Parameter Control (Phone as UI)
| Attribute | Value |
|-----------|-------|
| **Value Proposition** | Full parameter control via BLE GATT from any generic BLE app (nRF Connect, LightBlue). No proprietary app needed. Keeps BOM low (no OLED, no buttons beyond e-stop). Phone screen is a better interface than any on-device display. |
| **Complexity** | Medium — BLE GATT server with characteristics for: start/stop, current (L/R), frequency, pulse width, stagger offset, duty cycle, session duration. Read/write/notify for session status. |
| **Evidence Level** | N/A — Implementation decision, not clinical. |
| **Dependencies** | Requires: ESP32-S3 BLE stack (NimBLE or Bluedroid). Firmware parameter validation layer. |
| **Implementation Notes** | GATT service UUID: custom. Characteristics: current_mA (uint16, ×100 for 0.01 mA resolution), frequency_Hz (uint8), pulse_width_us (uint16), stagger_ms (uint8), duty_on_s (uint8), duty_off_s (uint8), session_min (uint8), channel_enable (bitfield), command (start/stop/pause), status (notify: running/paused/fault/complete), impedance_L/R (notify: uint16 Ω). |

### D-4: Soft-Start Current Ramp (Configurable 0 → Target)
| Attribute | Value |
|-----------|-------|
| **Value Proposition** | Prevents startle response and discomfort at session start. All commercial devices do this, but most DIY implementations skip it. Configurable ramp time differentiates from both. |
| **Complexity** | Low — Linear interpolation from 0 to target current over configurable period (default 30s). DAC output increases each stimulation cycle. |
| **Evidence Level** | MEDIUM — Standard clinical practice. cerbomed NEMOS ramps over ~20–30s. Parasym has gradual onset. |
| **Dependencies** | None beyond basic stimulation engine. |
| **Implementation Notes** | Ramp rate: target_mA / (ramp_time_s × frequency). At 25 Hz, 30s ramp, 2 mA target: increment = 2.0 / (30 × 25) = 0.00267 mA per pulse = ~3 µA per pulse. Well within 12-bit DAC resolution. Also implement ramp-down at session end (symmetric). |

### D-5: Per-Channel Current Calibration
| Attribute | Value |
|-----------|-------|
| **Value Proposition** | Left and right ear impedance differ. Perception threshold differs. Users should be able to set independent current levels per channel (e.g., left = 1.5 mA, right = 2.0 mA) to achieve symmetric subjective intensity. Commercial devices with bilateral capability (Nuerisym) appear to use a single intensity control. |
| **Complexity** | Low — MCP4922 dual DAC already provides independent channels. Just expose both in the BLE interface. |
| **Evidence Level** | LOW — No clinical study has evaluated per-channel intensity titration. Logical extension of bilateral stimulation. |
| **Dependencies** | Requires: TS-8 (dual channels), D-3 (BLE control). |

### D-6: Session Scheduling with Duty Cycle Presets
| Attribute | Value |
|-----------|-------|
| **Value Proposition** | Pre-programmed session profiles: "Insomnia Standard" (25 Hz, 200 µs, 2 mA, 30s/30s, 30 min), "RESET-AF Replication" (20 Hz, 200 µs, threshold, continuous, 60 min), "Exploration" (user-defined). Saves researchers from manually entering parameters every session. |
| **Complexity** | Low — Firmware stores N preset profiles in NVS (non-volatile storage). BLE interface includes profile select. |
| **Evidence Level** | N/A — UX feature. |
| **Dependencies** | Requires: D-3 (BLE control), NVS on ESP32-S3. |

### D-7: Charge Density Calculation and Display
| Attribute | Value |
|-----------|-------|
| **Value Proposition** | Real-time calculation of charge per phase (I × pulse_width) and charge density (charge / electrode_area) displayed to user. Safety-aware researchers want to know their charge density relative to McCreery limits. No commercial device shows this. |
| **Complexity** | Low — Arithmetic in firmware. Electrode area is a configurable constant (typical cymba conchae electrode: ~0.5–1.0 cm²). |
| **Evidence Level** | HIGH — McCreery et al. charge density limits are the established safety framework. |
| **Dependencies** | Requires: electrode area parameter, D-3 (BLE display). |
| **Implementation Notes** | Display: "Charge/phase: X.XX µC | Density: X.XX µC/cm² | Limit: 30 µC/cm²". Warn at >50% of limit. Hard-block at >80% of limit. |

### D-8: Open Data Export (CSV/JSON)
| Attribute | Value |
|-----------|-------|
| **Value Proposition** | All session data (parameters, impedance, HRV, events) exportable in standard formats. Enables: personal n=1 analysis, community data aggregation, publication-quality data. Commercial devices lock data in proprietary apps. |
| **Complexity** | Low — USB serial dump or BLE file transfer of stored session logs. Python companion script for parsing and visualization. |
| **Evidence Level** | N/A — Community value proposition. |
| **Dependencies** | Requires: TS-10 (session logging), D-2 (HRV data). |

### D-9: Waveform Verification Output (Scope Test Point)
| Attribute | Value |
|-----------|-------|
| **Value Proposition** | A buffered test point on the PCB that mirrors the output waveform — lets users verify charge balance, pulse timing, and current amplitude with an oscilloscope across a known load resistor. Builds trust in the device and enables community verification. |
| **Complexity** | Very Low — Buffer amplifier or voltage follower on a header pin. Sense resistor in output path serves double duty. |
| **Evidence Level** | N/A — Transparency feature. |
| **Dependencies** | PCB design stage. |

### D-10: Firmware OTA Updates (BLE DFU)
| Attribute | Value |
|-----------|-------|
| **Value Proposition** | Community can push firmware improvements without disassembling the device. ESP32-S3 supports OTA natively. Critical for a living open-source project. |
| **Complexity** | Medium — ESP32 OTA partition scheme. BLE DFU service (Nordic-style or ESP-IDF native). Signature verification to prevent malicious firmware. |
| **Evidence Level** | N/A — Community maintenance feature. |
| **Dependencies** | Requires: dual OTA partitions in flash layout, BLE DFU protocol implementation. |
| **Implementation Notes** | Must verify firmware signature before applying. Rollback capability if new firmware fails watchdog on first boot. Never interrupt stimulation for OTA — require session-stopped state. |

---

## Anti-Features

Things to **deliberately NOT build**. Including them would be unsafe, counterproductive, or scope-destructive.

### AF-1: Transcranial Stimulation Capability (tDCS/tACS)
| Reason to Avoid | Completely different safety profile, electrode placement, current paths, and regulatory landscape. Auricular taVNS and transcranial stimulation share almost nothing except "electricity on body." Adding tDCS capability invites users to run current through the brain with a device not designed for that. |
| What to Do Instead | Document clearly: "This device is designed ONLY for auricular stimulation. Do NOT use with head-mounted electrodes or attempt transcranial stimulation." |

### AF-2: Current Above 5 mA
| Reason to Avoid | No taVNS protocol uses >5 mA. The auricular branch is small — higher current recruits pain fibers (Aδ, C fibers) with no additional vagal afferent benefit. Allowing higher current invites tissue damage and electrode burns. |
| What to Do Instead | Hardware overcurrent cutoff at 6 mA. Firmware cap at 5 mA. No "expert mode" bypass. |

### AF-3: Continuous High-Frequency Stimulation (>100 Hz)
| Reason to Avoid | Frequencies >100 Hz are not used in any taVNS protocol. High-frequency continuous stimulation risks nerve fatigue, pain, and is outside all clinical evidence. The firmware should reject parameters that produce pulse overlap (freq × pulse_width × 2 ≥ 1.0). |
| What to Do Instead | Hard frequency cap at 100 Hz. Warning at >50 Hz ("outside clinical evidence for taVNS"). |

### AF-4: Cloud-Connected Data Sync
| Reason to Avoid | Personal neurostimulation data is profoundly sensitive. Cloud sync adds: privacy risk, internet dependency, server maintenance burden, and zero clinical benefit. The biohacker community is privacy-conscious. |
| What to Do Instead | All data local (on-device flash + local export to phone/PC). Open CSV/JSON format — users can sync to wherever they want using their own tools. |

### AF-5: Proprietary Mobile App
| Reason to Avoid | Developing and maintaining iOS + Android apps is a massive scope expansion with no proportional benefit. Generic BLE apps (nRF Connect, LightBlue) work today. A proprietary app becomes the maintenance bottleneck that kills open-source projects. |
| What to Do Instead | Well-documented BLE GATT interface. Python companion script for post-session HRV analysis. Community can build apps if demand warrants — the protocol is open. |

### AF-6: On-Device OLED Display / Complex UI
| Attribute | Value |
|-----------|-------|
| **Reason to Avoid** | Adds $5–10 BOM cost, firmware complexity (display driver, UI state machine), power draw, and PCB real estate — all for a worse interface than a phone screen. Every UI element on the device is a distraction from getting the stimulation engine right. |
| **What to Do Instead** | Minimal on-device indicators: 1 LED for power/status (green=ready, blue=stimulating, red=fault). Physical e-stop button. Everything else via BLE on phone. |

### AF-7: Automatic Threshold Detection ("Smart Intensity")
| Attribute | Value |
|-----------|-------|
| **Reason to Avoid** | Some devices claim to auto-detect perception threshold via impedance changes or user physiological responses. This is unreliable, adds complexity, and creates a false sense of safety. Perception threshold varies with electrode placement, skin condition, attention, and time — only the user knows when they feel stimulation. |
| **What to Do Instead** | Manual titration: user increases current slowly (via ramp + manual adjustment) until they feel tingling, then stays at or slightly below that level. Document the protocol clearly. |

### AF-8: Neck (Transcervical) Stimulation Mode
| Attribute | Value |
|-----------|-------|
| **Reason to Avoid** | Transcervical VNS (e.g., gammaCore) stimulates the cervical vagus trunk directly — much more powerful, different safety concerns (cardiac effects, laryngeal nerve proximity), and requires different electrode design. Mixing auricular and cervical modes in one device creates dangerous confusion. |
| **What to Do Instead** | Documentation: "Auricular only. Not for cervical/neck stimulation." Hardware: electrode connector and impedance thresholds designed for ear electrodes only. |

### AF-9: Rechargeable-While-Stimulating
| Attribute | Value |
|-----------|-------|
| **Reason to Avoid** | USB charging during stimulation creates a galvanic path from mains through USB charger → device → patient. Even with isolation, this is an unnecessary risk vector. All commercial neurostim devices either disable charging during use or use removable batteries. |
| **What to Do Instead** | Hardware interlock: stimulation output is disabled when USB power is detected. Charge between sessions. Document: "Do not stimulate while charging." |

### AF-10: Attempting FDA/CE Compliance in v1
| Attribute | Value |
|-----------|-------|
| **Reason to Avoid** | Medical device certification (FDA 510(k), CE MDR Class IIa) costs $100K–$500K, takes 1–3 years, and requires quality management systems (ISO 13485) incompatible with a volunteer open-source project. Attempting compliance poisons the design process with regulatory overhead. |
| **What to Do Instead** | Clear "NOT A MEDICAL DEVICE" disclaimer on all documentation, firmware splash, and PCB silkscreen. "For personal research use only." Design to be *safe* (which we are doing), but don't claim to be *certified*. |

---

## Feature Dependencies

```
SAFETY FOUNDATION (must be built first)
├── TS-3: Hardware overcurrent → independent, no firmware required
├── TS-4: DC-blocking caps → passive component, layout stage
├── TS-5: Galvanic isolation → PCB architecture decision
└── TS-7: Emergency stop → hardware button + watchdog

STIMULATION ENGINE (requires safety foundation)
├── TS-1: Charge-balanced waveform → requires TS-4, TS-5
├── TS-2: Constant-current output → requires TS-3, TS-5
├── TS-8: Dual channels → doubles TS-1 + TS-2
├── TS-6: Impedance monitoring → requires TS-2 (current source), ADC
└── TS-9: Configurable parameters → requires stimulation engine working

BINAURAL DIFFERENTIATION (requires stimulation engine)
├── D-1: Asynchronous stagger → requires TS-8
├── D-4: Soft-start ramp → requires TS-2
├── D-5: Per-channel calibration → requires TS-8, D-3
└── D-7: Charge density display → requires TS-9

CONNECTIVITY & DATA (parallel to stimulation engine after BLE basics)
├── D-3: BLE parameter control → requires ESP32-S3 BLE stack
├── D-2: HRV via Polar H10 → requires D-3 (BLE dual-role)
├── TS-10: Session logging → requires RTC/time, flash filesystem
├── D-8: Data export → requires TS-10
└── D-10: OTA updates → requires BLE, flash partitioning

PRESETS & POLISH (requires all above)
├── D-6: Session presets → requires TS-9, D-3
└── D-9: Scope test point → PCB design stage, no firmware dependency
```

---

## Commercial Device Comparison Matrix

| Feature | cerbomed NEMOS | Parasym | Nuerisym | **OpenBinaural (this project)** |
|---------|---------------|---------|----------|-------------------------------|
| **Channels** | 1 (unilateral) | 1 (unilateral) | 2 (bilateral) | 2 (bilateral) |
| **Electrode site** | Cymba conchae | Tragus | Cymba conchae | Cymba conchae |
| **Bilateral mode** | No | No | Synchronous | Synchronous + Asynchronous |
| **Configurable params** | Limited (current only, via clinician) | Limited (current + program select) | Limited (current, preset programs) | Fully open (freq, PW, current, duty, stagger) |
| **Current range** | 0.1–5 mA | 0–25 mA (tragus/TENS hybrid) | 0.5–5 mA (est.) | 0.1–5 mA |
| **HRV integration** | None | None | Via app (limited) | Native Polar H10 + full HRV metrics |
| **Data export** | Proprietary | Proprietary app | Proprietary app | Open CSV/JSON |
| **Price** | ~$700–800 | ~$500–700 | ~$500–800 | <$100 BOM |
| **Open source** | No | No | No | Yes (GPL v3) |
| **Impedance check** | Yes | Unknown | Yes | Yes |
| **Charge density display** | No | No | No | Yes |
| **OTA firmware** | No (clinic-managed) | Via app | Via app | Yes (BLE DFU) |

**Confidence note:** Commercial device specifications are from training data (product pages, clinical trial documentation, and user reports available through ~2024). Specific technical details (e.g., Parasym's exact current range, Nuerisym's internal architecture) may have changed. MEDIUM confidence overall.

---

## MVP Recommendation

### Must ship in v1 (TENS hack + firmware):
1. **TS-1** Charge-balanced biphasic waveform — safety non-negotiable
2. **TS-2** Constant-current output (0.5–5 mA) — reproducibility non-negotiable
3. **TS-3** Hardware overcurrent (6 mA cutoff) — safety non-negotiable
4. **TS-4** DC-blocking capacitor — safety non-negotiable
5. **TS-7** Emergency stop / fault-safe — safety non-negotiable
6. **TS-9** Configurable parameters (at minimum via serial console if BLE not ready)
7. **D-4** Soft-start ramp — comfort and safety

### Must ship in v2 (custom PCB):
8. **TS-5** Galvanic isolation — safety, requires PCB design
9. **TS-6** Impedance monitoring — electrode quality feedback
10. **TS-8** Dual channels — binaural capability
11. **D-1** Asynchronous stagger — the differentiating feature
12. **D-3** BLE parameter control — proper UI

### Must ship in v3 (full system):
13. **D-2** HRV via Polar H10 — efficacy measurement
14. **TS-10** Session logging — data capture
15. **D-8** Data export — community value
16. **D-5** Per-channel calibration — bilateral refinement

### Defer (nice to have):
- **D-6** Session presets — convenience, ship when stable
- **D-7** Charge density display — informational
- **D-9** Scope test point — PCB design decision, zero firmware cost
- **D-10** OTA updates — community maintenance, ship when firmware is stable

---

## Sources and Confidence Assessment

| Source | Type | Confidence | Notes |
|--------|------|------------|-------|
| Stavrakis et al. (2020) "RESET-AF" JACC | RCT | HIGH | 20 Hz, 200 µs, tragus, 1h daily. Foundational cardiac taVNS parameters. |
| Stavrakis et al. (2023) "TREAT-AF" follow-up | RCT | HIGH | Confirmed RESET-AF parameters in larger cohort. |
| Peuker & Filler (2002) Clin Anat | Anatomical study | HIGH | Cymba conchae ABVN innervation density. Foundational electrode placement rationale. |
| McCreery et al. (2010) IEEE TBME | Safety limits study | HIGH | Charge density limits for neural stimulation. 30 µC/cm² per phase. |
| Shannon (1992) IEEE TBME | Safety model | HIGH | k-factor model for stimulation-induced tissue damage. |
| Task Force ESC/NASPE (1996) Circulation | HRV standards | HIGH | RMSSD, SDNN, LF/HF definitions and measurement standards. |
| Laborde et al. (2017) Front Psychol | HRV review | HIGH | Practical guidelines for HRV measurement in research. |
| taVNS meta-analyses 2020–2024 (multiple) | Systematic reviews | MEDIUM | Parameter convergence: 25 Hz, 200 µs, cymba conchae. Specific insomnia evidence thinner than cardiac. |
| Nuerisym device documentation | Product info | MEDIUM | Bilateral taVNS device. Specifics from public product pages and clinical partner publications. |
| Parasym device documentation | Product info | MEDIUM | Unilateral (left tragus). Widely used in cardiac research. |
| cerbomed NEMOS documentation | Product info | MEDIUM | CE-marked auricular VNS. Cymba conchae. Used in epilepsy/depression RCTs. |
| Binaural asynchronous stagger rationale | Theoretical | LOW | No published validation. Project's primary research hypothesis. Rationale from NTS convergence physiology. |
| Insomnia-specific taVNS protocols | Pilot studies / case series | LOW-MEDIUM | Smaller evidence base than cardiac/epilepsy. Parameters extrapolated from autonomic modulation literature. |

**Overall Feature Landscape Confidence: MEDIUM** — Safety features and stimulation parameters are well-grounded in HIGH-confidence clinical literature. The binaural asynchronous differentiator is LOW-confidence (the entire point is to test it). Insomnia-specific protocol parameters are MEDIUM (extrapolated from autonomic/cardiac trials with strong mechanistic rationale but limited direct insomnia RCT evidence).
