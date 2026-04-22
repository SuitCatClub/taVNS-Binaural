# Circuit Design Review — OpenBinaural-taVNS Custom PCB

**Date:** 2025-07-15
**Scope:** Full analog signal chain + power architecture for binaural taVNS stimulator
**Reviewer:** AI (cross-referenced against ARCHITECTURE.md, PITFALLS.md, STACK.md, AGENTS.md)
**Verdict:** Topology is sound. 5 critical component issues found, all solvable. Corrected BOM below.

---

## 1. Topology Validation — V-to-I Converter + H-Bridge

### Verdict: ✅ CONFIRMED — Correct topology for this application

The V-to-I converter with op-amp error amplifier + sense resistor feedback + H-bridge polarity switching is the right choice. The existing ARCHITECTURE.md research already validated this over the Howland pump, and the reasoning is solid:

| Criterion | Howland Pump | V-to-I + H-Bridge | Winner |
|-----------|-------------|-------------------|--------|
| Stability into capacitive loads (skin) | Poor — oscillation risk at >2kΩ | Unconditionally stable at unity gain | V-to-I |
| Resistor matching sensitivity | 0.01% match needed for output Z | Not matching-sensitive | V-to-I |
| Component count | 5R + 1 op-amp, no H-bridge | 1R + 1 op-amp + H-bridge | Comparable |
| Biphasic generation | Needs bipolar DAC drive | Unipolar DAC + H-bridge switches polarity | V-to-I simpler |
| Charge balance confidence | Depends on perfect R-match | DAC code identical both phases, H-bridge swaps direction | V-to-I better |

**How the feedback loop works (confirmed):**

```
Current path:
  Op-amp output → H-bridge → electrode(+) → tissue → electrode(−) → Rsense → GND_ISO
                                                                        ↑
Feedback:                                                          V(−) input of op-amp

Servo: Op-amp adjusts output voltage until V(−) = V(+) = Vdac
       Therefore: I_load × Rsense = Vdac
       Therefore: I_load = Vdac / Rsense = Vdac / 100Ω
```

The load (electrodes + tissue) is inside the feedback loop but only in the series path — it doesn't affect loop stability because the op-amp just adjusts its output voltage to compensate for any load impedance changes. The sense resistor provides the feedback, not the load.

**Compliance headroom (with ±15V rails):**

```
V_compliance = V_opamp_swing − V_Rsense
             = 13V (OPA-class output swing) − 0.5V (5mA × 100Ω)
             = 12.5V

Z_max at 5mA  = 12.5V / 5mA  = 2.5kΩ
Z_max at 3mA  = 12.5V / 3mA  = 4.17kΩ
Z_max at 2mA  = 12.5V / 2mA  = 6.25kΩ  ← clinical sweet spot
Z_max at 1mA  = 12.5V / 1mA  = 12.5kΩ
```

This covers >95% of real clinical scenarios with well-prepared cymba conchae electrodes (500Ω–3kΩ). Out-of-compliance conditions (Z > 5kΩ) indicate bad electrode contact and should trigger firmware pause — this is correct behavior, not a limitation.

---

## 2. Critical Issues Found

### 🔴 CRITICAL #1: TMA 0515D Is the Wrong Isolated DC-DC

**Problem:** The TMA 0515D provides only **1kV isolation** — below the project's 1500VDC reinforced isolation target. Additionally, AGENTS.md explicitly specifies: *"Never use B0515S (single rail) — use B0515D-1WR3 (dual ±15V)."*

**Why 1kV is insufficient:** The isolation barrier is the last line of defense between mains-coupled ground faults (through USB charger → laptop → USB cable) and the patient. IEC 60601-1 Type BF requires reinforced isolation. The SI8622EC provides 3750Vrms on the digital signals, but the DC-DC converter carries POWER across the barrier — if it breaks down, milliamps flow into the patient domain. The weakest component in the isolation chain sets the safety rating. At 1kV, the TMA 0515D is the weak link.

**Fix:** Replace with **B0515D-1WR3** (MORNSUN):
- Input: 4.5–5.5V
- Output: ±15V @ 34mA each rail
- Power: 1W
- **Isolation: 1500VDC** ✅
- SIP-6 through-hole (same footprint family)
- Widely available on AliExpress, DigiKey, Farnell (~$5–8)

**Tradeoff:** None. Same input/output specs, same form factor, better isolation. Direct replacement.

---

### 🔴 CRITICAL #2: AD8628ARZ Cannot Run on ±15V Rails

**Problem:** The AD8628ARZ has a **maximum total supply voltage of 5V** (±2.5V). On the isolated side where ±15V is available, the AD8628 would be instantly destroyed if powered from the ±15V rails. Even if you derive a separate ±2.5V rail, the AD8628's output can only swing ≈0–5V, which is insufficient to drive the compliance voltage needed (up to 12.5V) through the H-bridge and load.

**The op-amp in this topology must:**
1. Be powered from the ±15V rails (or close to them)
2. Swing its output from ~0V to ~13V to drive current through varying electrode impedances
3. Have very low input offset voltage for charge balance (DC-blocking cap is primary defense, but lower offset = less stress on the cap)

**Fix — Option A (RECOMMENDED): ADA4522-2ARZ** (Analog Devices)
- **Zero-drift auto-zero** architecture (like AD8628, but high-voltage)
- Max supply: **±18V (36V total)** — runs directly on ±15V ✅
- Vos: **0.3µV typical** (better than AD8628's 1µV)
- GBW: 2.7MHz (better than AD8628's 2.5MHz)
- Slew rate: 1.4V/µs (better than AD8628's 1V/µs)
- **Dual channel** — one IC covers both binaural channels ✅
- CMRR: 150dB, PSRR: 140dB
- Rail-to-rail input; output swings to within 150mV of rails
- SOIC-8 or MSOP-8
- DigiKey: ~$5–7

**Fix — Option B (BUDGET): OPA2277PA** (Texas Instruments)
- Precision but NOT zero-drift
- Max supply: ±18V ✅
- Vos: ±20µV max (acceptable — DC-blocking cap catches the 0.2µA residual)
- GBW: 1MHz
- Slew rate: 0.8V/µs
- Dual channel ✅
- DIP-8 or SOIC-8
- DigiKey: ~$3–4
- This is the op-amp already chosen in ARCHITECTURE.md

**Recommendation:** Use **ADA4522-2ARZ** for zero-drift (matches the AD8628's original intent) with the voltage headroom to actually work in this circuit. The 0.3µV offset means the DC-blocking cap barely works — the net DC offset current through 100Ω sense = 0.003µA, which is negligible.

**Why not derive ±2.5V for the AD8628?** Because even with a low-voltage supply, the AD8628's output can only swing 0–5V. The V-to-I converter needs to output up to 12.5V (at 5mA into 2.5kΩ) to maintain current regulation. A 5V op-amp physically cannot drive this topology.

---

### 🔴 CRITICAL #3: Boost Converter Missing — LiPo Cannot Power B0515D-1WR3 Directly

**Problem:** The B0515D-1WR3 (and the TMA 0515D) requires **4.5V minimum input**. A LiPo cell ranges from 3.0V (empty) to 4.2V (full). Even at full charge, the battery is 300mV below the DC-DC's minimum input. The device literally cannot produce ±15V from the battery without a boost stage.

**Fix: Add a boost converter — LiPo (3.0–4.2V) → 5.0V**

**RECOMMENDED: TPS61023DRLR** (Texas Instruments)
- Input: 0.5V–5.5V (works across full LiPo range including deep discharge)
- Output: 5.0V fixed
- Output current: up to 500mA (system draws ~200mA peak)
- Efficiency: 93–96% at typical load
- Switching frequency: 1.8MHz (above audio band, small inductor)
- Quiescent: 5µA in PFM mode (excellent for sleep device)
- Package: SOT-23-5
- External components: 1µH inductor + 10µF ceramic × 2
- DigiKey: ~$1.50

**BUDGET ALTERNATIVE: MT3608** (common AliExpress module)
- Input: 2V–24V
- Output: up to 28V (set by feedback divider — configure for 5V)
- Output current: up to 2A
- Efficiency: 85–93% (slightly worse than TPS61023)
- Switching frequency: 1.2MHz
- Comes as a complete breakout board on AliExpress for ~$0.50
- **Caveat:** Higher ripple noise than TPS61023. For prototype/breadboard phase, acceptable. For final PCB, use TPS61023.

**Noise concern:** The boost converter's switching noise can couple into the analog domain. BUT — the B0515D-1WR3 isolated DC-DC provides galvanic isolation between the 5V digital domain and the ±15V patient domain. The boost converter noise stays on the digital side. The isolated DC-DC's own switching noise (50–100mV ripple at ~100kHz) needs LC filtering on the isolated output — this is already addressed in the architecture (100µF + 100nF + 1nF decoupling per rail).

**Power budget verification:**

| Consumer | Current from 5V | Notes |
|----------|----------------|-------|
| ESP32-S3 (via on-board 3.3V LDO) | ~80mA avg, 350mA peak | BLE TX bursts |
| B0515D-1WR3 (isolated DC-DC) | ~62mA | 250mW at 80% eff |
| Digital isolators (×3) | ~15mA | Si8621/Si8622 quiescent |
| Boost converter overhead | ~10mA | Quiescent + switching losses |
| LED + misc | ~5mA | |
| **Total** | **~172mA avg, ~440mA peak** | |

TPS61023 at 500mA output: ✅ adequate with margin.
LiPo 2000mAh at 3.7V / (172mA × 5V / 0.94 eff) ≈ **8+ hours** → ~16 sessions of 30min.

---

### 🟡 SIGNIFICANT #4: DRV8871DDAR — Minimum Voltage Incompatibility

**Problem:** The DRV8871 has a **minimum supply voltage (Vs) of 6.5V**. In the V-to-I topology, the H-bridge's supply voltage equals the op-amp's output, which must be able to go as low as:

```
V_opamp_min = I_min × (Z_load + Rsense)
            = 0.5mA × (500Ω + 100Ω) = 0.3V
```

At 0.3V on VM, the DRV8871 is far below its 6.5V minimum operating voltage and will not function. The internal logic, gate drivers, and charge pump require a minimum supply to operate correctly.

**This means the DRV8871 CANNOT be used in the V-to-I topology where VM = op-amp output.**

**Fix — Option A (RECOMMENDED for prototype): Discrete MOSFET H-Bridge**

Use 4 discrete MOSFETs configured as a full H-bridge:
- **High-side P-channel:** 2× BSS84 (50V, 130mA, SOT-23) — ~$0.10 each
- **Low-side N-channel:** 2× BSS138 (50V, 220mA, SOT-23) — ~$0.10 each
- **Gate resistors:** 4× 1kΩ (limit gate current, ~$0.01 each)

Dead-time is enforced by the **ESP32-S3 MCPWM peripheral**, which generates complementary outputs with hardware-guaranteed dead-time (configurable, set ≥500ns). The MCPWM signals cross the isolation barrier via the Si8622/Si8641 digital isolators (8ns propagation delay — negligible).

**Advantages:**
- No minimum voltage — works from 0V to 50V (BSS84/BSS138 ratings)
- Full compliance headroom (op-amp can swing 0–13V freely)
- Dead-time is MCPWM hardware-enforced — not software
- Component cost: <$1 total for all 4 channels (8 MOSFETs + 8 resistors)
- Shoot-through protection from MCPWM dead-time

**Disadvantages:**
- More components than a single IC (8 MOSFETs + 8 resistors per 2 channels)
- PCB layout matters — keep gate traces short to avoid ringing
- Must verify dead-time on oscilloscope during bringup

**Fix — Option B (SIMPLER but limited): DRV8837DSGR** (TI, the original architecture choice)
- VM range: **0V–11V** (works at low voltages ✅)
- Maximum VM: 11V absolute max
- Built-in shoot-through protection and dead-time (~300ns)
- SOT-23-6, ~$1.50
- **Limitation:** Compliance capped at 11V → Z_max at 5mA = 2.2kΩ (vs 2.5kΩ with full ±15V)
- At clinical sweet spot (2mA): Z_max = 5.5kΩ — more than adequate

**Tradeoff analysis:**

| Criterion | DRV8871 | DRV8837 | Discrete MOSFETs |
|-----------|---------|---------|-----------------|
| Min VM | ❌ 6.5V | ✅ 0V | ✅ 0V |
| Max VM | 45V | ⚠️ 11V | 50V (BSS84) |
| Z_max at 5mA | N/A (can't use) | 2.2kΩ | 2.5kΩ |
| Z_max at 2mA | N/A | 5.5kΩ | 6.25kΩ |
| Dead-time | Internal | Internal ~300ns | MCPWM hardware |
| Shoot-through | Internal | Internal | MCPWM dead-time |
| Component count | 1 IC (×2 for binaural) | 1 IC (×2) | 4 MOSFETs + 4R (×2) |
| Cost per channel | ~$2.50 | ~$1.50 | ~$0.50 |

**Recommendation:** Use **discrete MOSFET H-bridge** for maximum compliance headroom and zero voltage restrictions. For breadboard prototype, DRV8837 is acceptable if you document the 11V compliance limit and warn users about maximum impedance at high currents.

---

### 🟡 SIGNIFICANT #5: Isolated-Side 5V Rail Missing

**Problem:** The MCP4922 DAC is on the isolated (patient) side of the barrier. It requires 2.7V–5.5V supply. The isolated side only has ±15V. You need a clean 5V (or 3.3V) rail on the isolated side for:
1. **MCP4922 DAC** supply and reference
2. **Si8622/Si8641 isolator** Vdd2 (isolated-side supply)
3. **MCP3201 ADC** (for impedance sensing, if used)
4. Any other digital ICs on the isolated side

**Fix: Add an LDO on the isolated side — +15V → +5V**

**RECOMMENDED: MC78L05ACPG** (ON Semiconductor) or equivalent 78L05
- Fixed 5V output, 100mA max
- Input: 7V–30V → +15V input is perfect ✅
- Dropout: ~2V (no problem with 15V→5V, 10V headroom)
- Package: TO-92 (through-hole, easy to prototype)
- Cost: ~$0.30
- External components: 330nF input ceramic + 100nF output ceramic (minimum)

**ALTERNATIVE: MCP1700-5002E/TO** (Microchip)
- LDO, 5V, 250mA
- Lower dropout (178mV typical) — overkill here but runs cooler
- SOT-23-3 or TO-92
- Better PSRR (55dB vs 78L05's 36dB)
- Cost: ~$0.50

**Power dissipation check:**

```
P = (Vin - Vout) × Iload = (15V - 5V) × 50mA = 500mW
```

Where 50mA estimate = MCP4922 (~2mA) + isolator Vdd2 (~5mA) + MCP3201 (~1mA) + decoupling.

500mW in TO-92 is within the 78L05's thermal limit (625mW with no heatsink). Adequate for a prototype. For PCB, consider a SOT-223 variant for better thermal performance.

**Note: You do NOT need a −5V rail.** The op-amp (ADA4522-2) runs directly on ±15V. The DAC and digital ICs only need +5V. The negative rail (−15V) is used only by the op-amp's negative supply pin.

---

## 3. Additional Issues Flagged

### ℹ️ Issue #6: Digital Isolator Channel Count

**Problem:** The SI8622EC-B-IS provides only **2 channels** (1 forward + 1 reverse). The circuit needs ~10 isolated channels:

| Signal | Direction | Count |
|--------|-----------|-------|
| SPI SCK | Digital → Patient | 1 |
| SPI MOSI | Digital → Patient | 1 |
| SPI CS (DAC) | Digital → Patient | 1 |
| SPI LDAC | Digital → Patient | 1 |
| H-bridge ctrl Ch A (2 signals) | Digital → Patient | 2 |
| H-bridge ctrl Ch B (2 signals) | Digital → Patient | 2 |
| Heartbeat PWM | Digital → Patient | 1 |
| Fault flag | Patient → Digital | 1 |
| SPI MISO (if MCP3201 ADC used) | Patient → Digital | 1 |
| **Total** | **9 forward + 2 reverse** | **11** |

**Fix: Use a combination of isolator ICs:**
- **2× Si8641BB-IS** (4-channel unidirectional, digital→patient) = 8 forward channels
- **1× SI8622EC-B-IS** (2-channel bidirectional) = 1 forward + 1 reverse (heartbeat + fault)
- **1× Si8621BB-IS** (2-channel, 1+1 bidirectional) = 1 forward (spare SPI) + 1 reverse (MISO)
- **Total: 4 ICs, 11 channels covered**

Or simpler for prototype: **6× SI8622EC-B-IS** (2-ch each) = 6 forward + 6 reverse (12 channels, 1 spare). Same IC, simpler BOM, wastes a few reverse channels. At ~$3 each = $18 total.

**Recommendation for prototype:** Use **3× Si8641BB-IS** (4-ch unidirectional, $3 each) = 12 forward channels, plus **1× SI8622EC-B-IS** (2-ch bidirectional) for the 2 reverse channels (fault + MISO). Total: 4 ICs, $12.

### ℹ️ Issue #7: Overcurrent Protection Architecture

**What you have:** SMD0603B001TF PPTC fuse (10mA hold, 30mA trip).

**What you also need:** The PPTC fuse is a **slow** protection mechanism (milliseconds to trip). For safety-critical overcurrent cutoff, ARCHITECTURE.md and REQUIREMENTS.md specify:

> "Hardware overcurrent comparator (LM393 + sense resistor) disconnects output within 10µs when current exceeds 6mA, completely independent of firmware"

**Complete overcurrent architecture (per channel):**

```
Layer 1 (FAST, <10µs): LM393 comparator
  - V+ = voltage across Rsense (100Ω × I_load)
  - V- = 0.6V reference (from voltage divider: 6mA × 100Ω = 0.6V trip)
  - Output: open-collector → drives SR latch SET
  - SR latch Q output → kills H-bridge enable (P-MOSFET gate in supply path)
  - Latch requires manual reset (power cycle or button) — NOT auto-resetting

Layer 2 (SLOW, backup): PPTC fuse in series with electrode output
  - 10mA hold, 30mA trip
  - Catches any fault that slips past Layer 1
  - Passive, firmware-independent, hardware-only
  - Auto-resets when cooled
```

**Parts needed (per channel):**
- 1× LM393DR (dual comparator — one IC for both channels) — $0.30
- 2× 10kΩ + 20kΩ resistor divider (0.6V ref from 5V) — $0.02
- 1× SN74LVC1G80 (or equivalent D flip-flop, wired as SR latch) — $0.20
- 1× BSS84 P-MOSFET (power supply kill switch) — $0.10
- 1× SMD0603B001TF PPTC (your existing selection) — backup

### ℹ️ Issue #8: ECQ-E2475KF Size for PCB

**Your selection:** ECQ-E2475KF — 4.7µF, **250VDC**, 26×12×21.5mm

**Issue:** 250V rating is massive overkill for a ±15V circuit (max voltage across cap ≈ 15V). The physical size (26mm long) dominates the PCB layout. This is a prototype-phase pragmatic choice but needs replacement for the PCB.

**For prototype (breadboard):** Keep ECQ-E2475KF — it works and you may already have them.

**For PCB:** Switch to **ECQ-E1475KF** (4.7µF, **100V**, same Panasonic family)
- Smaller body: 18×10×15.5mm
- 100V is still 6.7× headroom over max operating voltage — plenty
- Available on Farnell (428 in stock per PHASE1-BOM)
- Use **2× in parallel** per channel = 9.4µF (per PHASE1-BOM recommendation)

**Note on capacitor type:** Film capacitors are CORRECT and mandatory here. Never ceramic (voltage coefficient, piezoelectric effects at low signal levels) or electrolytic (polarized, high ESR, leakage). The metallized polyester film type (MKT) is ideal for DC-blocking in this application.

### ℹ️ Issue #9: Voltage Reference for MCP4922

**Not in your BOM but needed:** The MCP4922 requires an external voltage reference on Vref. The quality of this reference directly determines current accuracy and drift.

**RECOMMENDED: MCP1501-20E/SN** (Microchip, 2.048V reference)
- Accuracy: ±0.08% (initial)
- Tempco: 10 ppm/°C
- Output current: ±10mA (drives MCP4922 Vref easily)
- Supply: 2.5V–5.5V → powered from isolated 5V rail
- SOIC-8
- Cost: ~$1.50

**Why 2.048V?** With Rsense = 100Ω and Vref = 2.048V:
```
I_max = Vref / Rsense = 2.048V / 100Ω = 20.48mA (way above 5mA clinical max)
Resolution = 20.48mA / 4096 = 5µA per LSB
At 5mA clinical max: DAC code = 5mA / 20.48mA × 4096 = 1000 (out of 4096)
At 0.1mA min step: DAC code change = ~5 LSBs → smooth control
```

This gives 10-bit effective resolution within the clinical range — more than adequate.

### ℹ️ Issue #10: V-to-I Loop Stability & Timing Analysis

**Is the loop stable?** Yes.

The V-to-I converter with sense resistor feedback is a unity-gain voltage follower from the op-amp's perspective (V- tracks V+). For a unity-gain stable op-amp (ADA4522-2 IS unity-gain stable), the loop is unconditionally stable regardless of load impedance. The electrode capacitance (10–100nF) is in series with the load path, not in the feedback path — it doesn't create additional poles in the feedback loop.

**Compensation needed?** None for the basic loop. Add a 100pF cap across Rsense if bench testing shows ringing on pulse edges (unlikely with ADA4522-2 at 2.7MHz GBW).

**Is 1.4V/µs slew rate adequate?** Yes, with margin.

```
For a 200µs pulse with 10µs rise target:
  V_step = 5mA × 100Ω = 0.5V (sense voltage change)
  Slew time = 0.5V / 1.4V/µs = 0.36µs
  
200µs pulse width vs 0.36µs rise time → 556:1 ratio. Excellent.
```

**Does MCP4922 settling time matter?** No.

```
MCP4922 settling: 4.5µs to ±0.5 LSB
RC filter after DAC: 1kΩ + 100nF = τ = 100µs, fc = 1.6kHz
Filter settling to 99%: 5τ = 500µs

But wait — the RC filter is BETWEEN the DAC and the op-amp input.
For a 200µs pulse, the DAC updates BEFORE the pulse starts (during the GAP/REST period).
The ISR writes the DAC code, waits for the 50µs gap, THEN switches the H-bridge.
By the time the H-bridge switches, the DAC output has settled through the RC filter.

Timing sequence (from architecture):
  1. Write DAC code via SPI (~2µs)
  2. Pulse LDAC (~100ns)  
  3. Wait 50µs GAP ← DAC settles during this time (4.5µs)
  4. Switch H-bridge → PHASE_POS begins with stable current setpoint

No timing issue.
```

---

## 4. Binaural Dual-Channel Architecture

### What's Shared vs Duplicated

| Component | Quantity | Shared/Independent | Notes |
|-----------|---------|-------------------|-------|
| **MCP4922 DAC** | 1× | Shared (dual-channel IC) | Ch A = left ear, Ch B = right ear. Independent 12-bit outputs. LDAC synchronizes both. |
| **ADA4522-2 op-amp** | 1× | Shared IC, independent channels | Dual package — one amp per channel. Each has independent feedback loop. |
| **H-bridge MOSFETs** | 2 sets of 4 | Independent per channel | Each channel has its own full H-bridge. Cannot multiplex — 20ms stagger requires simultaneous operation. |
| **Rsense (100Ω)** | 2× | Independent per channel | Each channel measures its own current. |
| **DC-blocking cap (4.7µF)** | 4× (2 per channel) | Independent per channel | 2× in parallel per channel = 9.4µF. |
| **Bleed resistor (1MΩ)** | 2× | Independent per channel | One per DC-blocking cap pair (across parallel pair). |
| **PPTC fuse (10mA)** | 2× | Independent per channel | One per electrode output. |
| **LM393 comparator** | 1× | Shared IC (dual comparator) | One comparator per channel, both in one IC. |
| **B0515D-1WR3 DC-DC** | 1× | Shared | Both channels draw from same ±15V. At 10mA total stim, well within 34mA/rail. |
| **78L05 LDO** | 1× | Shared | Isolated 5V for DAC + isolators + ADC. |
| **MCP1501-20 Vref** | 1× | Shared | Both DAC channels use same reference. |
| **MCP3201 ADC** | 1× | Shared (multiplexed) | Impedance measurement alternates between channels during REST periods. |
| **Si8641/Si8622 isolators** | 4× total | Shared | Single bank handles all signals for both channels. |
| **TPS61023 boost** | 1× | Shared | System-level, powers everything. |
| **ME6211 LDO** | 1× | Shared | 3.3V for ESP32-S3 and digital side. |

**Binaural stagger is handled by the single hardware timer ISR** — Channel A fires at t=0, Channel B fires at t=stagger_offset (default 20ms). Both channel state machines run in the same ISR. This guarantees deterministic stagger with zero drift.

---

## 5. Complete Corrected BOM

### Power Domain (Digital Side)

| # | Component | MPN | Qty | Purpose | Package | Price Est. |
|---|-----------|-----|-----|---------|---------|-----------|
| 1 | LDO 3.3V | **ME6211C33M5G** | 1 | ESP32-S3 + digital logic 3.3V | SOT-23-5 | $0.30 |
| 2 | Boost converter | **TPS61023DRLR** | 1 | LiPo 3.0–4.2V → 5.0V | SOT-23-5 | $1.50 |
| 2a | Boost inductor | 1µH, 1A, ≤100mΩ DCR | 1 | TPS61023 external | 3×3mm shielded | $0.50 |
| 2b | Boost caps | 10µF 10V X5R ceramic | 2 | Input + output decoupling | 0805 | $0.20 |

### Isolation Barrier

| # | Component | MPN | Qty | Purpose | Package | Price Est. |
|---|-----------|-----|-----|---------|---------|-----------|
| 3 | Isolated DC-DC | **B0515D-1WR3** | 1 | 5V → ±15V, 1W, **1500VDC** isolation | SIP-6 TH | $5.00 |
| 4 | Digital isolator (4-ch) | **Si8641BB-D-IS** | 3 | 12 forward channels (SPI + H-bridge + heartbeat) | SOIC-16 | $3.00 ea |
| 5 | Digital isolator (2-ch bidir) | **SI8622EC-B-IS** | 1 | 2 reverse channels (fault + MISO) | SOIC-8 | $3.00 |

### Isolated Side (Patient Domain) — Power

| # | Component | MPN | Qty | Purpose | Package | Price Est. |
|---|-----------|-----|-----|---------|---------|-----------|
| 6 | LDO 5V (isolated) | **MC78L05ACPG** | 1 | +15V → +5V for DAC, isolator Vdd2, ADC | TO-92 | $0.30 |
| 7 | Voltage reference | **MCP1501-20E/SN** | 1 | 2.048V precision Vref for MCP4922 | SOIC-8 | $1.50 |

### Isolated Side — Signal Chain (per channel, ×2)

| # | Component | MPN | Qty | Purpose | Package | Price Est. |
|---|-----------|-----|-----|---------|---------|-----------|
| 8 | DAC (dual 12-bit) | **MCP4922-E/P** | 1 | Current setpoint (both channels) | PDIP-14 TH | $3.00 |
| 9 | Op-amp (zero-drift dual) | **ADA4522-2ARZ** | 1 | V-to-I error amp (both channels) | SOIC-8 | $5.50 |
| 10 | Sense resistor | **YR1B100RCC** (or equiv 100Ω 0.1% 15ppm) | 2 | Current feedback | Axial TH | $1.50 ea |
| 11 | H-bridge P-MOSFET | **BSS84** (50V, 130mA) | 4 | High-side switches (2 per channel) | SOT-23 | $0.10 ea |
| 12 | H-bridge N-MOSFET | **BSS138** (50V, 220mA) | 4 | Low-side switches (2 per channel) | SOT-23 | $0.10 ea |
| 13 | Gate resistors | 1kΩ 1% | 8 | Gate drive current limiting | 0402/0603 | $0.01 ea |
| 14 | RC filter R | 1kΩ 1% | 2 | DAC output filter (1 per channel) | 0603 | $0.01 ea |
| 15 | RC filter C | 100nF C0G/NP0 | 2 | DAC output filter (1 per channel) | 0603 | $0.05 ea |
| 16 | ADC (impedance) | **MCP3201-CI/SN** | 1 | 12-bit SPI ADC for Z measurement | SOIC-8 | $2.00 |

### Safety Components (per channel, ×2)

| # | Component | MPN | Qty | Purpose | Package | Price Est. |
|---|-----------|-----|-----|---------|---------|-----------|
| 17 | DC-blocking cap | **ECQ-E1475KF** (prototype: ECQ-E2475KF OK) | 4 | 4.7µF 100V film (2 parallel per ch) | Radial TH | $1.50 ea |
| 18 | Bleed resistor | 1MΩ 1% metal film | 2 | Discharge DC-block cap during rest | Axial TH | $0.05 ea |
| 19 | PPTC fuse | **SMD0603B001TF** (or MF-MSMF010-2 1812 for prototype) | 2 | 10mA hold / 30mA trip backup fuse | 0603 SMD | $0.40 ea |
| 20 | Overcurrent comparator | **LM393DR** | 1 | Dual comparator (1 per channel) | SOIC-8 | $0.30 |
| 21 | Fault latch | **SN74LVC1G80** or discrete SR latch | 2 | Latches overcurrent fault | SOT-23-5 | $0.20 ea |
| 22 | Power kill MOSFET | **BSS84** | 2 | Cuts isolated analog power on fault | SOT-23 | $0.10 ea |

### Electrode Interface

| # | Component | MPN | Qty | Purpose | Package | Price Est. |
|---|-----------|-----|-----|---------|---------|-----------|
| 23 | Electrode connector | 2-pin JST-PH | 2 | Snap-on electrode connection per ear | TH | $0.20 ea |

### Decoupling (standard, distributed across board)

| # | Component | Qty | Notes |
|---|-----------|-----|-------|
| 24 | 100nF X7R ceramic 0603 | 12 | Every IC Vdd pin |
| 25 | 10µF X5R ceramic 0805 | 4 | B0515D output (±15V), 78L05 in/out |
| 26 | 100µF electrolytic 25V | 2 | Bulk on ±15V rails |
| 27 | 1nF C0G ceramic 0603 | 2 | High-frequency decoupling on ±15V |

---

### BOM Cost Summary

| Category | Cost |
|----------|------|
| Power (boost + LDOs + DC-DC + Vref) | ~$9 |
| Isolation (4 ICs) | ~$12 |
| Signal chain (DAC + op-amp + H-bridges + sense) | ~$15 |
| Safety (caps + fuses + comparator + latches) | ~$10 |
| Passives (decoupling + resistors + connectors) | ~$5 |
| **Total analog BOM** | **~$51** |

ESP32-S3 DevKitC-1 (~$8), LiPo + TP4056 (~$5), electrodes (~$10) add ~$23.
**Grand total BOM estimate: ~$74** — within the $100 target.

---

## 6. Design Changes Summary

| # | What Changed | Why | Risk if Not Fixed |
|---|-------------|-----|-------------------|
| 🔴 1 | TMA 0515D → **B0515D-1WR3** | 1kV isolation insufficient; need 1500VDC | Patient safety — isolation barrier compromised |
| 🔴 2 | AD8628ARZ → **ADA4522-2ARZ** | AD8628 max 5V supply — destroyed on ±15V; can't drive compliance voltage | Circuit does not function; op-amp destroyed on power-up |
| 🔴 3 | Added **TPS61023DRLR** boost converter | LiPo (3.0–4.2V) below DC-DC minimum (4.5V) | Device cannot produce ±15V — no stimulation possible |
| 🟡 4 | DRV8871 → **discrete MOSFET H-bridge** (BSS84 + BSS138) | DRV8871 min Vs = 6.5V — incompatible with V-to-I topology at low currents | Current regulation fails at low intensities; no soft-start possible |
| 🟡 5 | Added **MC78L05ACPG** isolated-side LDO | No 5V available on isolated side for DAC/ADC/isolator Vdd2 | MCP4922, MCP3201, and isolator Vdd2 have no supply — circuit dead |
| ℹ️ 6 | SI8622EC ×1 → **Si8641BB ×3 + SI8622EC ×1** | Need 11 isolated channels; SI8622EC only has 2 | Insufficient signals across barrier — can't control both channels |
| ℹ️ 7 | Added **LM393DR** + latch for overcurrent | PPTC alone is too slow (<10µs cutoff required) | Overcurrent cutoff too slow — safety gap |
| ℹ️ 8 | Added **MCP1501-20E/SN** voltage reference | MCP4922 needs clean, stable Vref for current accuracy | Current setpoint drifts with temperature and supply noise |

---

## 7. Unchanged (Confirmed Correct)

| Component | MPN | Verdict |
|-----------|-----|---------|
| ME6211C33M5G (3.3V LDO) | — | ✅ Correct. 100mV dropout, LiPo compatible, excellent standby current. |
| MCP4922-E/P (DAC) | — | ✅ Correct. Dual 12-bit SPI, covers both channels, LDAC for atomic update. |
| YR1B100RCC (100Ω sense R) | — | ✅ Correct. 0.1%, 15ppm tempco, 250mW. Perfect for V-to-I feedback. |
| SMD0603B001TF (PPTC fuse) | — | ✅ Correct as backup protection layer. Add LM393 fast comparator as primary. |
| ECQ-E2475KF (DC-block cap) | — | ✅ Functional but oversized. Swap to ECQ-E1475KF (100V) for PCB phase. |
| SI8622EC-B-IS (isolator) | — | ✅ Correct as one of multiple isolator ICs. Excellent 3750Vrms isolation. Need more units or complement with Si8641. |

---

## 8. Open Items for Phase Execution

1. **MCPWM dead-time tuning**: Configure ESP32-S3 MCPWM with ≥500ns dead-time. Verify on oscilloscope during Phase 2 bringup. The BSS84/BSS138 turn-on/off times are ~10ns — 500ns dead-time gives 50× margin.

2. **LC filter on ±15V rails**: The B0515D-1WR3 produces 50–100mV ripple at ~100kHz. Add per-rail: 10µH inductor + 10µF ceramic + 100µF electrolytic. Verify <10mV ripple at op-amp supply pins.

3. **DAC Vref calibration**: The MCP1501-20 is ±0.08% accurate. At 5mA, that's ±4µA error — acceptable. For tighter accuracy, add a firmware calibration step using the MCP3201 ADC to measure actual current and apply a trim factor.

4. **Impedance measurement**: MCP3201 on isolated side, sharing SPI bus with MCP4922 (different CS). Measure during REST period (39.4ms window). Average 4–8 samples for noise rejection.

5. **Thermal check**: 78L05 dissipates ~500mW at 50mA from 15V→5V. Verify TO-92 package temperature during sustained operation. If >80°C, add a small heatsink or switch to SOT-223 variant (MC78L05ABDR2G).

---

*This review should be updated after breadboard validation in Phase 2. Component choices may need adjustment based on bench measurements (noise, stability, thermal).*
