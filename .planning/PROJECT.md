# OpenBinaural-taVNS

## What This Is

An open-source, DIY Binaural Transcutaneous Auricular Vagus Nerve Stimulation (taVNS) device designed for personal insomnia self-experimentation and community sharing. The system delivers clinically-validated, charge-balanced biphasic stimulation to both auricular branches of the vagus nerve using a dual-path build strategy: first a modified commercial TENS unit for rapid validation, then a custom ESP32-S3-based PCB for the production-quality reference design. Efficacy is measured through HRV metrics via Polar H10 integration.

## Core Value

A fully open, clinically-grounded taVNS device that any technically-capable person can build, replicate, and verify — with firmware, hardware schematics, BOM, and HRV verification protocol all included.

## Requirements

### Validated

(None yet — ship to validate)

### Active

#### Phase 1 — TENS Hardware Hack
- [ ] **HW-01**: Commercial TENS unit modified to deliver constant-current output in 0.5–5mA range with ≤0.2mA resolution
- [ ] **HW-02**: DC-blocking capacitors and galvanic isolation implemented to prevent charge accumulation in tissue
- [ ] **HW-03**: Auricular electrode adapter (cymba conchae targeting) fabricated and validated for contact impedance <5kΩ

#### Phase 2 — Stimulation Engine (Firmware)
- [ ] **FW-01**: Charge-balanced biphasic square waveform generated at configurable frequency (1–100Hz) and pulse width (50–1000µs)
- [ ] **FW-02**: Soft-start current ramping (0→target over 30s) to prevent startle/discomfort
- [ ] **FW-03**: Binaural asynchronous stagger logic — Channel B fires with a configurable offset (default 20ms = half-period at 25Hz) after Channel A
- [ ] **FW-04**: Real-time electrode impedance sensing before and during sessions with automatic cutoff if impedance >10kΩ or <100Ω
- [ ] **FW-05**: Hardware watchdog + software emergency cutoff — device safe-fails to 0mA output on any fault
- [ ] **FW-06**: Session scheduler: configurable ON/OFF duty cycle (default 30s ON / 30s OFF), total session duration (default 30 min)

#### Phase 3 — Custom PCB
- [ ] **PCB-01**: ESP32-S3 microcontroller with dual-channel MCP4922 12-bit SPI DAC for independent channel waveform generation
- [ ] **PCB-02**: Precision constant-current output stage (Howland current pump or V-to-I with H-bridge) per channel
- [ ] **PCB-03**: Galvanic isolation between digital ground and patient ground (B0515S-1W isolated DC-DC + digital isolator)
- [ ] **PCB-04**: ±15V isolated analog rail from LiPo battery for compliance headroom (handles electrode impedance up to 2.4kΩ at 5mA)
- [ ] **PCB-05**: LiPo battery management (TP4056 USB-C charging + protection circuit)
- [ ] **PCB-06**: Complete KiCad schematic + PCB layout files published

#### Phase 4 — BLE Interface & HRV Logging
- [ ] **BLE-01**: BLE GATT server on ESP32-S3 exposes stimulation control (start/stop, current, frequency, pulse width, stagger offset)
- [ ] **BLE-02**: BLE client connects to Polar H10 (BLE HRS profile) and logs RR intervals to flash during session
- [ ] **BLE-03**: Post-session HRV report transmitted to phone: RMSSD, pNN50, LF/HF ratio, SDNN — pre/during/post comparison
- [ ] **BLE-04**: Python HRV analysis script for desktop post-processing and session visualization

#### Phase 5 — Documentation & Open-Source Release
- [ ] **DOC-01**: Clinical evidence summary: "Golden Parameters" table with citations (2020–2026 literature)
- [ ] **DOC-02**: Neurobiological rationale for asynchronous binaural stimulation documented
- [ ] **DOC-03**: Complete Assembly Guide (TENS hack + custom PCB build)
- [ ] **DOC-04**: Safety & Contraindications document (cardiac pacemakers, epilepsy, pregnancy, skin conditions)
- [ ] **DOC-05**: HRV verification protocol — baseline, stimulation, and recovery measurement guide
- [ ] **DOC-06**: GitHub release with versioned firmware, KiCad files, BOM, and documentation

### Out of Scope

- Invasive vagus nerve stimulation (VNS) — different modality, requires surgical implant
- Transcranial stimulation (tDCS/tACS) — different target, different safety profile
- CE/FDA medical device certification — for personal/research use only; compliance path is explicitly deferred
- Native mobile app (iOS/Android) — BLE control via a generic BLE terminal app (e.g., nRF Connect) is sufficient for v1; custom app is a future milestone
- Transcervical (neck) taVNS — auricular branch only for safety and reproducibility
- Cloud data sync — all data stays local on device + desktop

## Context

- **Clinical target**: Insomnia via vagal tone enhancement. Primary mechanism: increased parasympathetic activity (↑ RMSSD, ↑ HF power), reduction in sympathetic arousal. Key literature: RESET-AF trial parameters (25Hz, 200µs, 0.5–3mA), multiple HRV studies, and 2024 Nuerisym case reports.
- **Golden Parameters (current best evidence)**:
  - Frequency: 25Hz (most evidence; 10Hz for anti-inflammatory protocols)
  - Pulse width: 200–250µs (charge-balanced)
  - Intensity: 0.5–3mA (titrate to perception threshold, stay sub-pain)
  - Duty cycle: 30s ON / 30s OFF
  - Session: 30–60 minutes, once or twice daily
  - Electrode site: cymba conchae (highest auricular vagus density)
  - Binaural stagger: 20ms inter-channel offset (pulse-level asynchrony)
- **Safety charge density limit**: 30µC/cm² per phase (McCreery et al.). At 1mA × 250µs = 0.25µC per phase — well within safe limits even at 5mA × 500µs (2.5µC/cm²).
- **Build path**: TENS hack (Phase 1 MVP) → custom PCB (Phases 3+). TENS hack uses the existing high-voltage rail for compliance but adds precision current control and safety circuitry in the modification layer.
- **HRV measurement**: Polar H10 BLE chest strap (gold standard consumer HRV). Device acts as dual BLE role: peripheral (control) + central (H10 client).
- **Community target**: Technically capable biohackers, neuroscience researchers, and engineers. Documentation must be reproducible without expensive lab equipment.

## Constraints

- **Safety — Absolute**: No DC offset in electrode output. All outputs must be charge-balanced. Galvanic isolation mandatory. Hardware overcurrent cutoff at 6mA. These constraints cannot be relaxed.
- **Cost — BOM target**: TENS hack <$50 total; custom PCB BOM <$100 total (excluding PCB fabrication ~$20 for 5 boards at JLCPCB).
- **Components — Availability**: All components must be orderable from Mouser or Digi-Key. No obscure/EOL parts.
- **License — GPL v3**: All firmware, schematics, and documentation released under GPL v3. No proprietary dependencies.
- **Platform — ESP32-S3**: Firmware targets ESP32-S3. Arduino-ESP32 framework via PlatformIO.
- **Disclaimer — Not a medical device**: Device is for personal research use. All documentation must include clear contraindications and "not for medical use" disclaimer.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| ESP32-S3 over STM32 | BLE built-in eliminates external module; hardware timers sufficient for 250µs precision; larger Arduino community | — Pending |
| Arduino-ESP32 + PlatformIO over ESP-IDF | ISR latency difference (~2µs) is clinically irrelevant at 250µs pulse width; faster iteration | — Pending |
| Dual-path build (TENS hack → custom PCB) | TENS hack validates clinical parameters with minimal investment before committing to PCB design | — Pending |
| Polar H10 over PPG sensor on device | Gold standard RR interval accuracy; avoids motion artifact; BLE already present on ESP32-S3 | — Pending |
| BLE-only UI (no OLED) | Reduces BOM cost; phone is a better interface for parameter tuning; OLED adds complexity with no clinical benefit | — Pending |
| Asynchronous binaural (20ms stagger) over synchronous | Prevents simultaneous saturation of both NTS inputs; literature supports reduced adaptation; mirrors natural asymmetric vagal activity | — Pending |
| MCP4922 12-bit dual SPI DAC | 0.1mA resolution at 5mA full scale; SPI bus shared with other peripherals; cost ~$3 | — Pending |
| GPL v3 | Copyleft ensures community improvements stay open; prevents commercial capture of safety-critical medical device design | — Pending |

---
*Last updated: 2026-04-14 after initialization*
