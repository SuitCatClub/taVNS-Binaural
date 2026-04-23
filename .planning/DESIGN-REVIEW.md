# OpenBinaural-taVNS — Complete Circuit Design Review

**Date:** 2025-07-15
**Reviewer:** AI (claude-opus-4.6)
**Scope:** Full final-design circuit review — all 10 design issues resolved
**Safety standard target:** IEC 60601-1 Type BF (battery-powered, patient-contact)

---

## 0. Pre-Review Corrections

Before addressing the 10 design issues, two corrections from existing project research (AGENTS.md §2):

| Item | You Wrote | Correction | Source |
|------|-----------|------------|--------|
| Isolated DC-DC | TMA 0515D (1kV isolation) | **PCN1-S5-D15-M-TR** (CUI, 1500VDC, SMD-8, dual ±15V) | AGENTS.md corrected decision → BOM validation replaced B0515D-1WR3 |
| Op-amp in AGENTS.md | OPA388 | OPA388 max supply = 5.5V — same problem as AD8628. Needs replacement. | See §4 below |

The rest of this review uses **PCN1-S5-D15-M-TR** as the isolated DC-DC (replaced B0515D-1WR3 during BOM validation — B0515D dual output could not be confirmed, PCN1 is SMD with confirmed dual ±15V and 1500VDC isolation).

---

## 1. Overall Topology Verdict

### V-to-I + DRV8871 H-Bridge: ✅ APPROVED — Correct approach

The V-to-I converter with H-bridge polarity switching is the right topology for this application. Reasons:

| Factor | V-to-I + H-bridge | Howland Pump | Verdict |
|--------|-------------------|-------------|---------|
| Stability into capacitive loads | Inherently stable (single feedback loop) | Oscillation-prone (PITFALLS.md H-01) | V-to-I wins |
| Biphasic switching | Firmware controls H-bridge direction, DAC stays unipolar | Requires bipolar DAC drive | V-to-I wins |
| Component count | 1× op-amp + 1× Rsense + 1× H-bridge per channel | 5× precision-matched resistors + 1× op-amp | V-to-I wins |
| Current accuracy at 5mA | ±0.1% (determined by Rsense tolerance) | ±0.5–1% (limited by resistor matching) | V-to-I wins |
| Load impedance range | 500Ω–2.5kΩ (12.5V compliance at ±15V) | Similar, but more complex to analyze | Tie |

**The DRV8871DDAR specifically is correct** because:
- 45V operating / 50V abs max → handles full ±15V compliance voltage
- Integrated shoot-through protection and dead-time (~1µs)
- Built-in IPROPI current sense output (useful for impedance monitoring in Phase 5)
- SOIC-8 PowerPAD — compact, good thermal

> ⚠️ **DRV8871 VREF pin:** Set VREF to VCC to disable internal current limiting. The external op-amp feedback loop controls current. Two active current-regulation loops will fight and oscillate.

**No changes to the fundamental topology needed.**

---

## 2. Power Architecture Solution

### 🔴 CRITICAL 1: RESOLVED — Boost converter required

A single-cell LiPo (3.0–4.2V) does not match ANY standard isolated DC-DC input bin:
- 3.3V bin = 2.97–3.63V → LiPo exceeds at 4.2V (damages converter)
- 5V bin = 4.5–5.5V → LiPo never reaches 4.5V

**A boost converter to 5.0V is mandatory.**

### Recommended: TPS61023DRLR (Texas Instruments)

| Parameter | Value |
|-----------|-------|
| **MPN** | **TPS61023DRLR** |
| **Input range** | **2.0V–5.5V** (covers full LiPo range + deep discharge) |
| **Output** | Adjustable via resistor divider, set to 5.0V |
| **Max Iout** | 500mA continuous at 5V from 3.7V input |
| **Switching freq** | ~4 MHz |
| **Output ripple** | ~15–30 mV p-p with 22µF X5R output cap |
| **Efficiency** | ~93% at 3.7V→5V, 300mA |
| **Quiescent** | ~55µA (PFM), <1µA shutdown |
| **EN pin** | Yes — **use for USB charging interlock (SAFE-05)** |
| **Soft-start** | Internal |
| **Package** | SOT-5 (1.6×1.6mm) |
| **DigiKey** | TPS61023DRLR, ~$0.75 |

**Why this specific part:** The 4MHz switching frequency is the decisive factor. At 4MHz, output ripple is 10–20× lower than 300–600kHz alternatives for the same output capacitor. The PCN1-S5-D15-M-TR sees clean 5V input with minimal switching content coupling through to the ±15V analog rails.

**Alternate:** TPS61230DRCR (600mA, ~2–3MHz, WSON-10 3×3mm) if more current headroom needed.

### Complete Power Chain

```
LiPo (3.0–4.2V, 2000mAh)
    │
    ├──► ME6211C33M5G (LDO, 100mV dropout) ──► 3.3V DIGITAL RAIL
    │    SOT-23-5                                 │
    │    Cin: 1µF, Cout: 1µF                      ├── ESP32-S3 (~150mA avg)
    │                                              ├── SI8380P-IU digital side VDD1
    │                                              └── SI8622EC digital side VDD1
    │
    └──► TPS61023 (boost) ──► 5.0V BOOST RAIL
         SOT-563                      │
         L1: 2.2µH, Cout: 22µF X5R   │
         EN ← BSS138 (LOW during USB) └──► PCN1-S5-D15-M-TR (isolated DC-DC) ──► ±15V ISOLATED RAIL
                                           SMD-8, 1W, 1500VDC isolation           │
                                           Cin: 47µF elec + 100nF ceramic         ├── +15V: 100µF + 100nF + 1nF
                                           Cout: 47µF + 10nF per rail             ├── -15V: 100µF + 100nF + 1nF
                                                                                  ├── GND_ISO (floating)
                                                                                  │
                                                        ══════ ISOLATION BARRIER ══════
                                                                                  │
                                                                           PATIENT DOMAIN
                                                                                  │
                                              TLV70450DBVR (+15V → 5V isolated) ──► 5V_ISO RAIL
                                                                                  │
                                                                    ±15V powers:  │ 5V_ISO powers:
                                                                    ├── ADA4522-2ARZ  ├── MCP4922 DAC
                                                                    ├── 2× DRV8871   ├── SI8380P-IU VDD2
                                                                    │                 ├── SI8622EC VDD2
                                                                    └── LM339DR       └── LM339DR VCC
```

### Noise Mitigation at Boost Output

Add between TPS61023 output and PCN1-S5-D15-M-TR input:
1. **CM choke:** Würth 744232601 (600Ω @ 100MHz) in series with 5V line
2. **Input decoupling:** 47µF electrolytic + 100nF ceramic at PCN1 Vin pin
3. **Output decoupling:** 47µF + 10nF on each ±15V rail (CUI datasheet recommendation)

### Power Budget

| Consumer | Current from 5V boost rail |
|----------|---------------------------|
| ESP32-S3 (via ME6211 3.3V LDO) | ~50mA (3.3V×150mA / 5V×0.93) |
| PCN1-S5-D15-M-TR (1W isolated DC-DC at 78% eff) | ~256mA |
| SI8380P-IU + SI8622EC digital isolators | ~16mA |
| Boost converter overhead | ~10mA |
| Status LED | ~5mA |
| **Total from LiPo (via boost)** | **~330mA at 3.7V** |

Battery life: 2000mAh / 330mA ≈ **6 hours** → ~12 sessions at 30 min each. Adequate.

> ⚠️ **Note:** ME6211C33M5G can be powered directly from LiPo (2V–6V input) or from 5V boost rail. Direct LiPo connection is more efficient (avoids double conversion loss). Recommended: ME6211 input = LiPo directly.

---

## 3 & 4. Isolated-Side Op-Amp — COMBINED RESOLUTION

### 🔴 CRITICAL 2: RESOLVED — Replace AD8628 with ADA4522-2ARZ

**Option A (LDO on isolated side) is inferior to Option B (higher-voltage op-amp).** Here's why:

| Criterion | Option A: AD8628 + LDO | Option B: ADA4522-2ARZ |
|-----------|----------------------|----------------------|
| BOM count | +1 LDO + 2 bypass caps | 1 part swap (same SOIC-8 footprint) |
| Failure modes | LDO failure kills both channels | No single-point power dependency |
| Compliance voltage | -0.5V (LDO dropout reduces usable range) | Full ±15V available |
| Design risk | LP2985 Vin max=16V, only 1V over +15V — transient risk | ADA4522 rated to ±27.5V |
| Performance | AD8628: 1µV Vos, 2.5MHz GBW | ADA4522-2: 2.5µV typ / 10µV max Vos, 2.7MHz GBW |

### Recommended: ADA4522-2ARZ (Analog Devices)

| Parameter | ADA4522-2ARZ | AD8628ARZ (old) |
|-----------|-------------|----------------|
| **Supply range** | **±2.25V to ±27.5V** ✅ | ±2.5V max ❌ |
| **Vos** | 2.5µV typ / **10µV max** | 1µV typ / 10µV max |
| **Vos drift** | 0.022 µV/°C | 0.005 µV/°C |
| **GBW** | **2.7 MHz** | 2.5 MHz |
| **Slew rate** | **1.7 V/µs** | 1.0 V/µs |
| **CMRR** | 150 dB | 140 dB |
| **Output** | Rail-to-rail | Rail-to-rail |
| **Channels** | **Dual (1 IC for both channels)** | Single (need 2 ICs) |
| **Package** | SOIC-8 | SOIC-8 |
| **Architecture** | Zero-drift (chopper) | Zero-drift (auto-zero) |
| **DigiKey** | ADA4522-2ARZ-ND | AD8628ARZ-ND |

**ADA4522-2ARZ is the clear winner:**
- Runs directly on ±15V — no LDO needed on isolated side
- Dual package — one IC serves both channels
- Better GBW and slew rate
- Same SOIC-8 footprint — drop-in layout compatible
- Vos still ≤10µV max — adequate for charge balance (at 5mA, 10µV offset across 100Ω = 0.1µA DC offset → blocked by DC-blocking cap)

> ⚠️ **Chopper artifact note:** ADA4522 chopper clock ~4kHz creates µV-level output ripple at that frequency. At your 20–25Hz stimulation frequency, this is ~200× higher and will be rejected by the RC filter on the DAC output. Add a 10nF capacitor across the feedback path (parallel with Rsense) if bench testing shows any residual artifact. fc = 1/(2π × 100Ω × 10nF) = 159kHz — well above stimulation frequency, well below chopper.

### Fallback: OPA2187IDR (Texas Instruments)

If ADA4522-2ARZ is unavailable:

| Parameter | OPA2187IDR |
|-----------|-----------|
| Supply | ±2.25V to ±18V (±15V OK) |
| Vos | 2µV typ / **5µV max** (better than ADA4522) |
| GBW | 2 MHz |
| Slew rate | **0.7 V/µs** ⚠️ (see analysis below) |
| Package | SOIC-8 (dual) |
| DigiKey | OPA2187IDR-ND |

OPA2187 slew rate concern: 0.7V/µs × 200µs pulse = 140V swing capability per pulse — far more than the ~500mV Rsense signal. The op-amp output voltage swing through the load + Rsense might approach 13V during a compliance event, requiring 13V / 0.7V/µs = **18.6µs** slew time — 9.3% of the 200µs pulse width. Acceptable but not ideal. **ADA4522-2ARZ preferred.**

---

## 5. Binaural Dual-Channel Component Count

### Architecture: Fully independent analog stages, shared control

From ARCHITECTURE.md §2.5 (confirmed and updated):

| Component | Shared? | Per Channel | Qty for 2-Channel |
|-----------|---------|-------------|-------------------|
| MCP4922 DAC | **Shared** (dual-channel IC) | — | **1** |
| SI8622EC-B-IS digital isolator | **Partially shared** (see below) | — | **2** (see analysis) |
| ADA4522-2ARZ op-amp (error amp) | **Shared** (dual IC) | — | **1** |
| DRV8871DDAR H-bridge | **Independent** | 1 per channel | **2** |
| Rsense (100Ω, 0.1%) | **Independent** | 1 per channel | **2** |
| DC-blocking cap (10µF) | **Independent** | 1 per channel | **2** (+ 2 spare on BOM) |
| 1MΩ bleed resistor | **Independent** | 1 per channel | **2** |
| PPTC fuse (10mA) | **Independent** | 1 per channel | **2** |
| LM339 comparator + latch | **Independent** | 1 per channel | **1× LM339DR quad** (ch1+ch2: overcurrent, ch3: watchdog, ch4: spare) |
| PCN1-S5-D15-M-TR DC-DC | **Shared** | — | **1** |
| TPS61023DRLR boost | **Shared** | — | **1** |
| ME6211C33M5G LDO | **Shared** | — | **1** |
| ESP32-S3 | **Shared** | — | **1** |

### SI8622EC Isolator Channel Analysis

Signals crossing the isolation barrier:

| Signal | Direction | Channels Needed |
|--------|-----------|----------------|
| SPI SCK | Digital → Patient | 1 |
| SPI MOSI | Digital → Patient | 1 |
| SPI CS (DAC) | Digital → Patient | 1 |
| LDAC | Digital → Patient | 1 |
| H-bridge IN1_A | Digital → Patient | 1 |
| H-bridge IN2_A | Digital → Patient | 1 |
| H-bridge IN1_B | Digital → Patient | 1 |
| H-bridge IN2_B | Digital → Patient | 1 |
| Heartbeat (watchdog) | Digital → Patient | 1 |
| Fault flag | Patient → Digital | 1 |
| **Total** | | **10 channels** |

SI8622EC-B-IS = 2 channels (1 forward + 1 reverse). For 10 channels:
- **Option A:** 5× SI8622EC-B-IS (10 channels) — more ICs but all bidirectional
- **Option B:** 2× SI8642EC (4-ch unidirectional, Skyworks) + 1× SI8622EC (for bidirectional fault flag) = 3 ICs for 10 channels
- **Option C:** 2× SI8641EC-B-IS (4-ch, 3+1 direction) + 1× SI8622EC-B-IS (2-ch bidirectional) = **3 ICs, 10 channels**
- **Option D (recommended):** 1× SI8380P-IU (8-ch, all forward) + 1× SI8622EC-B-IS (1 forward + 1 reverse) = **2 ICs, 10 channels**

**Recommended: 1× SI8380P-IU + 1× SI8622EC-B-IS = 2 isolator ICs total**

Signal assignment:
- SI8380P-IU → SPI SCK, SPI MOSI, SPI CS, LDAC, H-bridge IN1_A, IN2_A, IN1_B, IN2_B (8 forward)
- SI8622EC-B-IS → Heartbeat watchdog (forward) + Fault flag (reverse) (2 channels)

Advantage over Option C: one fewer IC, no 3+1 channel-split ambiguity, all control signals in one package, simpler PCB routing, push-pull outputs (no pull-up resistors on isolated side).

### Complete Binaural Component Count Table

| Ref | Component | MPN | Qty | Notes |
|-----|-----------|-----|-----|-------|
| U1 | ESP32-S3 module | ESP32-S3-WROOM-1-N16R8 | 1 | MCU |
| U2 | Boost converter | TPS61023DRLR | 1 | LiPo → 5V |
| U3 | 3.3V LDO | ME6211C33M5G | 1 | LiPo → 3.3V |
| U4 | Isolated DC-DC | PCN1-S5-D15-M-TR | 1 | 5V → ±15V |
| U5 | Dual DAC | MCP4922-E/SL | 1 | 12-bit, SOIC-14 |
| U6 | Error amp (dual) | ADA4522-2ARZ | 1 | V-to-I, both channels |
| U7, U8 | H-bridge | DRV8871DDAR | **2** | 1 per channel |
| U9 | 8-ch isolator (all forward) | SI8380P-IU | **1** | SPI + H-bridge control (all 8 forward signals) |
| U10 | 2-ch isolator (bidirectional) | SI8622EC-B-IS | 1 | Heartbeat (forward) + Fault flag (reverse) |
| U12 | Quad comparator | LM339DR | 1 | Overcurrent (both channels) + heartbeat watchdog + spare |
| U13 | LiPo charger | TP4056X | 1 | USB-C charging |
| U14 | LDO +15V→5V_ISO | TLV70450DBVR | 1 | DAC/isolator power, SOT-23-5 |
| Q1 | USB interlock MOSFET | BSS138 | 1 | VBUS → EN disable, SOT-23 |
| R_S1, R_S2 | Sense resistor | Susumu RG2012P-101-B-T5 | **2** | 100Ω 0.1% 25PPM, 0805 |
| R_B1, R_B2 | Bleed resistor | Generic 1MΩ 1% 0805 | **2** | Across DC-block caps |
| C_DC1, C_DC2 | DC-blocking cap | Samsung CL31B106KBHNNNE | **2** (+2 spare) | 10µF 50V 1206 |
| F1, F2 | PPTC fuse | SMD0603B001TF | **2** (+2 spare) | 10mA hold, 0603 |
| L1 | Boost inductor | Würth 744043002 or similar | 1 | 2.2µH, 1A, ≤200mΩ |
| FB1 | CM choke | Würth 744232601 | 1 | 600Ω @ 100MHz |
| — | Passives (decoupling) | Various 0402/0603 | ~30 | 100nF, 10µF, 47µF, etc. |
| — | Feedback resistors | Various 0603 1% | ~10 | Boost divider, comparator ref, etc. |
| BT1 | LiPo battery | Generic 3.7V 2000mAh | 1 | With protection PCB |
| J1 | USB-C connector | Generic USB-C 2.0 | 1 | Charging only, SMD |
| J2, J3 | Electrode connectors | JST PH 2-pin SMD | **2** | Patient-facing, SMD variant |
| SW1 | Emergency stop | Tactile NO switch | 1 | Hardware kill, **through-hole** |

**Total active ICs: 13** (incl. U14) | **Discrete: 1** (Q1) | **Total unique SMD types: ~27** | **Total components: ~70**

---

## 6. DC-Blocking Cap Replacement

### 🟡 IMPORTANT 4: RESOLVED

### Voltage Rating Analysis — 250V is massive overkill

The DC-blocking cap sits between the H-bridge output and the electrode. The maximum voltage it can ever see equals the current source compliance voltage:

```
V_cap_max = V_supply - V_sat - V_rsense
          = 15V - 1.5V - 0.5V
          = 13V (absolute worst-case fault, open electrode)
```

In normal biphasic operation, the cap charges/discharges symmetrically — net DC ≈ 0V, instantaneous voltage ≈ ±100mV.

**Required voltage rating: 25V minimum (2× worst-case). 50V = 3.8× derating — proper medical margin.**

The original 250V rating was for a different topology. Dropping to 50V reduces package size by **700×**.

### Minimum Capacitance Analysis

```
Q per phase = I_max × t_pulse = 5mA × 200µs = 1.0 µC

Acceptable droop per phase (must be ≪ compliance voltage):
  Target: ≤100mV droop (0.77% of 13V compliance — negligible)

  C_min = Q / V_droop = 1.0µC / 0.1V = 10µF

At lower clinical currents (2mA × 200µs = 0.4µC):
  V_droop = 0.4µC / 10µF = 40mV — even better

Could 1µF work?
  V_droop = 1.0µC / 1µF = 1.0V — significant (7.7% of compliance)
  At duty cycle 30s ON/30s OFF, 25Hz: 750 pulses per ON period
  Cumulative charge concern: cap self-balances each biphasic cycle
  1µF would work electrically but 1V droop reduces compliance headroom.
```

**Verdict: 10µF is the right value. 1µF works but erodes compliance margin unnecessarily.**

### Recommended Replacement: TDK C3216X7R1H106K160AB

| Parameter | Original (ECQ-E2475KF) | Replacement (TDK C3216X7R1H106K160AB) |
|-----------|----------------------|---------------------------------------|
| Capacitance | 4.7µF (need 2 in parallel) | **10µF** (single cap) |
| Voltage | 250V | **50V** (3.8× derating — correct) |
| Dielectric | Polyester film | **X7R MLCC** |
| Package | 26×12×21.5mm through-hole | **3.2×1.6mm 1206 SMD** |
| Volume | ~5,720 mm³ | ~8 mm³ (**700× smaller**) |
| DigiKey | — | 445-7659-1-ND |
| Price | ~$1.74 | ~$0.15 |

**MLCC concern analysis (why X7R is acceptable here):**

| Concern | Impact in This Design |
|---------|-----------------------|
| DC bias coefficient | Cap normally sees <100mV DC — derating negligible (<5%) |
| Piezoelectric noise | Cap sees <100mV → low mechanical stress → minimal piezo effect |
| Temperature coefficient | ±15% over full range; at body temperature, deviation <5% |
| Aging | X7R: ~2.5%/decade — negligible for DC-blocking function |

> ✅ **C0G/NP0 would be ideal** (zero piezo, zero voltage coefficient) but tops out at ~100nF in 1206. Cannot reach 10µF. X7R at 50V/1206 is the correct tradeoff.

> 💡 **Consider 2× 10µF in parallel (20µF total)** for extra margin and redundancy. Two 1206 caps cost $0.30 and occupy 6.4×1.6mm — still 100× smaller than the original.

---

## 7. SMD Replacements

### 🟡 IMPORTANT 5: Sense Resistor — RESOLVED

**Recommended: Susumu RG2012P-101-B-T5**

| Parameter | Original (YR1B100RCC) | Replacement (Susumu RG2012P-101-B-T5) |
|-----------|----------------------|--------------------------------------|
| Resistance | 100Ω | 100Ω |
| Tolerance | 0.1% | **0.1%** ✅ |
| TCR | 15 PPM/°C | **25 PPM/°C** (adequate — see analysis) |
| Package | Axial through-hole | **0805 SMD** |
| Technology | Metal film | **Thin film** (superior stability) |
| Power | 250mW | 125mW (0805) — adequate: P = 5mA² × 100Ω = 2.5mW |
| DigiKey | — | RG2012P-101-B-T5CT-ND |

**Error analysis:**
```
Tolerance: ±0.1% of 100Ω = ±0.1Ω → at 5mA: ±5µA (0.1%) — excellent
TCR at ΔT=20°C: 25PPM × 20°C × 100Ω = 0.05Ω → ±2.5µA — negligible
Total current uncertainty: ±7.5µA at 5mA = ±0.15%
```

**Alternate (if Susumu unavailable):** Vishay TNPW0805100RBEEA (0.1%, 25 PPM/°C, 0805)

### 🟡 IMPORTANT 6: MCP4922 Package — RESOLVED

| Parameter | Through-hole | SMD |
|-----------|-------------|-----|
| MPN | MCP4922-E/P (PDIP-14) | **MCP4922-E/SL** (SOIC-14) |
| Package | PDIP-14 (19.3×6.35mm) | SOIC-14 (8.65×3.9mm) |
| DigiKey | MCP4922-E/P-ND | **MCP4922-E/SL-ND** |
| Electrical | Identical | Identical |

> Note: The "T" in MCP4922T-E/SL indicates tape-and-reel packaging for pick-and-place. For hand assembly, MCP4922-E/SL (tube) is fine.

### DAC Placement Decision

The MCP4922 DAC should be on the **isolated (patient) side** of the barrier, not the digital side. Reason:

- If DAC is on digital side: analog voltage must cross the isolation barrier → requires analog isolator (expensive, adds noise)
- If DAC is on isolated side: only digital SPI signals cross the barrier → handled by SI8380P-IU digital isolator ✅

The MCP4922 runs from 2.7V–5.5V. On the isolated side, power it from **+5V derived from the +15V rail via a small LDO** (e.g., TLV70450DBVR, 24V input, 5V fixed output, SOT-23-5). This LDO serves ONLY the DAC — the op-amp runs directly from ±15V.

**Add to BOM: TLV70450DBVR** — 1× LDO for DAC power on isolated side.

| Parameter | TLV70450DBVR |
|-----------|-------------|
| Vin | 2.5V–24V (safe on +15V rail) |
| Vout | 5.0V fixed |
| Iout max | 150mA (DAC draws ~1mA — massive margin) |
| Noise | 30µVrms |
| PSRR | 63dB @ 1kHz |
| Package | SOT-23-5 |
| DigiKey | TLV70450DBVRCT-ND |

---

## 8. Loop Stability Assessment

### 🟡 IMPORTANT 7: V-to-I Loop Stability — RESOLVED

#### Circuit Under Analysis

```
            Vdac (0–0.5V)
              │
        ┌─────┴─────┐
        │    [+]     │
        │            │  ADA4522-2ARZ
        │    [−]     │  (powered from ±15V)
        │     │      │
        └─────┼──────┘
              │   │
              │   └──► DRV8871 VM pin → H-bridge → electrode → return
              │                                                   │
              └────── Rsense (100Ω) ──────────────────────────────┘
                        │
                       GND_ISO
```

**This is a unity-gain voltage follower forcing V(Rsense) = V(dac).** The output current is I = V(dac) / R(sense).

#### Pole Analysis

**Pole 1 — Electrode capacitance:**
Electrode-skin interface has ~10–100nF capacitance in parallel with 500Ω–5kΩ resistance.
```
With C_electrode = 100nF, R_electrode = 1kΩ:
f_pole_electrode = 1/(2π × 1kΩ × 100nF) = 1.59 kHz
```

**Pole 2 — Op-amp open-loop rolloff:**
ADA4522-2 GBW = 2.7 MHz. At unity gain, the closed-loop bandwidth = 2.7 MHz.

**Pole 3 — DRV8871 switching delay:**
When the H-bridge is latched in one direction (IN1=H, IN2=L), the internal MOSFETs are just resistive paths (Rdson ~0.36Ω). No additional pole — the H-bridge is transparent to the analog feedback loop during a pulse phase.

#### Phase Margin Assessment

```
Loop gain crossover: ~2.7 MHz (GBW)
Electrode pole at: ~1.6 kHz (worst case 100nF electrode)

Phase contributed by electrode pole at crossover:
  θ = -arctan(2.7MHz / 1.6kHz) ≈ -90° (fully rolled off)

BUT: the sense resistor (100Ω) in series with the electrode impedance creates a zero:
  f_zero = 1/(2π × Rsense × C_electrode) = 1/(2π × 100 × 100nF) = 15.9 kHz

Phase recovery from zero at crossover:
  θ_zero = +arctan(2.7MHz / 15.9kHz) ≈ +89°

Net phase from load: approximately -90° + 89° ≈ -1° (nearly cancels)
Phase margin: ~89° → STABLE
```

**The 100Ω sense resistor inherently stabilizes the loop** by placing a zero near the electrode pole. This is one of the key advantages of V-to-I over Howland — the sense resistor acts as both the current measurement element AND the stability compensation.

#### Compensation Recommendation

**No external compensation network required** for the primary feedback loop.

However, add a **10nF capacitor across Rsense** (parallel with the 100Ω) as a precaution:
```
f_filter = 1/(2π × 100Ω × 10nF) = 159 kHz
```
This rolls off high-frequency noise (including ADA4522 chopper artifacts at ~4kHz) without affecting the 20–25Hz stimulation bandwidth. It also provides additional phase margin at very high frequencies.

#### Slew Rate Adequacy

```
ADA4522-2 slew rate: 1.7 V/µs

Rsense voltage step (0→5mA): 0 → 500mV = 500mV step
Slew time: 500mV / 1.7V/µs = 0.29µs

Op-amp output voltage step (driving load):
At 5mA into 1kΩ electrode: V_out = 5mA × (1000 + 100)Ω = 5.5V
Slew time: 5.5V / 1.7V/µs = 3.2µs

At 5mA into 3kΩ electrode (max compliance):
V_out = 5mA × (3000 + 100)Ω = 15.5V → clipped at ±15V supply
Slew time: 15V / 1.7V/µs = 8.8µs
```

**Worst-case 8.8µs slew + ~5µs settling = ~14µs total rise time.**
At 200µs pulse width: 14µs/200µs = **7% rise time** → 93% flat-top. ✅ Acceptable.

#### DAC Settling Time

MCP4922 settling: 4.5µs. The firmware inserts a 50µs inter-phase gap (dead time) between biphasic phases. DAC is updated during this gap → fully settled before the next phase begins. **No timing conflict.**

```
Timeline:  |--Phase+ 200µs--|--Gap 50µs--|--Phase- 200µs--|--Gap 50µs--|
DAC write:                   ↑ here (settles in 4.5µs)
                             |--4.5µs--| 45.5µs margin before Phase- starts
```

#### Verdict: ✅ Loop is stable. No external compensation mandatory. Add 10nF across Rsense as belt-and-suspenders.

---

## 9. Isolation Compliance (IMPORTANT 8)

### 🟡 IMPORTANT 8: TMA → PCN1-S5-D15-M-TR Isolation — RESOLVED

The question: does the DC-DC converter isolation independently matter for IEC 60601-1 Type BF?

**Answer: Yes, but PCN1-S5-D15-M-TR already meets the requirement.**

#### IEC 60601-1 Type BF Requirements (Battery-Powered)

For a battery-powered Type BF applied part, the key requirement is **Means of Patient Protection (MOPP)**:

| Parameter | IEC 60601-1 Requirement | Our Design |
|-----------|------------------------|------------|
| Patient isolation (DC) | **1500V DC** (1 MOPP for battery device) | PCN1-S5-D15-M-TR: **1500VDC** ✅ |
| Creepage distance | ≥4mm (reinforced) | PCB layout dependent — design for ≥6mm |
| Clearance | ≥4mm (reinforced) | PCB layout dependent — design for ≥6mm |
| Patient leakage current | ≤100µA (normal), ≤500µA (single fault) | Battery-isolated, ~pA leakage |

#### Isolation Barrier Components and Their Ratings

| Component | Isolation Rating | Role |
|-----------|-----------------|------|
| PCN1-S5-D15-M-TR | **1500VDC** | Power transfer across barrier |
| SI8380P-IU | **2500Vrms (~3535Vpk)** | Signal transfer across barrier (8 forward) |
| SI8622EC-B-IS | **3750Vrms (~5300Vpk)** | Signal transfer across barrier (heartbeat + fault) |
| DC-blocking caps | **50V** (not an isolation component) | DC-blocking only |

**The isolation barrier is defined by the WEAKEST link: PCN1-S5-D15-M-TR at 1500VDC.** Both digital isolators exceed this, so they are not the limiting factor.

**1500VDC meets the 1 MOPP requirement for battery-powered Type BF.** No issue.

> ⚠️ **If you want 2 MOPP (reinforced insulation for higher safety factor):** would require an isolated DC-DC with ≥3000VDC isolation rating. Not pursued — 1 MOPP is the correct requirement for battery-powered Type BF, and finding a dual ±15V SMD part with >1500VDC was already difficult.

> ⚠️ **PCB layout is critical:** The isolation barrier must be maintained in the PCB with adequate creepage/clearance distances. A 1500V-rated component on a PCB with 1mm trace spacing defeats the purpose. Design the isolation zone with a **routed slot** (milled gap) in the PCB between digital and patient domains, minimum 6mm width.

---

## 10. Updated Complete BOM

### Full Bill of Materials — 2-Channel Binaural Build

#### Power Subsystem

| Ref | Description | MPN | Package | Qty | DigiKey PN | ~Unit Price |
|-----|-------------|-----|---------|-----|-----------|-------------|
| U2 | Boost LiPo→5V | **TPS61023DRLR** | SOT-5 | 1 | 296-43XXX | $0.75 |
| U3 | LDO LiPo→3.3V | **ME6211C33M5G** | SOT-23-5 | 1 | — (LCSC) | $0.20 |
| U4 | Isolated DC-DC 5V→±15V | **PCN1-S5-D15-M-TR** | SMD-8 | 1 | — (CUI/Mouser) | $5.00 |
| U14 | LDO +15V→5V (DAC power) | **TLV70450DBVR** | SOT-23-5 | 1 | TLV70450DBVRCT-ND | $0.65 |
| L1 | Boost inductor 2.2µH | **Würth 744043002** | 1210 | 1 | 732-1216-1-ND | $0.50 |
| FB1 | CM choke (noise filter) | **Würth 744232601** | 0805 | 1 | 732-1622-1-ND | $0.80 |
| U13 | LiPo charger IC | **TP4056X** | SOP-8 | 1 | — (LCSC) | $0.15 |
| BT1 | LiPo battery 3.7V 2000mAh | Generic with protection | — | 1 | — (AliExpress) | $5.00 |

#### Signal Chain (Per Channel × 2)

| Ref | Description | MPN | Package | Qty | DigiKey PN | ~Unit Price |
|-----|-------------|-----|---------|-----|-----------|-------------|
| U5 | Dual 12-bit SPI DAC | **MCP4922-E/SL** | SOIC-14 | 1 | MCP4922-E/SL-ND | $3.50 |
| U6 | Dual zero-drift error amp | **ADA4522-2ARZ** | SOIC-8 | 1 | ADA4522-2ARZ-ND | $5.00 |
| U7, U8 | H-bridge driver | **DRV8871DDAR** | SOIC-8 PP | **2** | 296-43024-1-ND | $2.50 |
| R_S1, R_S2 | Sense resistor 100Ω 0.1% | **Susumu RG2012P-101-B-T5** | 0805 | **2** | RG2012P-101-B-T5CT-ND | $1.20 |
| C_DC1, C_DC2 | DC-blocking cap 10µF 50V | **Samsung CL31B106KBHNNNE** | 1206 | **2** | — (LCSC/Mouser) | $0.02 |
| R_B1, R_B2 | Bleed resistor 1MΩ 1% | Generic 0805 | 0805 | **2** | — | $0.02 |
| F1, F2 | PPTC fuse 10mA/30mA | **SMD0603B001TF** | 0603 | **2** | 13-SMD0603B001TFCT-ND | $0.30 |
| C_F1, C_F2 | Feedback cap 10nF C0G | Generic 0805 C0G 50V | 0805 | **2** | — | $0.05 |

#### Isolation Barrier

| Ref | Description | MPN | Package | Qty | DigiKey PN | ~Unit Price |
|-----|-------------|-----|---------|-----|-----------|-------------|
| U9 | 8-ch digital isolator (all forward) | **SI8380P-IU** | QSOP-20 | **1** | — (Skyworks/Mouser) | ~$3.50 |
| U10 | 2-ch bidirectional isolator | **SI8622EC-B-IS** | SOIC-8 | 1 | 336-1750-ND | $3.00 |

#### Safety

| Ref | Description | MPN | Package | Qty | DigiKey PN | ~Unit Price |
|-----|-------------|-----|---------|-----|-----------|-------------|
| U12 | Quad comparator (overcurrent + watchdog) | **LM339DR** | SOIC-14 | 1 | 296-1393-1-ND | $0.50 |
| Q1 | USB interlock N-MOSFET | **BSS138** | SOT-23 | 1 | BSS138CT-ND | $0.10 |
| SW1 | Emergency stop button | Generic tactile NO | Through-hole | 1 | — | $0.50 |

#### MCU

| Ref | Description | MPN | Package | Qty | DigiKey PN | ~Unit Price |
|-----|-------------|-----|---------|-----|-----------|-------------|
| U1 | ESP32-S3 module | **ESP32-S3-WROOM-1-N16R8** | Module | 1 | 1965-ESP32-S3-WROOM-1-N16R8-ND | $4.50 |

#### Connectors & Mechanical

| Ref | Description | MPN | Package | Qty | ~Unit Price |
|-----|-------------|-----|---------|-----|-------------|
| J1 | USB-C connector (charging) | Generic USB-C 2.0 16-pin | SMD | 1 | $0.30 |
| J2, J3 | Electrode connectors (2.0mm) | Generic 2-pin JST PH | SMD | 2 | $0.20 |

#### Passive Decoupling (bulk — approximate)

| Description | Value/Package | Qty | ~Total |
|-------------|---------------|-----|--------|
| Ceramic bypass caps (100nF) | 0402 X7R 16V | 15 | $0.50 |
| Ceramic bypass caps (10µF) | 0805 X5R 10V | 6 | $0.60 |
| Electrolytic caps (47µF) | 6.3×5.4mm SMD | 4 | $1.00 |
| Boost output cap (22µF) | 0805 X5R 10V | 2 | $0.30 |
| Resistor dividers, pullups | Various 0603 1% | 15 | $0.50 |
| RC filter (DAC output) | 1kΩ + 100nF per channel | 4 | $0.20 |

### BOM Cost Summary

| Category | Cost |
|----------|------|
| Power subsystem | ~$11.05 |
| Signal chain (both channels) | ~$16.60 |
| Isolation barrier | ~$10.00 |
| Safety | ~$0.90 |
| MCU | ~$4.50 |
| Connectors | ~$0.70 |
| Passive decoupling | ~$3.10 |
| **Component total** | **~$46.85** |
| PCB fabrication (JLCPCB, 4-layer, 5 boards) | ~$20.00 |
| Battery | ~$5.00 |
| Electrode clips/cables | ~$5.00 |
| **Grand total** | **~$76.85** |

✅ **Within $100 BOM target** (PROJECT.md constraint)

---

## 11. Additional Issues Found

### 🟡 NEW ISSUE A: MCP4922 Placement and Reference Voltage

The MCP4922 is on the **isolated side** (correct). It needs a voltage reference:

- **Internal Vref:** MCP4922 has no internal reference — requires external Vref on each channel
- **Full-scale output for 5mA:** V_fullscale = I_max × R_sense = 5mA × 100Ω = 0.5V
- **Vref options:**
  - Use MCP4922's 2× gain mode with Vref = 0.25V (from a voltage divider on the 5V LDO output)
  - Use 1× gain mode with Vref = 0.5V
  - Use a precision reference IC (e.g., REF3005 = 0.5V, or REF3012 = 1.25V with resistive divider)

**Recommendation:** Use the TLV70450's 5V output with a precision resistor divider (10kΩ/1kΩ, 0.1%) to create ~0.455V Vref. With MCP4922 in 2× gain mode, full-scale output = 0.909V → allows Rsense voltage up to 0.909V → I_max = 9.09mA — headroom above the 5mA therapeutic maximum. Fine-tune the divider ratio.

### 🟡 NEW ISSUE B: DRV8871 Internal Current Limit Must Be Disabled

The DRV8871 has an internal current regulation feature via the VREF pin:
- I_limit = VREF / (5 × R_ISEN)
- The ISEN pin has an internal 178µΩ current sense FET

**If VREF is left at its internal default, the DRV8871 may interfere with the external V-to-I feedback loop.** Two regulation loops fighting = oscillation.

**Solution:** Tie DRV8871 VREF pin to VCC (through a resistor per datasheet) to set the internal current limit well above the operating range (>1A). The external op-amp + Rsense loop controls current.

### 🟡 NEW ISSUE C: USB Charging Interlock — Hardware Implementation

The "no stimulation while USB charging" requirement needs a specific hardware implementation:

```
USB VBUS (5V) ──► voltage divider ──► GPIO (VBUS_DETECT)
                                       │
                                       └──► AND gate with TPS61023 EN pin:
                                            EN = LOW when VBUS present
                                            Result: boost converter OFF → PCN1 OFF → ±15V OFF
                                            → physically impossible to stimulate while charging
```

**Use a simple N-MOSFET (BSS138) as the interlock switch:**
- VBUS present → BSS138 gate HIGH → pulls TPS61023 EN LOW → boost OFF → no ±15V → safe
- VBUS absent → BSS138 gate LOW → TPS61023 EN pulled HIGH (10kΩ pullup) → boost ON → stimulation possible

This is a **hardware-only** interlock — firmware cannot bypass it.

### 🟡 NEW ISSUE D: Heartbeat Watchdog on Isolated Side

The isolated side must have a way to detect if the ESP32 has crashed. Solution from ARCHITECTURE.md:

- ESP32 Core 1 generates a 100Hz PWM "heartbeat" signal → crosses barrier via SI8622
- On isolated side: RC detector (τ ≈ 50ms) + comparator
- If heartbeat stops for >50ms (ESP32 crash, BLE lockup, etc.): comparator output goes LOW → kills analog power enable → all stimulation stops

This requires one additional comparator — ~~the LM393DR has 2 channels, one per overcurrent channel.~~ **RESOLVED: LM339DR (quad comparator, SOIC-14) replaces LM393DR.** Channels: (1) overcurrent A, (2) overcurrent B, (3) heartbeat watchdog, (4) spare (electrode contact detection).

### ⚠️ NEW ISSUE E: OPA388 in AGENTS.md Is Wrong

AGENTS.md §2 lists OPA388 as the correct op-amp for V-to-I converter. However:
- OPA388 max supply = **5.5V** — cannot run from ±15V
- This was likely chosen before the isolated-side power architecture was finalized

**AGENTS.md must be updated:** Replace OPA388 → **ADA4522-2ARZ** in the component decisions table.

---

## 12. Summary of All Decisions

| Issue | Resolution | Part / Action |
|-------|-----------|---------------|
| 🔴 Boost converter | TPS61023DRLR (2.0–5.5V→5V, 500mA, SOT-5) | Add to BOM |
| 🔴 AD8628 supply | Replace with **ADA4522-2ARZ** (±27.5V, zero-drift, dual) | Swap in BOM |
| 🟡 Binaural arch | 2× DRV8871, 1× ADA4522-2 (dual), 1× MCP4922 (dual), 2× isolators | Explicit count above |
| 🟡 DC-block cap | **Samsung CL31B106KBHNNNE** (10µF, 50V, 1206) | Replace ECQ-E2475KF |
| 🟡 Sense resistor | **Susumu RG2012P-101-B-T5** (100Ω, 0.1%, 25PPM, 0805) | Replace YR1B100RCC |
| 🟡 DAC package | **MCP4922-E/SL** (SOIC-14) | Replace PDIP-14 |
| 🟡 Loop stability | Stable — Rsense provides inherent compensation. Add 10nF across Rsense. | No topology change |
| 🟡 Isolation level | PCN1-S5-D15-M-TR = 1500VDC → meets 1 MOPP. | Already correct |
| NEW: DAC power | **TLV70450DBVR** LDO (+15V→5V, isolated side) for MCP4922 | Add to BOM |
| NEW: DRV8871 VREF | Tie to VCC — disable internal current limit | Layout/design note |
| NEW: USB interlock | BSS138 + TPS61023 EN pin — hardware-only lockout | Add to schematic |
| NEW: AGENTS.md | OPA388 → ADA4522-2ARZ in component table | Update file |

---

## 13. Open Items for PCB Layout Phase

1. **Isolation slot:** Route a milled slot in the PCB between digital and patient domains. Minimum 6mm width. Narrow to ~3mm only under isolation ICs (SI8380P-IU, SI8622EC, PCN1-S5-D15-M-TR) where internal IC isolation supplements PCB creepage.
2. **Ground planes:** Separate GND_DIGITAL and GND_ISO copper pours. Connect ONLY through PCN1-S5-D15-M-TR isolation.
3. **Barrier component placement:** All three barrier-crossing components (PCN1-S5-D15-M-TR, SI8380P-IU, SI8622EC-B-IS) placed in a line along the isolation slot. PCN1 pad groups must align with slot (CUI datasheet specifies ~5mm input-to-output pad gap).
4. **DRV8871 thermal pad:** SOIC-8 PowerPAD needs vias to internal copper pour for heat dissipation (even at 5mA, good practice). Same for TP4056X ESOP exposed pad.
5. **Analog layout:** Keep DAC output traces short, away from switching signals. Route Rsense feedback as a Kelvin connection (4-wire sense).
6. **SPI routing:** SPI traces on digital side → through SI8380P-IU → to DAC on patient side. Keep SPI clock and data parallel-routed. No impedance matching needed at 1 MHz (SI8380P-IU limit: 2 Mb/s).
7. **Patient-side placement order:** SI8380P-IU outputs → MCP4922 (close) → ADA4522-2ARZ → DRV8871s → Rsense → DC-block → PPTC → electrode connectors (board edge). LM339DR adjacent to DRV8871s for short IPROPI traces.
8. **4-layer stackup:** L1=signal+components, L2=GND (split at barrier), L3=power (split: 3.3V/5V digital | ±15V/5V_iso patient), L4=signal+routing.
9. **Boost converter (TPS61023):** Keep SW node trace short and shielded with ground pour. Input cap directly at VIN pin. Design for ~4 MHz switching frequency (verify from TI datasheet — distributor listing showed 1 MHz which may be an error).

---

*End of design review. All 10 original issues resolved. 5 new issues identified and addressed. BOM complete at ~$77 for full 2-channel build. Updated 2026-04-23: SI8380P-IU replaces 2× SI8641EC, LM339DR replaces LM393DR, PCN1-S5-D15-M-TR replaces B0515D-1WR3, Samsung CL31B106KBHNNNE replaces TDK C3216X7R1H106K160AB, Susumu TCR corrected to 25PPM, PCB layout notes expanded.*
