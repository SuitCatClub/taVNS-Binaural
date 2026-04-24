# Electrical Design Review — OpenBinaural-taVNS

**Date:** 2026-04-24
**Reviewer:** EE review agent (warm review — full context read)
**Documents reviewed:** AGENTS.md, DESIGN-REVIEW.md, CIRCUIT-DESIGN-REVIEW.md, CLINICAL-REVIEW.md, PROJECT.md, ROADMAP.md
**Scope:** Complete electrical design — topology, components, safety, power chain, signal chain, isolation barrier

---

## Executive Summary

This is a well-researched design that has clearly been through several correction cycles. The fundamentals are sound: V-to-I current source topology, galvanic isolation architecture, multi-layer safety approach, and component selection methodology are all solid. The team catches its own mistakes and documents corrections, which is the single most important quality in a medical device project.

That said, I found **one critical unresolved contradiction**, **three significant gaps**, and **several minor issues and confirmations**. Nothing here requires redesigning the topology. Most findings are about resolving document conflicts and closing loose ends before committing to a PCB.

**Bottom line:** This design will work. The open items are tractable. Fix the H-bridge question definitively, resolve the Vref approach, and plan the impedance sensing data path — then you're ready for breadboard.

---

## Finding 1: DRV8871 Minimum Voltage — The Contradiction Is Real

**Severity: 🔴 CRITICAL — Must be resolved before schematic capture**

This is the biggest open question in the design. Two project documents directly contradict each other:

- **DESIGN-REVIEW.md §1** says: *"The DRV8871DDAR specifically is correct"* and lists it in the final BOM (U7, U8)
- **CIRCUIT-DESIGN-REVIEW.md §2 CRITICAL #4** says: *"DRV8871 has a minimum supply voltage (Vs) of 6.5V... At 0.3V on VM, the DRV8871 is far below its 6.5V minimum operating voltage and will not function."*

**The CIRCUIT-DESIGN-REVIEW is correct.** Here's why:

In the V-to-I topology as drawn (DESIGN-REVIEW.md §8), the op-amp output connects directly to DRV8871's VM pin. The op-amp servo adjusts its output voltage until V(Rsense) = V(DAC). At low currents into low-impedance loads:

```
V_opamp_out = I × (Z_load + R_sense)
            = 0.5mA × (500Ω + 100Ω)
            = 0.3V
```

At 0.3V on VM, the DRV8871's internal gate drivers, charge pump, and control logic cannot function. The datasheet specifies 6.5V minimum for a reason — the internal N-channel high-side FET needs a charge pump referenced to VM, and below 6.5V that pump doesn't generate enough gate-source voltage to fully enhance the FET.

This isn't a borderline case. At soft-start (0mA ramping up through 0.5mA), the device passes through the non-functional region on every single session start. At the clinical sweet spot of 2mA into 1kΩ, VM = 2.2V — still well below 6.5V.

**The DRV8871 cannot work in this topology. The DESIGN-REVIEW.md BOM is wrong on this point, and AGENTS.md §2 still lists DRV8871DDAR as the correct part.**

### Resolution options (ranked)

**Option A — Discrete MOSFET H-bridge (RECOMMENDED):**

4× BSS84 (P-ch high-side) + 4× BSS138 (N-ch low-side) per 2-channel build. Gate drive from digital isolator outputs (5V_ISO level). Dead-time enforced by ESP32-S3 MCPWM peripheral (hardware-guaranteed, configurable ≥500ns).

| Pro | Con |
|-----|-----|
| No minimum voltage — works 0V to 50V | More components (8 FETs + 8 gate resistors) |
| Full ±15V compliance headroom | Layout-sensitive (keep gate traces short) |
| Dead-time is hardware-enforced (MCPWM) | Must verify dead-time on scope during bringup |
| Component cost < $1 total | Loses IPROPI current sense — need alt impedance method |
| Transparent to analog feedback loop | — |

**Option B — DRV8837DSGR (simpler, compliance-capped):**

VM range 0–11V. Integrated dead-time (~300ns). SOT-23-6. Drop-in for control signals.

| Pro | Con |
|-----|-----|
| No minimum voltage | Max VM = 11V → compliance capped at ~10.5V |
| Integrated protection | Z_max at 5mA = 2.1kΩ (vs 2.5kΩ with full ±15V) |
| Simple 1-IC per channel | Z_max at 2mA = 5.25kΩ — still adequate |

**Option C — Topology change: keep DRV8871 at fixed ±15V, add series pass element:**

DRV8871 VM = +15V (fixed). Op-amp drives a series N-MOSFET (e.g., 2N7002) between H-bridge output and load. The FET acts as the variable resistance element; the op-amp controls its gate to regulate current.

| Pro | Con |
|-----|-----|
| Keeps DRV8871 and its IPROPI feature | Adds complexity (1 FET + biasing per channel) |
| Full compliance headroom | Series FET Vds drop reduces compliance slightly |
| DRV8871 always within spec | More analysis needed for loop stability |
| IPROPI still available for impedance sensing | Pass FET needs to handle full ±15V |

**My recommendation:** Option A (discrete MOSFETs) for the final PCB. It's the cleanest solution — no minimum voltage, full compliance, and the MCPWM-enforced dead-time is actually *better* than the DRV8871's fixed internal dead-time because it's configurable. Option B (DRV8837) is acceptable for breadboard prototyping if you document the 11V compliance ceiling and its implications at high current + high impedance.

**Action required:** Update AGENTS.md §2, DESIGN-REVIEW.md BOM, and DESIGN-REVIEW.md §1 to remove DRV8871 and specify the chosen replacement. This is the single most important correction before hardware bringup.

---

## Finding 2: MCP4922 Voltage Reference — Two Incompatible Recommendations

**Severity: 🟡 SIGNIFICANT — Must be decided before schematic capture**

The two review documents propose different Vref strategies:

| Source | Approach | Vref | Gain | I_max | Resolution |
|--------|----------|------|------|-------|------------|
| CIRCUIT-DESIGN-REVIEW §Issue 9 | MCP1501-20E/SN precision ref IC | 2.048V | 1× | 20.48mA | 5.0 µA/LSB |
| DESIGN-REVIEW §11 Issue A | Resistor divider from TLV70450 | ~0.455V | 2× | 9.09mA | 2.2 µA/LSB |

**Neither is wrong, but they have different tradeoffs:**

The MCP1501-20 approach:
- ✅ Precision reference: ±0.08%, 10 ppm/°C — current accuracy is limited by Rsense, not Vref
- ✅ Dedicated IC with excellent PSRR (60dB) and load regulation
- ⚠️ At 5mA clinical max, you use only DAC codes 0–1000 out of 4096 (24% of range)
- ⚠️ Effective resolution in clinical range: ~10 bits
- ⚠️ Adds one more IC to isolated-side BOM

The resistor-divider approach:
- ✅ Zero additional ICs
- ✅ Full 12-bit resolution within clinical range
- ⚠️ Divider noise and drift couple directly to Vref → current accuracy
- ⚠️ TLV70450 noise (30µVrms) is excellent, but divider resistor thermal noise adds
- ⚠️ Using 2× gain mode doubles DAC noise contribution

**My recommendation:** Use the **MCP1501-20E/SN**. For a medical device, a dedicated precision reference is the right call. The 10-bit effective resolution (5µA/LSB) gives 0.1% current accuracy at 5mA — more than adequate for clinical stimulation where currents are titrated by feel. The extra $1.50 and one SOIC-8 footprint are trivial costs for clean, stable current setpoints.

However, if the 5µA step size feels too coarse at low currents (e.g., 0.5mA = code 100, steps of 5µA = 1% per step), consider **MCP1501-10E/SN** (1.024V output):
- I_max = 10.24mA (2× clinical max — good headroom)
- Resolution = 2.5 µA/LSB
- At 5mA: code = 2000 (49% of range — better utilization)
- At 0.5mA: code = 200, step size = 0.5% — smoother at low currents

**Action required:** Choose one Vref approach, add it to the BOM, and remove the other from the design documents.

---

## Finding 3: SPI MISO Path Missing for Impedance Sensing

**Severity: 🟡 SIGNIFICANT — Affects safety feature implementation**

The current isolator assignment uses all available channels:

| IC | Channels | Assignment |
|----|----------|------------|
| SI8380P-IU | 8 forward | SPI SCK, MOSI, CS, LDAC, IN1_A, IN2_A, IN1_B, IN2_B |
| SI8621EC-B-IS | 1 forward + 1 reverse | Heartbeat (F) + Fault flag (R) |
| **Total** | **9F + 1R** | **No MISO** |

If an ADC (MCP3201 or similar) is placed on the isolated side for proportional impedance measurement, its SPI MISO data needs a reverse channel back to the ESP32. There is no available reverse channel.

Simultaneously, if the DRV8871 is replaced by discrete MOSFETs (Finding 1, Option A), the IPROPI current sense output is lost. Impedance measurement then requires measuring the op-amp output voltage or the Rsense voltage with a dedicated ADC — which needs MISO.

**Resolution options:**

**Option A — Add a second SI8621EC-B-IS:**
- Gives 1 additional forward + 1 reverse channel
- Reverse channel = MISO for ADC
- Forward channel = spare (or second fault signal)
- Cost: +$3, one more SOIC-8
- **This is the cleanest solution.** You get proportional impedance measurement with full resolution.

**Option B — Threshold-only impedance detection (no ADC):**
- Use the LM339 comparator with two reference voltages per channel:
  - One comparator checks "op-amp output voltage too high" → Z > limit → open electrode
  - One comparator checks "op-amp output voltage too low" → Z < limit → short/dry electrode
- Fault output feeds into the existing composite fault flag (reverse channel)
- **LM339 has 4 comparators:** overcurrent A, overcurrent B, and now Z_high + Z_low? That's 4 channels for two channels of overcurrent + two channels of impedance — you'd need 2× LM339, or prioritize which checks get hardware comparators.
- Alternative: use 2 LM339 comparators for overcurrent (one per channel), 1 for heartbeat watchdog, and 1 for a multiplexed impedance check (mux between channels during REST).

Actually, looking at the current allocation: LM339 has (1) overcurrent A, (2) overcurrent B, (3) heartbeat watchdog, (4) spare. That spare channel could be a single impedance threshold comparator, multiplexed between channels.

**My recommendation:** Option A if you want proper impedance readback for logging and display. Option B if you're willing to accept threshold-only detection (which is what most commercial stimulators do anyway — they just say "electrode error", they don't tell you the impedance value).

For v1, Option B is probably sufficient. But plan the PCB footprint for a second SI8621EC-B-IS as a DNP (do-not-populate) option for future upgrade.

**Action required:** Decide on impedance measurement approach. If proportional: add second SI8621EC-B-IS + MCP3201 to BOM. If threshold-only: document which LM339 channel is used and the reference voltage settings.

---

## Finding 4: DC-Blocking Capacitor — MLCC vs Film Debate

**Severity: 🟡 SIGNIFICANT for documentation; electrically acceptable as-is**

CIRCUIT-DESIGN-REVIEW.md §Issue 8 explicitly states: *"Film capacitors are CORRECT and mandatory here. Never ceramic."*

DESIGN-REVIEW.md §6 selected Samsung CL31B106KBHNNNE — an MLCC X7R ceramic — and argues it's acceptable because operating voltage is <100mV.

**The DESIGN-REVIEW analysis is technically correct.** At <100mV bias:
- X7R voltage coefficient: negligible (<1% capacitance change)
- Piezoelectric effect: negligible mechanical stress
- Leakage current: >10GΩ insulation resistance — DC blocking is effective
- Temperature drift: ±15% over full range, but the cap only needs to pass AC and block DC

**However, I want to flag a subtlety that neither document addresses:**

X7R MLCCs have **hysteresis in their voltage-capacitance curve**. During biphasic operation, the cap sees alternating voltage swings. While the average is ~0V, the instantaneous voltage swings create a micro-hysteresis loop. In an X7R, this hysteresis is lossy (dielectric absorption / soakage). The absorbed charge from one phase may not fully release during the opposite phase, creating a tiny net DC offset over many cycles.

**How much offset?** X7R dielectric absorption is typically 2–3%. For a 100mV swing, that's 2–3mV residual. Through 100Ω Rsense, that's 20–30µA peak transient. This decays within a few milliseconds and is further attenuated by the 1MΩ bleed resistor (though very slowly at that time constant).

**Is this clinically significant?** No. 30µA is well below perception threshold (~100µA for most people at the cymba conchae). And it's a decaying transient, not a sustained DC current.

**My verdict:** The Samsung CL31B106KBHNNNE is acceptable for the PCB build. The 700× size reduction over the film cap is a legitimate design win. But I'd recommend:

1. **Document the rationale** — Add a design note explaining why MLCC was chosen over film, with the dielectric absorption analysis. Future reviewers (or certification bodies, if it ever comes to that) will ask this question.

2. **Consider C0G/NP0 at reduced capacitance as a supplementary option.** You can't get 10µF in C0G, but you could use 2× 4.7µF C0G in a larger package (2220 or similar) if they exist at 50V. They may not — C0G above 1µF is rare. Worth checking but not blocking.

3. **If you ever find a compact film cap that works:** TDK has B32529-series 10µF/50V metallized polyester in relatively small packages (7.5mm pitch radial). Not SMD, but for a mixed-technology PCB it's an option. I wouldn't chase this for v1.

**Action required:** Document the MLCC decision rationale in the design review. Resolve the contradiction between CIRCUIT-DESIGN-REVIEW (says film mandatory) and DESIGN-REVIEW (says MLCC acceptable). One of them needs to yield.

---

## Finding 5: Emergency Stop Switch — UX/Safety Concern

**Severity: 🟡 SIGNIFICANT — Affects user safety during fault events**

The BOM lists "Generic tactile NO switch" for emergency stop (SW1). A typical tactile switch is 6mm × 6mm with ~1mm travel and requires deliberate finger placement.

For a device that delivers electrical current to a person's ear during sleep or rest:

- The user may be lying down in a dark room
- Their fine motor control may be impaired by the very stimulation they're trying to stop
- Panic situations demand large, unambiguous mechanical controls

**Recommendation:** Replace the tactile switch with one of:

1. **Slider/toggle switch in the power path** — physically disconnects the LiPo. User slides to OFF. Unambiguous state (can feel position in the dark). Example: ALPS SSSS213100 or similar SPDT slide switch.

2. **Large pushbutton (≥12mm)** — still momentary, but physically large and raised from the enclosure surface. Easier to find by feel. Example: E-Switch TL1100 series.

3. **Magnetic reed kill switch** — remove a magnetic clip to disconnect. Used in treadmill emergency stops. Novel for this application but very intuitive.

The emergency stop should break the power path as close to the patient as possible — ideally on the isolated side. If it kills the TPS61023 EN pin (like the USB interlock does), the ±15V rail drops and stimulation stops within the decay time of the output capacitors (~50ms for 47µF × 10kΩ bleed).

**Better: break the H-bridge drive signals directly.** A normally-closed switch in series with the H-bridge enable or gate-drive signal gives immediate (< 1µs) current cessation. The capacitor decay issue doesn't apply because the H-bridge actively sinks the load to ground when disabled.

**Action required:** Specify an emergency stop switch appropriate for bedside medical use. Document the response time from button press to zero patient current.

---

## Finding 6: PCN1-S5-D15-M-TR Output Regulation & Startup Sequencing

**Severity: 🟡 MINOR-to-SIGNIFICANT depending on load conditions**

The PCN1-S5-D15-M-TR is an **unregulated** isolated DC-DC. Its output voltage varies with load and input:

- At light load: output may be higher than ±15V (possibly ±17–18V)
- At heavy load near rated current: output droops below ±15V (possibly ±12–13V)
- At no load: output could overshoot significantly at startup

**Implications:**

1. **ADA4522-2ARZ absolute max is ±27.5V (55V total).** Even at ±18V (worst-case light load), the op-amp is safe. ✅

2. **Compliance headroom varies with actual output voltage.** If the ±15V droops to ±13V under load, compliance drops:
   - V_compliance = 13V - 1.5V (saturation) - 0.5V (Rsense) = 11V
   - Z_max at 5mA = 2.2kΩ (down from 2.5kΩ at nominal ±15V)
   - Still adequate for clinical use (well-prepared electrodes are 500Ω–3kΩ)

3. **Startup overshoot concern:** When TPS61023 EN goes HIGH, the 5V rail ramps up, and the PCN1 starts converting. With light/no load on the isolated side, the ±15V outputs may overshoot during the first few milliseconds. The 47µF + 100nF decoupling on each rail should absorb this, but verify on scope.

4. **Cross-regulation:** With asymmetric loading (+15V rail draws more than -15V rail), the unregulated converter may produce unequal outputs. E.g., +13V and -16V instead of ±15V. The ADA4522-2 can handle this (its supply range is asymmetric-tolerant), but the compliance calculation changes.

**Recommendation:**
- Add a note to verify PCN1 output voltages at actual load during bench bringup
- Consider adding 15V Zener diodes (BZX84C15, SOT-23) on each rail as voltage clamps — they dissipate nothing during normal operation and protect against startup overshoot or light-load overvoltage. Cost: $0.05 each.

**Action required:** Characterize PCN1-S5-D15-M-TR output voltage range at expected load during Phase 2 bringup. Add Zener clamp diodes to BOM as insurance.

---

## Finding 7: Power Budget — Overestimated But Not Wrong

**Severity: ℹ️ INFO — Correct for sizing, misleading for battery life**

DESIGN-REVIEW.md §2 states PCN1-S5-D15-M-TR draws 256mA from the 5V boost rail. This is the **maximum converter input at full 1W output**, not the actual load.

Actual isolated-side power budget:

| Consumer | Current from ±15V | Notes |
|----------|-------------------|-------|
| ADA4522-2ARZ (quiescent) | ~2mA per channel | From +15V and -15V |
| Stimulation load (both channels) | ~10mA peak | 5mA × 2 channels |
| TLV70450 load (5V_ISO consumers) | ~12mA from +15V | MCP4922 + isolators + Vref + LM339 |
| **+15V rail total** | ~24mA | Within 33mA per-rail limit |
| **-15V rail total** | ~12mA | Within 33mA per-rail limit |

Total output power: (24mA + 12mA) × 15V = 540mW (54% of 1W rating).
Input at 78% efficiency: 540mW / 0.78 = 692mW → **138mA from 5V boost** (not 256mA).

Corrected battery life:
- ME6211 (direct LiPo): ~150mA at 3.3V
- Boost converter (LiPo→5V→isolated): 138mA × 5V / 3.7V / 0.93 ≈ 200mA from LiPo
- **Total: ~350mA from LiPo → 2000mAh / 350mA ≈ 5.7 hours** (~11 sessions at 30 min)

The DESIGN-REVIEW's 6-hour estimate at 330mA is close enough. The difference (330mA vs 350mA) is within estimation uncertainty.

**Action required:** None critical. Optionally correct the 256mA figure in DESIGN-REVIEW.md to reflect actual load (~138mA) vs maximum (~256mA). Both numbers are useful — just label them clearly.

---

## Finding 8: Document Inconsistencies (Housekeeping)

**Severity: ℹ️ INFO — No electrical impact, but creates confusion**

### 8a. TPS61023 package designation

| Document | Package listed |
|----------|---------------|
| AGENTS.md | SOT-563 |
| CIRCUIT-DESIGN-REVIEW.md | SOT-23-5 |
| DESIGN-REVIEW.md | SOT-5 (1.6×1.6mm) |

The correct package for TPS61023DRLR is **SOT-563** (6-pin, 1.6×1.2mm). AGENTS.md is correct. The other two documents have the wrong package name. This matters for footprint selection in KiCad.

### 8b. TPS61023 switching frequency

| Document | Frequency |
|----------|-----------|
| CIRCUIT-DESIGN-REVIEW.md | 1.8 MHz |
| DESIGN-REVIEW.md | ~4 MHz |
| TI datasheet (TPS61023) | **4.0 MHz typical** |

The DESIGN-REVIEW is correct. The CIRCUIT-DESIGN-REVIEW's 1.8 MHz appears to be from a different variant or an error. This matters for inductor selection (at 4 MHz, the specified 2.2µH Würth 744043002 is appropriate — at 1.8 MHz you'd want a larger inductor).

### 8c. Overcurrent comparator part number

CIRCUIT-DESIGN-REVIEW.md §Issue 7 specifies LM393DR (dual comparator). DESIGN-REVIEW.md §11 Issue D upgrades to LM339DR (quad). AGENTS.md §BOM lists LM339DR. The CIRCUIT-DESIGN-REVIEW needs updating — LM339DR is the correct choice (need 4 channels: 2× overcurrent, 1× watchdog, 1× spare/impedance).

### 8d. BOM reference IC across CIRCUIT-DESIGN-REVIEW vs DESIGN-REVIEW

The CIRCUIT-DESIGN-REVIEW.md BOM (§5) still lists B0515D-1WR3, Si8641BB-IS, MC78L05ACPG, MCP3201, and discrete MOSFET H-bridge — while the DESIGN-REVIEW.md BOM (§10) lists PCN1-S5-D15-M-TR, SI8380P-IU, TLV70450DBVR, DRV8871, and no MCP3201. These are clearly from different design iterations. **One canonical BOM should be the source of truth.** I'd suggest the DESIGN-REVIEW.md BOM is newer and should be canonical, with the DRV8871 correction from Finding 1 applied.

**Action required:** Reconcile document inconsistencies. Establish one BOM as canonical. Add a version/date to each document's header so readers know which is newer.

---

## Finding 9: LM339 Supply Voltage and Open-Collector Behavior

**Severity: ℹ️ MINOR — Design note, not a problem**

The BOM shows LM339DR powered on the isolated side. The LM339 can operate from a single supply (3V–36V) or split supply. On the isolated side, you have +15V, -15V, 5V_ISO, and GND_ISO.

**The LM339 should be powered from 5V_ISO (VCC) and GND_ISO (GND)**, not from ±15V. Reasons:
- The LM339 comparator inputs monitor Rsense voltage (0–0.5V) and heartbeat signal (~3.3V logic level) — all within the 0–5V range
- Running LM339 from ±15V (30V total) is within its 36V max, but the open-collector output pull-up resistor then needs to connect to 5V_ISO anyway (for logic-level compatibility with the SR latch or isolator input)
- Lower supply = lower power dissipation

**The open-collector outputs need pull-up resistors** (10kΩ to 5V_ISO). Make sure these are in the schematic — LM339 outputs can only sink current, not source it. Without pull-ups, the fault/watchdog signals stay floating.

**For the overcurrent latch:** The LM339 open-collector output going LOW (overcurrent detected) should SET an SR latch. The latch output then kills the H-bridge. The latch must be hardware-reset only (power cycle or dedicated reset button — not firmware-resettable). Make sure the latch power-up state is "safe" (not latched, stimulation disabled until explicitly started).

**Action required:** None critical. Verify pull-up resistors are specified for all LM339 outputs. Verify latch power-up default state.

---

## Finding 10: Bleed Resistor Time Constant

**Severity: ℹ️ MINOR — Functional but not optimal**

R_bleed = 1MΩ across 10µF DC-blocking cap → τ = 10 seconds → 5τ = 50 seconds to fully discharge.

After stimulation stops, any residual charge on the DC-blocking cap bleeds through 1MΩ. The maximum stored charge is tiny (worst case: ~100mV on 10µF = 1µC → discharge current = 100mV / 1MΩ = 0.1µA), so this is not a safety concern.

**However**, during active biphasic stimulation, the bleed resistor provides a continuous DC offset discharge path. At 25Hz stimulation, each biphasic cycle rebalances the cap. But during the 30-second OFF period in duty-cycled operation, the bleed resistor is the only discharge path. With τ = 10s, after 30s OFF, the cap retains only e^(-30/10) = 5% of any accumulated offset. This is fine.

**Minor optimization:** A lower bleed resistance (100kΩ instead of 1MΩ) gives τ = 1s, meaning the cap is fully discharged within 5s. This is more aggressive but still draws negligible current (100mV / 100kΩ = 1µA). The tradeoff is a slightly higher DC load on the current source during stimulation (1µA through the bleed resistor vs 0.1µA). At 5mA stimulation current, 1µA is 0.02% error — undetectable.

**Action required:** Optional — consider reducing bleed resistor to 100kΩ for faster cap discharge. Not critical.

---

## Finding 11: Charge Balance Under Pulse Timing Asymmetry

**Severity: ℹ️ MINOR — Already well-handled, but worth documenting the analysis**

Charge balance is the most important safety property of this device. The design handles it at three levels:

1. **Firmware:** Same DAC code for both phases, same timer duration → I₊ × t₊ = I₋ × t₋
2. **Hardware:** DC-blocking cap prevents net DC regardless of firmware behavior
3. **Safety margin:** 10µF cap allows only 100mV offset per µC of charge imbalance

**Potential asymmetry sources:**

| Source | Magnitude | Impact |
|--------|-----------|--------|
| H-bridge dead-time (500ns–1µs) | 0.25–0.5% of 200µs pulse | Negligible — same dead-time on both transitions |
| Op-amp slew rate asymmetry (positive vs negative) | <0.1µs typical for ADA4522-2 | Negligible |
| DAC code write timing jitter | <2µs (SPI transfer) | ISR writes before phase starts; settled during 50µs gap |
| H-bridge Rdson mismatch (BSS84 vs BSS138) | ~10-50mΩ at 5mA → ~50-250µV | Completely negligible |
| Rsense self-heating | ΔR = 25ppm/°C × ΔT; P = 5mA² × 100Ω = 2.5mW → ΔT < 1°C | <0.003% resistance change → negligible |

**Worst-case cumulative charge imbalance per cycle:** < 0.5% of 1µC/phase = < 5nC. On the 10µF cap, this is 0.5µV per cycle. At 25Hz for 30 minutes (45,000 cycles), cumulative offset = 45,000 × 0.5µV = 22.5mV — if it accumulated monotonically (it won't, because the bleed resistor and thermal/statistical averaging prevent monotonic drift).

**Verdict:** Charge balance is robust. The three-layer approach (firmware symmetry + hardware DC-blocking + physical charge density margin) is textbook correct. No action needed.

---

## Finding 12: SI8380P-IU Propagation Delay & SPI Timing

**Severity: ℹ️ INFO — No issue found, confirming correctness**

SI8380P-IU maximum propagation delay: ~20ns typical, ~50ns max. At the MCP4922's maximum SPI clock rate of 20 MHz (50ns period), the isolator delay approaches half a clock period.

**But the ESP32-S3's SPI peripheral drives MCP4922 at ~1 MHz (per project documentation), not 20 MHz.** At 1 MHz (1000ns period), the 50ns isolator delay is 5% of the clock period — well within timing margin. Setup and hold times for MCP4922 are ~40ns and ~15ns respectively; the isolator adds ~50ns to each signal, which shifts all signals equally (SCK, MOSI, CS all go through the same isolator IC) — relative timing is preserved.

The only timing concern would be if SCK and MOSI went through different isolator ICs with different propagation delays. Since both go through the SI8380P-IU, channel-to-channel skew is specified at ~1ns — effectively zero.

**Verdict:** SPI through the SI8380P-IU isolator is clean at 1 MHz. No timing issue. ✅

---

## Finding 13: BSS138 USB Interlock — Verify Gate Threshold vs VBUS Voltage

**Severity: ℹ️ MINOR — Likely fine, worth checking**

The USB interlock uses BSS138 with USB VBUS (5V) on the gate. BSS138 Vgs(th) = 0.8V–1.5V (varies by manufacturer). At 5V gate drive, the FET is fully enhanced. ✅

**But:** USB VBUS may not be a clean 5V. During cable insertion/removal, VBUS can ring or droop. And some USB chargers produce 4.5V–5.5V. None of this affects the BSS138 (it just needs > 1.5V to turn on).

**One consideration:** When VBUS is removed (USB disconnected), the BSS138 gate must return to 0V. The gate has high impedance — add a 100kΩ pull-down resistor from gate to source (GND) to ensure the gate discharges. Without it, parasitic capacitance could hold the gate above Vgs(th) for several seconds after USB disconnect, delaying stimulation re-enablement.

Also verify that the TPS61023 EN pin has a defined pull-up (10kΩ to VBAT or VIN) so that when BSS138 is OFF (no USB), EN is HIGH and the boost converter runs.

**Action required:** Verify BSS138 gate pull-down and TPS61023 EN pull-up are in the schematic.

---

## Confirmed Correct — What's Solid

These elements have been reviewed and are correct. No changes needed.

### ✅ V-to-I Topology

The V-to-I converter with sense resistor feedback and H-bridge polarity switching is the right choice over Howland pump. The stability analysis (89° phase margin, Rsense zero cancels electrode pole) is correct. The inherent advantage — load impedance variations only change op-amp output voltage, not loop stability — is real and significant for a device contacting variable-impedance biological tissue.

### ✅ ADA4522-2ARZ Op-Amp

Zero-drift, ±27.5V supply, dual channel, 2.7 MHz GBW, 1.7V/µs slew rate. Perfect for this application. The slew rate analysis (worst-case 14µs rise into 3kΩ load, 7% of 200µs pulse) is correct and acceptable. The 10nF cap across Rsense (fc = 159kHz) is good practice for HF noise rejection without affecting the 20-25Hz stimulation bandwidth.

### ✅ ME6211C33M5G LDO

100mV dropout handles the full LiPo range (3.0–4.2V → 3.3V from 3.4V input minimum). The decision to power it directly from LiPo (not from 5V boost) is correct — avoids double conversion loss and provides independent 3.3V even if the boost converter is off (USB interlock active).

### ✅ TPS61023 Boost Converter

Mandatory for bridging the LiPo (max 4.2V) to PCN1 (min 4.5V) gap. 4 MHz switching frequency is a smart choice — lower output ripple for the same capacitor size, less noise coupling into the analog domain. BSS138 interlock on EN pin is clean.

### ✅ PCN1-S5-D15-M-TR Isolated DC-DC

1500VDC isolation meets 1 MOPP for battery-powered Type BF. SMD-8 package is PCB-friendly. Dual ±15V output provides adequate compliance headroom. The weakest-link analysis (PCN1 at 1500VDC limits the barrier, not the 2500/3750Vrms digital isolators) is correct.

### ✅ SI8380P-IU + SI8621EC-B-IS Isolator Architecture

2 ICs for 10 channels (8 forward + 1 forward + 1 reverse) is efficient. The SI86XY naming convention analysis (Y = reverse channels) is correct — the project previously tried to use SI8622 (0F+2R) where SI8621 (1F+1R) was needed. Good catch, well-documented.

### ✅ TLV70450DBVR Isolated-Side LDO

+15V → 5V for MCP4922, isolator VDD2s, and Vref IC. 24V max input handles any PCN1 output overshoot. 150mA capacity vs ~12mA actual load — large margin. Thermal: 200mW in SOT-23-5 → ~46°C rise at 230°C/W θJA → safe.

### ✅ BSS138 USB Interlock Concept

Hardware-only interlock that firmware cannot bypass. Killing the boost converter EN kills the entire isolated power domain. No ±15V = no current through the patient. This is the correct architecture for a "charging means no stimulation" requirement.

### ✅ Susumu RG2012P-101-B-T5 Sense Resistor

100Ω, 0.1%, 25 PPM/°C, 0805 thin-film. Current accuracy dominated by this part: ±0.1% (5µA at 5mA). TCR contribution at ΔT=20°C: 0.05Ω (2.5µA). Total: ±7.5µA at 5mA = 0.15%. Excellent for medical stimulation.

### ✅ Samsung CL31B106KBHNNNE DC-Block (with caveats per Finding 4)

10µF/50V/X7R/1206. Electrically correct value (100mV droop at 5mA × 200µs). Voltage rating provides 3.8× derating over worst-case 13V compliance voltage. See Finding 4 for MLCC-specific discussion.

### ✅ Yageo SMD0603B001TF PPTC Fuse

10mA hold / 30mA trip. Well-sized for a 5mA working current. The 10mA hold current means the fuse stays cool during normal operation; the 30mA trip catches catastrophic fault currents. Correctly identified as the slow backup layer behind the fast LM339 comparator-based cutoff.

### ✅ Multi-Layer Safety Architecture

The defense-in-depth approach is textbook correct for medical devices:

| Layer | Speed | Mechanism |
|-------|-------|-----------|
| 1 — Firmware | ~µs | Parameter validation, ISR-level bounds |
| 2 — LM339 latch | <10µs | Analog comparator trips on overcurrent |
| 3 — PPTC fuse | ~ms | Self-resettable backup |
| 4 — DC-blocking cap | Passive | Prevents DC regardless of all above |
| 5 — Heartbeat watchdog | <50ms | Detects firmware crash, kills analog power |
| 6 — USB interlock | Hardware | Prevents stimulation during charging |
| 7 — Emergency stop | User action | Physical kill switch |

Each layer is independent. No single failure (firmware crash, MOSFET short, isolator failure) can deliver unsafe current to the patient. This is IEC 60601-1 thinking even if formal certification isn't pursued.

### ✅ Core Assignment (FreeRTOS)

Core 1 for stimulation ISR, Core 0 for BLE/session management. This prevents BLE stack operations (which can take 10+ ms) from creating timing jitter in the stimulation waveform. The stagger between channels (default 20ms) is maintained by the same hardware timer ISR on Core 1 — no inter-core synchronization needed for phase timing.

### ✅ Charge Density Safety Margin

At worst-case parameters (5mA × 500µs, 0.4cm² electrode): 2.5 µC/cm² = 12× below McCreery 30 µC/cm² limit. At clinical sweet spot (2mA × 200µs): 0.5 µC/cm² = 60× below limit. Generous margin that accounts for electrode area estimation uncertainty.

---

## Summary of Actions

| # | Finding | Severity | Action |
|---|---------|----------|--------|
| 1 | DRV8871 6.5V minimum VM | 🔴 CRITICAL | Replace with discrete MOSFETs (BSS84+BSS138) or DRV8837. Update AGENTS.md + DESIGN-REVIEW.md BOM. |
| 2 | Vref approach undecided | 🟡 SIGNIFICANT | Choose MCP1501-10 or MCP1501-20. Remove resistor-divider alternative from docs. |
| 3 | SPI MISO missing for impedance ADC | 🟡 SIGNIFICANT | Decide: add 2nd SI8621EC for proportional measurement, or use LM339 threshold-only. |
| 4 | MLCC vs Film cap contradiction | 🟡 SIGNIFICANT (docs) | Document MLCC rationale. Resolve contradiction between the two review documents. |
| 5 | Emergency stop switch type | 🟡 SIGNIFICANT | Replace tactile switch with larger/more accessible control. |
| 6 | PCN1 output voltage variation | 🟡 MINOR | Add Zener clamps. Characterize during bringup. |
| 7 | Power budget overestimate | ℹ️ INFO | Optionally correct 256mA to actual ~138mA. |
| 8 | Document inconsistencies | ℹ️ INFO | Reconcile packages, frequencies, part numbers across documents. |
| 9 | LM339 supply & pull-ups | ℹ️ MINOR | Verify pull-up resistors and power supply connection in schematic. |
| 10 | Bleed resistor value | ℹ️ MINOR | Consider reducing to 100kΩ for faster cap discharge. |
| 11 | Charge balance | ℹ️ INFO | Confirmed robust — three-layer approach is correct. |
| 12 | SPI through isolator | ℹ️ INFO | Confirmed — no timing issue at 1 MHz. |
| 13 | BSS138 gate pull-down | ℹ️ MINOR | Add 100kΩ gate pull-down. Verify TPS61023 EN pull-up. |

---

## Final Assessment

**This design is fundamentally sound.** The topology is correct, the safety architecture is thorough, and the component selection methodology — catch errors, document corrections, iterate — is exactly right for a medical device project.

The DRV8871 issue (Finding 1) is the only thing I'd call a showstopper, and it has clear solutions. Everything else is refinement, documentation cleanup, or minor optimization.

**You're closer to hardware bringup than you probably think.** Resolve the H-bridge, pick a Vref, decide on impedance sensing, and start the breadboard.

---

*Review completed 2026-04-24. Reviewed against AGENTS.md rev 2026-04-23, DESIGN-REVIEW.md rev 2026-04-23, CIRCUIT-DESIGN-REVIEW.md rev 2025-07-15.*
