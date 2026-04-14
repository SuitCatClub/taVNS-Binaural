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
## 4. Critical Component Decisions (Corrected)

These errors were caught in research. Do NOT revert to the wrong parts.

| Component | ❌ WRONG | ✅ CORRECT | Why |
|-----------|----------|-----------|-----|
| LDO Regulator | AMS1117-3.3 | **ME6211** | AMS1117 has 1.1V dropout — incompatible with LiPo (3.7V nom). ME6211 = 100mV dropout |
| Isolated DC-DC | B0515S-1W | **B0515D-1WR3** | B0515S = single +15V only. Howland pump / V-to-I needs dual **±15V**. -1WR3 = dual rail |
| BLE Library | NimBLE-Arduino 2.3.x | **NimBLE-Arduino ≥2.4.0** (use 2.5.0) | 2.3.x has regression breaking ESP32-S3 dual-role BLE (peripheral + central simultaneously) |
| Timer API | Arduino-ESP32 2.x style | **Arduino-ESP32 3.x API** | Timer API changed in 3.x. Old tutorials will compile but behave incorrectly. Use RepeatTimer example. |
| Current source topology | Howland current pump | **V-to-I converter (OPA388)** | Research resolved: V-to-I is more stable into varying electrode impedance |

---

## 5. Safety Architecture (Non-Negotiable)

These constraints are absolute. Never remove, bypass, or defer them.

```
Patient ←→ [DC-Block Caps] ←→ [Galvanic Isolation Barrier] ←→ [ESP32 + DAC]
              ↕                         ↕
        [Hardware overcurrent]   [Si8621/Si8622 digital isolators]
        [cutoff, H-bridge]       [B0515D-1WR3 isolated DC-DC]
                                 [1500VDC reinforced isolation]
```

- **IEC 60601-1 Type BF** patient isolation — battery-powered device
- **DC-blocking capacitors** on ALL patient-facing outputs — hardware enforced, not firmware
- **Charge-balanced biphasic waveform** — enforced in both hardware and firmware
- **Charge density limit:** 30 µC/cm² per phase (McCreery 1990). At 5mA × 500µs = 2.5 µC/cm² — 12× safety margin
- **No stimulation while USB charging** — ground loop risk. Hardware interlock required.
- **Emergency cut-off** — hardware-level, not firmware-only

**Core assignment (FreeRTOS, never swap):**
- **Core 1** → Stimulation ISR (highest priority, hardware timer) + impedance monitor
- **Core 0** → BLE stack (NimBLE) + session manager + flash logger
- BLE must NEVER preempt the stimulation ISR

---

## 6. Clinical Preset Parameters (Validated)

All parameters reviewed against published literature. Do not change without citation.

| Preset | Freq | PW | Current | Duty | Duration | Mode | Evidence |
|--------|------|----|---------|------|----------|------|----------|
| Insomnia — 25Hz | 25Hz | 200µs | 2.0mA | 30s/30s | 30min | 20ms stagger | Pilot RCT (NEMOS/Nuerisym default) |
| Insomnia — 20Hz | 20Hz | 200µs | 2.0mA | 30s/30s | 30min | 20ms stagger | Pilot RCT (Chinese insomnia studies) |
| RESET-AF | 20Hz | 200µs | threshold | Continuous | 60min | **L ear only** | RCT (Stavrakis 2020, 2023) |
| Anti-Inflammatory | 25Hz | 250µs | 1.5mA | 30s/30s | 60min | 20ms stagger | Pilot (Lerman 2016) |
| Dysautonomia/hEDS | 25Hz | 200µs | 0.5→1.0mA ramp | 30s/30s | 30min | 20ms stagger | **THEORETICAL — zero clinical evidence** |
| Exploration | user | user | user | user | user | user | N/A |

**Corrected parameters (do not revert):**
- Anti-inflammatory: was 10Hz → corrected to **25Hz** (Lerman et al. 2016 used 25Hz; no human taVNS data at 10Hz)
- RESET-AF: was bilateral synchronous → corrected to **unilateral left ear** (per Stavrakis trials)
- Dysautonomia: was 1.0mA fixed → corrected to **0.5mA ramp** (hEDS sensory hypersensitivity)

**Stagger note:** The 20ms bilateral asynchronous stagger is this project's **novel experimental hypothesis**. It is NOT validated by any published study. Always label it as experimental.

---

## 7. Key Clinical Citations

| Paper | Year | Relevance |
|-------|------|-----------|
| Stavrakis et al. — RESET-AF | 2020 | Foundational AF taVNS RCT. JACC CE. 20Hz/200µs/left tragus. |
| Stavrakis et al. — TREAT-AF | 2023 | Follow-up multicenter RCT. Results nuanced — verify primary endpoint. |
| Lerman et al. | 2016 | Only published taVNS study showing TNF-α reduction. Used 25Hz. Mol Med. |
| Tracey KJ | 2002 | Foundational inflammatory reflex. Nature. |
| McCreery et al. | **1990** | 30 µC/cm² charge density limit. IEEE TBME. **Year is 1990, not 2010.** |
| Peuker & Filler | 2002 | Cymba conchae = ~100% ABVN innervation vs tragus ~45%. |
| Bretherton et al. | 2019 | Elderly pilot RCT. Autonomic balance improvement. |

Full annotated bibliography: `.planning/research/REFERENCES.md` (41 resolved refs, DOI links)

---

## 8. Project Roadmap (9 Phases)

| Phase | Name | Status | Key Deliverable |
|-------|------|--------|----------------|
| 1 | TENS Hack | Not started | Modified TENS unit producing 0.5–5mA at ear electrodes |
| 2 | Waveform Engine | Not started | ESP32-S3 charge-balanced biphasic ISR, scope-verified |
| 3 | Firmware Safety | Not started | Bounds checking, watchdog, soft-start, fault-safe |
| 4 | Safety Hardware | Not started | Isolation barrier, DC-block, overcurrent, USB interlock |
| 5 | Dual-Channel | Not started | Binaural stagger, per-channel current, impedance sensing |
| 6 | BLE Interface | Not started | Phone control + Polar H10 HRV integration + OTA |
| 7 | Session Management | Not started | 6 presets, session logging, CSV export |
| 8 | Custom PCB | Not started | KiCad 4-layer PCB, BOM, JLCPCB fabrication |
| 9 | Documentation | Not started | Build guide, safety docs, clinical refs, GitHub release |

Current phase: check `.planning/STATE.md`
## 11. Workflow Commands

```
gsd-plan-phase N     → Plan phase N (creates PLAN.md in .planning/phases/N/)
gsd-execute-phase N  → Execute planned phase N
gsd-progress         → Check current state and route to next action
gsd-verify-work      → Verify phase goal achievement
gsd-complete-milestone → Archive milestone 1, prepare v2
```

Config: `.planning/config.json` — `claude-opus-4.6`, `interactive`, `fine` granularity, all agents ON.
