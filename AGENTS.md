# AGENTS.md — AI Session Briefing for OpenBinaural-taVNS

> Read this file first. Every time. No exceptions.
> Written by the AI that built this project — for the AI that continues it.

---

## 1. What This Project Is

Open-source DIY **Binaural Transcutaneous Auricular Vagus Nerve Stimulation (taVNS)** device.
- **Owner:** SuitCatClub (`suitcatclub@gmail.com`)
- **Purpose:** Personal insomnia self-experimentation + open-source community reference design
- **Build path:** TENS unit hardware hack first → custom ESP32-S3 PCB
- **License:** GPL v3
- **Repo:** `https://github.com/SuitCatClub/taVNS-Binaural`

Start with `.planning/PROJECT.md` for full context. Then `.planning/ROADMAP.md` for current phase.

---

## 2. Critical Component Decisions (Corrected)

These errors were caught in research. Do NOT revert to the wrong parts.

| Component | ❌ WRONG | ✅ CORRECT | Why |
|-----------|----------|-----------|-----|
| LDO Regulator | AMS1117-3.3 | **ME6211C33M5G** | AMS1117 has 1.1V dropout — incompatible with LiPo (3.7V nom). ME6211 = 100mV dropout, 0.1µA standby, SOT-23-5. Source: AliExpress/LCSC |
| Isolated DC-DC | B0515S-1W | **PCN1-S5-D15-M-TR** | B0515S = single +15V only. V-to-I needs dual **±15V**. PCN1-S5-D15-M-TR (CUI) = dual ±15V, 1500VDC isolation (1 MOPP Type BF), SMD-8, unregulated (acceptable — V-to-I self-regulates). Source: Mouser/CUI |
| Boost converter | (missing) | **TPS61023 (SOT-563)** | LiPo max 4.2V never reaches PCN1's 4.5V min input. TPS61023: 0.5–5.5V→5V, 3.7A peak switch, 900nA quiescent, SOT-563 (easier hand-solder than WSON). DigiKey: ~$0.75 |
| Op-amp (V-to-I error amp) | OPA388 / AD8628ARZ | **ADA4522-2ARZ** | OPA388 and AD8628 max supply = 5–5.5V — cannot run on ±15V isolated rail. ADA4522-2ARZ = zero-drift, ±27.5V, dual SOIC-8, 2.7MHz GBW. DigiKey: ADA4522-2ARZ-ND |
| Isolated-side DAC LDO | (missing) | **TLV70450DBVR** | MCP4922 DAC sits on isolated side, needs 5V. Taps +15V rail. 24V input, 5V/150mA, SOT-23-5. DigiKey: TLV70450DBVRCT-ND |
| Digital Isolators | SI8622EC-B-IS ×N | **1× SI8380P-IU + 1× SI8622EC-B-IS** | 10 signals cross the barrier (8 forward + 2 bidirectional). SI8380P-IU = 8-ch all-forward (QSOP-20, push-pull, 2500Vrms) — all SPI + H-bridge signals. SI8622EC = 2-ch bidirectional for heartbeat/fault. 2 ICs total (was 3). |
| USB charging interlock | (missing) | **BSS138 N-MOSFET** | VBUS present → BSS138 pulls TPS61023 EN LOW → boost OFF → ±15V OFF → hardware-only stimulation lockout during charging |
| Current source topology | Howland current pump | **V-to-I converter** | V-to-I more stable into varying electrode impedance. Rsense inherently stabilizes the feedback loop. |
| H-Bridge | A4950ELJTR-T (40V) | **DRV8871DDAR** | 45V operating / 50V abs max. Integrated IPROPI current sense used for impedance monitoring. SOIC-8 PowerPAD. DigiKey: 296-43024-1-ND |
| DC-blocking cap | ECQ-E2475KF (250V, 26mm TH) | **Samsung CL31B106KBHNNNE** | 250V massively overrated (cap sees ≤13V). Replacement: 10µF/50V/X7R/1206 SMD. 700× smaller. 1/10 price of TDK equivalent. Source: LCSC/Mouser |
| Current sense resistor | YR1B100RCC (axial TH) | **Susumu RG2012P-101-B-T5** | Same value (100Ω/0.1%/25PPM) in 0805 thin-film SMD. AEC-Q200. DigiKey: RG2012P-101-B-T5CT-ND |
| DAC package | MCP4922-E/P (PDIP-14) | **MCP4922-E/SL (SOIC-14)** | Through-hole PDIP not suitable for final SMD PCB. Identical electrically. DigiKey: MCP4922-E/SL-ND |
| BLE Library | NimBLE-Arduino 2.3.x | **NimBLE-Arduino ≥2.4.0** (use 2.5.0) | 2.3.x has regression breaking ESP32-S3 dual-role BLE (peripheral + central simultaneously) |
| Timer API | Arduino-ESP32 2.x style | **Arduino-ESP32 3.x API** | Timer API changed in 3.x. Old tutorials will compile but behave incorrectly. Use RepeatTimer example. |
| PPTC Fuse | MF-MSMF010-2 (100mA hold) | **Yageo SMD0603B001TF (10mA hold)** | "010" in Bourns MF-MSMF naming = 0.10A = 100mA — far above device's 5mA working current. 10mA hold / 30mA trip, 0603 SMD. DigiKey: 13-SMD0603B001TFCT-ND |
| Battery voltage sense | (missing) | **2× 0603 resistor divider → ESP32 ADC** | Essential for low-battery lockout (refuse start <3.2V, ramp down <3.0V) and battery % BLE characteristic. Zero BOM cost. |

---

## 3. Safety Architecture (Non-Negotiable)

These constraints are absolute. Never remove, bypass, or defer them.

```
Patient ←→ [DC-Block Caps 10µF/50V] ←→ [Galvanic Isolation Barrier] ←→ [ESP32 + DAC]
              ↕                                    ↕
        [Hardware overcurrent]        [1× SI8380P-IU (8-ch forward) + 1× SI8622EC-B-IS (2-ch bidi)]
        [PPTC 10mA + LM339 latch]    [PCN1-S5-D15-M-TR isolated DC-DC, 1500VDC]
        [DRV8871 IPROPI contact check] [BSS138 USB charging interlock]
        [Heartbeat watchdog latch]
```

- **IEC 60601-1 Type BF** patient isolation — battery-powered device
- **DC-blocking capacitors** on ALL patient-facing outputs — hardware enforced, not firmware
- **Charge-balanced biphasic waveform** — enforced in both hardware and firmware
- **Charge density limit:** 30 µC/cm² per phase (McCreery 1990). At 5mA × 500µs = 2.5 µC/cm² — 12× safety margin
- **No stimulation while USB charging** — BSS138 + TPS61023 EN pin — hardware-only, firmware cannot override
- **Emergency cut-off** — hardware-level, not firmware-only
- **Electrode contact detection** — DRV8871 IPROPI + LM339 comparator. Abort if Z > 10kΩ or Z < 100Ω. **Must be implemented before any human use.**
- **Low-battery lockout** — refuse to start if VBAT < 3.2V; ramp down and stop if VBAT < 3.0V mid-session
- **DRV8871 VREF** — must be tied to VCC to disable internal current limiting. Two active current loops will fight and oscillate.

**Core assignment (FreeRTOS, never swap):**
- **Core 1** → Stimulation ISR (highest priority, hardware timer) + impedance monitor
- **Core 0** → BLE stack (NimBLE) + session manager + flash logger
- BLE must NEVER preempt the stimulation ISR

---

## 4. Clinical Preset Parameters (Validated)

All parameters reviewed against published literature. Do not change without citation.

All parameters reviewed against published literature. Do not change without citation. Intensity should always be titrated to perception threshold — never apply fixed current without a threshold-finding ramp first.

| Preset | Freq | PW | Current | Duty | Duration | Mode | Evidence Level |
|--------|------|----|---------|------|----------|------|----------------|
| Insomnia — 25Hz | 25Hz | 200µs | titrate | 30s/30s | 30min | 20ms stagger | RCT (NEMOS/Nurosym default) |
| Insomnia — 20Hz | 20Hz | 200µs | titrate | 30s/30s | 30min | 20ms stagger | RCT (Chinese insomnia studies, Jiao/Li/Fang) |
| RESET-AF | 20Hz | 200µs | threshold | Continuous | 60min | **L ear only** | RCT (Stavrakis 2020, 2023). ⚠️ Tragus in trial — cymba may be superior but is a protocol deviation |
| Anti-Inflammatory | 25Hz | 250µs | 1.5mA sub-threshold | 30s/30s | 60min | 20ms stagger | Pilot (Lerman 2016, TNF-α reduction) |
| Depression — Adjunctive | 20Hz | 200µs | titrate | 30s/30s 2×/day | 30min | **L ear only** | RCT N=160 (Fang/Rong 2016). Adjunctive only — not standalone treatment |
| Epilepsy — Adjunctive | 25Hz | 250µs | titrate | Continuous | 60min 3×/day | 20ms stagger | CE-marked (NEMOS), Bauer 2016 RCT |
| Stress / Acute Anxiety | 25Hz | 200µs | titrate | Continuous | 15min | 20ms stagger | Pilot RCT (Burger 2019, Szeska 2020) |
| HRV Optimization | 25Hz | 200µs | titrate | 30s/30s | 15min | 20ms stagger | Converging pilots. **Best test bed for binaural stagger hypothesis.** |
| Migraine Prevention | 1Hz | 250µs | titrate | Continuous | 20min | 20ms stagger | RCT (Straube 2015, NEMOS). Discrete pulses felt individually at 1Hz — warn user. |
| Dysautonomia/hEDS | 25Hz | 200µs | 0.5→1.0mA ramp | 30s/30s | 30min | 20ms stagger | **THEORETICAL — zero clinical evidence.** First session: 15min max. |
| Orofacial / Bruxism | 25Hz | 200µs | 1.5mA sub-threshold | 30s/30s | 30min | 20ms stagger | **RCT evidence (2025)** — Guzel et al. (N=40, bruxism), Percin et al. (N=50, TMD). ⚠️ Full-text params pending — review .planning/research/OROFACIAL-HYPERTONICITY.md before finalising |
| Exploration | user | user | user | user | user | user | Hard limits: ≤5mA, 1–100Hz, 50–1000µs, charge/phase ≤25µC |

**Corrected parameters (do not revert):**
- Anti-inflammatory: was 10Hz → corrected to **25Hz** (Lerman et al. 2016 used 25Hz; no human taVNS data at 10Hz)
- RESET-AF: was bilateral synchronous → corrected to **unilateral left ear** (per Stavrakis trials)
- Dysautonomia: was 1.0mA fixed → corrected to **0.5mA ramp** (hEDS sensory hypersensitivity)

**Stagger note:** The 20ms bilateral asynchronous stagger is this project's **novel experimental hypothesis**. It is NOT validated by any published study. Physiologically sound (NTS refractory period timing) but unconfirmed. Always label as experimental. The HRV Optimization preset is the designed test bed for this hypothesis.

**Session limits:** Max 3 sessions/day, max 4h total daily exposure (per NEMOS CE protocol). No hard block — firmware warning only. Minimum 30min between sessions.

---

## 5. Key Clinical Citations

| Paper | Year | Relevance |
|-------|------|-----------|
| Stavrakis et al. — RESET-AF | 2020 | Foundational AF taVNS RCT. JACC CE. 20Hz/200µs/left tragus. |
| Stavrakis et al. — TREAT-AF | 2023 | Follow-up multicenter RCT. Results nuanced — verify primary endpoint. |
| Lerman et al. | 2016 | Only published taVNS study showing TNF-α reduction. Used 25Hz. Mol Med. |
| Tracey KJ | 2002 | Foundational inflammatory reflex. Nature. |
| McCreery et al. | **1990** | 30 µC/cm² charge density limit. IEEE TBME. **Year is 1990, not 2010.** |
| Peuker & Filler | 2002 | Cymba conchae = ~100% ABVN innervation vs tragus ~45%. |
| Bretherton et al. | 2019 | Elderly pilot RCT. Autonomic balance improvement. |
| Fang / Rong et al. | 2016 | Depression RCT N=160. 20Hz/200µs/1mA, unilateral left, adjunctive. |
| Bauer et al. | 2016 | Epilepsy RCT. 25Hz/250µs. Supports NEMOS CE-marked parameters. |
| Burger et al. | 2019 | Stress/anxiety pilot RCT. 25Hz/200µs/continuous/15min. |
| Szeska et al. | 2020 | Stress reduction pilot. Parameters align with Burger 2019. |
| Straube et al. | 2015 | Migraine prevention RCT. **1Hz** — unique parameter, NEMOS device. |
| Yakunina et al. | 2017 | fMRI confirmation: cymba conchae produces strongest NTS/brainstem activation. |
| Napadow et al. | 2012 | Respiratory-gated stimulation enhanced analgesic effects. Future feature candidate. |
| Guzel et al. | 2025 | **First taVNS RCT for bruxism** (N=40). Significant masseter tone reduction + sympathovagal improvement. J Oral Rehabil. |
| Percin et al. | 2025 | auricular VNS for TMD RCT (N=50). Significant pressure pain threshold increases across orofacial muscles. Rev Assoc Med Bras. |
| Yamazaki et al. | 2008 | Preclinical: VNS → NTS → trigeminal nucleus caudalis (Sp5) pathway demonstrated. |
| Oshinsky et al. | 2014 | VNS reduces glutamate in trigeminal nucleus caudalis by ~70%. Preclinical. |

Full annotated bibliography: `.planning/research/REFERENCES.md` (41+ resolved refs, DOI links)

---

## 6. Project Roadmap (9 Phases)

| Phase | Name | Status | Key Deliverable |
|-------|------|--------|----------------|
| 1 | TENS Hack | **SCRAPPED** | Skipped — going directly to custom ESP32 build for real clinical parameters |
| 2 | Waveform Engine | **START HERE** | ESP32-S3 charge-balanced biphasic ISR, scope-verified |
| 3 | Firmware Safety | Not started | Bounds checking, watchdog, soft-start, fault-safe |
| 4 | Safety Hardware | Not started | Isolation barrier, DC-block, overcurrent, USB interlock |
| 5 | Dual-Channel | Not started | Binaural stagger, per-channel current, impedance sensing |
| 6 | BLE Interface | Not started | Phone control + Polar H10 HRV integration + OTA |
| 7 | Session Management | Not started | 6 presets, session logging, CSV export |
| 8 | Custom PCB | Not started | KiCad 4-layer PCB, BOM, JLCPCB fabrication |
| 9 | Documentation | Not started | Build guide, safety docs, clinical refs, GitHub release |

Current phase: check `.planning/STATE.md`

---

## 7. Self-Improvement

Update this file at the end of every session when:
- A component decision changes or a new correction is found
- A phase is completed: update the roadmap table status
- A new clinical finding adjusts a preset parameter
- A new pitfall is discovered during implementation
- Any "never do" rule is added or refined

---

## 8. Never Do

| ❌ Never | Why |
|----------|-----|
| Use AMS1117 as LDO | 1.1V dropout — dead on LiPo below 4.1V |
| Use B0515S (single rail) | Need dual ±15V for V-to-I |
| Power AD8628 / OPA388 from ±15V | Max supply 5–5.5V — instant destruction |
| Leave DRV8871 VREF floating or low | Internal current limit fights external V-to-I loop → oscillation. Tie VREF to VCC. |
| Use NimBLE < 2.4.0 | ESP32-S3 dual-role BLE regression |
| Skip galvanic isolation | Patient-facing circuitry always behind isolation barrier |
| Use ECQ-E2475KF on final PCB | 26mm through-hole cap — physically too large, 250V massively overrated |
| Use PDIP (through-hole) DAC | MCP4922-E/P not suitable for SMD PCB — use MCP4922-E/SL |
| Allow stimulation while USB charging | Ground loop through isolation barrier. BSS138 interlock is mandatory. |
| Allow stimulation without electrode contact check | Implement IPROPI-based impedance check before all sessions |
| Allow stimulation with VBAT < 3.0V | ±15V rail becomes unstable → unpredictable current output |
| Flash firmware (OTA) during active stimulation | Could crash stimulation ISR mid-pulse → asymmetric H-bridge state |
| Use electrodes < 0.3cm² contact area | Charge density risk: 5mA × 500µs / 0.3cm² = 8.3µC/cm² — approaching limit |

---

## 9. Full BOM Reference

Complete validated BOM for 2-channel binaural build: **`.planning/DESIGN-REVIEW.md`**
Clinical and device completeness review: **`.planning/CLINICAL-REVIEW.md`**
Orofacial/bruxism research: **`.planning/research/OROFACIAL-HYPERTONICITY.md`**
Tongue-palate/parafunctional research: **`.planning/research/TONGUE-PALATE.md`** (pending)

**BOM cost summary:** ~$77 for complete 2-channel build (within $100 target)

**Key distributor assignments:**
| Source | Parts |
|--------|-------|
| DigiKey | ADA4522-2ARZ, DRV8871DDAR, TLV70450DBVR, SI8380P-IU, SI8622EC-B-IS, MCP4922-E/SL, Susumu RG2012P-101-B-T5, SMD0603B001TF, BSS138, LM339DR |
| LCSC/AliExpress | ME6211C33M5G, TP4056X, Samsung CL31B106KBHNNNE, passives (0402/0603/0805), LiPo battery |
| Mouser/CUI | PCN1-S5-D15-M-TR, TPS61023 (SOT-563) |

---

## 8. Workflow Commands

```
gsd-plan-phase N     → Plan phase N (creates PLAN.md in .planning/phases/N/)
gsd-execute-phase N  → Execute planned phase N
gsd-progress         → Check current state and route to next action
gsd-verify-work      → Verify phase goal achievement
gsd-complete-milestone → Archive milestone 1, prepare v2
```

Config: `.planning/config.json` — `claude-opus-4.6`, `interactive`, `fine` granularity, all agents ON.
