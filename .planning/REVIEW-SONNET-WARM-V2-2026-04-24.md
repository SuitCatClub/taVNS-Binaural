# Second-Pass Design Review — OpenBinaural-taVNS
## Analog Signal Chain · Power Architecture · Safety Systems

**Reviewer:** Ren (claude-sonnet-4.5, warm instance)
**Date:** 2026-04-24
**Documents reviewed:** AGENTS.md, DESIGN-REVIEW.md (2025-07-15, claude-opus-4.6), CIRCUIT-DESIGN-REVIEW.md (2025-07-15, earlier cross-reference review)
**Scope:** Second-pass review after two prior reviews. Goal: find what was missed, challenge what was accepted.

---

## 1. Executive Summary

The foundational topology choices — V-to-I current source with H-bridge polarity switching, dual ±15V isolated rails, ADA4522-2ARZ error amp, galvanic isolation via digital isolators — are sound and correct. The first pass reviews identified the critical blockers (DRV8871 VM floor, AD8628 supply voltage, missing boost converter) accurately. However, the second review document (DESIGN-REVIEW.md) contains a significant internal contradiction: it validates the DRV8871 as correct in §1 while AGENTS.md already replaced it with the DRV8837 — and neither document fully addresses the compliance overvoltage problem the DRV8837 creates at the top end of the operating range. The DC-blocking capacitor selection has regressed from film (fail-safe) to MLCC (fail-short) without implementing the series-pair mitigation that was the sole justification for using MLCC at all. Both of these issues carry direct patient-safety implications. Everything else I found is either confirmatory or engineering-grade concerns that deserve attention but don't block the design.

---

## 2. Assessment of the 8 Known Issues

### Issue 1 — DRV8871 minimum VM = 6.5V
**Verdict: UNDERSTATED**

The root issue is correctly identified: the DRV8871 can't operate when VM < 6.5V, and in this V-to-I topology VM = op-amp output, which must go as low as 0.3V (0.5mA × 600Ω). Fatal.

But what was missed: **the DESIGN-REVIEW.md (the more recent, supposedly final review) never resolved this.** Section 1 opens with "The DRV8871DDAR specifically is correct" and then justifies it on the basis of its 45V/50V rating. The BOM lists DRV8871DDAR as U7 and U8. AGENTS.md independently corrected this to DRV8837DSGR, but the two documents are now in direct contradiction, and the contradicting document (DESIGN-REVIEW.md) is the longer, more authoritative-looking one.

Second problem with the proposed fix: **the DRV8837's 11V absolute maximum VM creates an overvoltage risk at the top end.** The op-amp can output up to ~13.5V driving a 5mA × (2.5kΩ + 100Ω) = 13.0V compliance scenario. The specified electrode impedance range goes up to 2.5kΩ. At 5mA into 2.2kΩ electrode: V_opamp = 5mA × (2200Ω + 100Ω) = 11.5V — already 0.5V above the DRV8837's absolute maximum. The DRV8837 doesn't just clip at 11V; it gets damaged.

This is only OK if the design accepts a de-rated compliance limit of ~2kΩ at 5mA (or alternatively, if firmware never allows 5mA into Z > 2kΩ — but that's a firmware-only protection on a safety-critical parameter). Neither limit is documented or architecturally enforced.

**Concrete recommendation:** Add a clamp at the op-amp output: a pair of Schottky diodes (e.g., BAT54) from op-amp output to a 10.5V zener (biased from +15V rail). This hardware-limits VM to 10.5V regardless of op-amp output, protecting the DRV8837 while still covering 99% of clinical scenarios (2mA clinical sweet spot has 5kΩ compliance with DRV8837). Document the 2.2kΩ max impedance at 5mA as a design limit, not an edge case.

---

### Issue 2 — No SPI MISO reverse channel
**Verdict: UNDERSTATED**

The known issue identifies the problem correctly, and the proposed fix (SI8622EC with 0F+2R reverse channels) was reasonable. But DESIGN-REVIEW.md went a different direction: 1× SI8380P-IU (8 forward) + 1× SI8621EC-B-IS (1F+1R). This provides exactly **one** reverse channel, assigned to the fault flag.

The MCP3201 ADC for impedance sensing — which is listed in the BOM — has its MISO signal with no return path across the isolation barrier. This is a functional blocker for Phase 5 impedance sensing, not just a future consideration. The ADC is on the BOM, the firmware architecture assumes impedance monitoring, the safety architecture depends on contact detection, and there is no wired path for the ADC to talk back.

One reverse channel is not enough. The design needs: fault flag (1 reverse) + MCP3201 MISO (1 reverse) = 2 reverse channels minimum. The correct fix is to add either a second SI8621EC-B-IS (+1R, +1F unused), or swap to SI8622EC-B-IS where both channels are reverse. The SI8380P-IU + SI8621EC combination would then be: SI8380P-IU (8F) + SI8621EC (1F: heartbeat, 1R: fault) + one additional 1R isolator for MISO. Or accept one more SI8621EC (1F spare, 1R MISO).

A subtlety that makes this worse: if DRV8837 is used (no IPROPI), the only contact-detection path IS the MCP3201 reading Rsense voltage during rest periods. Block MISO, and contact detection disappears. Without contact detection, the safety architecture explicitly states "Must be implemented before any human use." This makes the unrouted MISO path a pre-human-use blocker.

---

### Issue 3 — Vref strategy undecided
**Verdict: AGREE — resolved**

MCP1501-10E/SN (1.024V) with MCP4922 in 2× gain mode gives 20.48mA FSR, 5µA/LSB, 1000 usable codes over the 0–5mA clinical range. Math checks out. The CIRCUIT-DESIGN-REVIEW used MCP1501-20 (2.048V, 1× gain) which is mathematically identical. Both are correct. DESIGN-REVIEW.md has this in the BOM and explicitly documents the Vref calculation.

No concerns. AGENTS.md is consistent with DESIGN-REVIEW.md on this.

---

### Issue 4 — PPTC fuse inadequate as primary overcurrent
**Verdict: AGREE — resolution is good, one open detail**

The LM339DR quad comparator solution in DESIGN-REVIEW.md is better than the LM393 dual (CIRCUIT-DESIGN-REVIEW's suggestion) because it allocates channels explicitly: Ch1 = overcurrent A, Ch2 = overcurrent B, Ch3 = heartbeat watchdog, Ch4 = spare. That's clean architecture.

One open detail not flagged by prior reviews: the latch trip threshold is set at 6mA (0.6V across 100Ω Rsense). The PPTC fuse holds at 10mA. There is a 4mA gap (6mA–10mA) where the fast latch has fired AND the PPTC hasn't tripped. In this gap, the latch Q output must actually kill the stimulation. The documents describe the latch killing the H-bridge enable — verify that the LM339 open-collector output can drive whatever gate topology is used on the H-bridge supply kill path. If DRV8871 is used: drive its nSLEEP pin. If DRV8837: drive nSLEEP or IN1=IN2=0. If discrete MOSFET bridge: P-channel supply MOSFET gate. Each is different. This interface needs to be explicit in the schematic.

---

### Issue 5 — DC-blocking cap failure mode
**Verdict: UNDERSTATED — and the current BOM makes it worse**

The known issue correctly states MLCC fails short (bad) and film fails open (safe). The proposed mitigation is two MLCC in series per channel. Two series MLCC: if one fails short, the other still blocks. That's a valid engineering compromise.

**DESIGN-REVIEW.md did not implement this mitigation.** The BOM specifies a single Samsung CL31B106KBHNNNE (10µF 50V X7R MLCC) per channel, noting "Consider 2× 10µF in parallel" — that's parallel, not series. Parallel MLCC still fails short simultaneously; it does not restore the safety property.

CIRCUIT-DESIGN-REVIEW.md was explicit: "Film capacitors are CORRECT and mandatory here. Never ceramic (voltage coefficient, piezoelectric effects at low signal levels)... metallized polyester film type (MKT) is ideal for DC-blocking in this application." This was the first review's unambiguous conclusion. The second review reversed this without acknowledging the safety regression.

The MLCC choice saves board space and cost, which are real engineering benefits. But the safety tradeoff has to be consciously documented and the series-pair mitigation actually implemented if MLCC is chosen. Right now the BOM has the space/cost of MLCC without the safety of either film or the series-pair mitigation.

This is compounded by a new finding I detail below: the DC-blocking cap failure mode matters more than it appears because of the watchdog response gap.

---

### Issue 6 — Emergency stop button too small
**Verdict: AGREE — low priority**

Correct assessment. Both reviews agree this is a human-factors concern for the final product but not a blocker for the breadboard phase. The through-hole tactile switch in the BOM is fine as a placeholder.

---

### Issue 7 — Type BF / 1 MOPP compliance language
**Verdict: AGREE — important precision**

The isolation chain (PCN1: 1500VDC; SI8380P-IU: 2500Vrms; SI8621EC: 3750Vrms) targets 1 MOPP. The documents correctly acknowledge this is "targeting compliance" not "achieving compliance" — actual compliance requires testing, not just component selection. The PCB layout note about isolation slots (6mm routed gap) and split GND planes is the right path forward.

One addition: creepage and clearance requirements on the PCB must cover the DC-blocking capacitor footprint too. The DC-block cap sits in the patient domain but its pads are close to the Rsense pads (which are also in the patient domain). These are same-domain so it's not an isolation concern, but the electrode connector is ultimately at the board edge. Verify minimum 8mm creepage from live patient conductors to the PCB edge/mounting hardware per IEC 60601-1 Table 4.

---

### Issue 8 — Loop stability 89° phase margin overstated
**Verdict: AGREE — but overstating the concern somewhat**

The hand analysis at face value: 89° is the result of a near-perfect pole-zero cancellation at the loop crossover. That's correct math for the model used. The claim that realistic margin is 50–70° is reasonable — higher-order poles from the op-amp's internal gain stages, the H-bridge gate delay (~10ns propagation in DRV8837), PCB trace inductance, and the Randles model's constant-phase element (not a simple RC) all add phase lag beyond what the model captures.

That said, 50–70° is still a very comfortable margin. The ADA4522-2ARZ is unity-gain stable and specifically characterized into capacitive loads. A real pathological stability concern would require phase margin below 30°. The "concern" here is about the documented number being wrong, not about the circuit being unstable.

One thing the prior analysis does correctly handle: the 10nF capacitor across Rsense (already recommended) establishes a second zero at 159kHz that adds phase at high frequency and provides additional margin. This wasn't in the original stability argument but strengthens the conclusion.

The ADA4522's chopper clock (~4kHz) is not modeled and creates a small periodic artifact, but at 1–5µV amplitude through 100Ω sense, the resulting current artifact is < 50nA — well below physiological significance. Not a stability concern, not a safety concern.

---

## 3. New Findings

---

### New Finding 1 — DESIGN-REVIEW.md validates DRV8871 while AGENTS.md replaced it with DRV8837

**Severity: Critical**

**What's wrong:** DESIGN-REVIEW.md Section 1 explicitly states "The DRV8871DDAR specifically is correct" and lists it in the BOM as U7 and U8. AGENTS.md's corrected component decisions table says DRV8871 → DRV8837DSGR with the reason being the 6.5V minimum VM incompatibility. These two documents are in direct contradiction.

**Why it matters:** This isn't just a document inconsistency — it's a question of which document someone building this device will follow. The more detailed, authoritative-looking DESIGN-REVIEW.md endorses the wrong part. If a builder follows DESIGN-REVIEW.md (as they plausibly would, since it's the comprehensive BOM), they get DRV8871, which makes the circuit non-functional at low stimulation currents.

**Proposed fix:** DESIGN-REVIEW.md's §1 "Topology Verdict" needs to be corrected. The IPROPI current sense feature of DRV8871 that the review praises is irrelevant if the device can't operate below 6.5V VM. Consider flagging this explicitly at the top of that section rather than deep in the text, and update the BOM to DRV8837DSGR consistently.

---

### New Finding 2 — DC-blocking cap regression: film → MLCC without implementing the series-pair mitigation

**Severity: Critical (patient safety regression)**

**What's wrong:** CIRCUIT-DESIGN-REVIEW.md explicitly required film caps as "mandatory" for DC-blocking, stating MLCC is inappropriate due to short-circuit failure mode. The known Issue #5 proposes "two MLCC in series" as the only acceptable MLCC mitigation. DESIGN-REVIEW.md replaced film caps with a single X7R MLCC (Samsung CL31B106KBHNNNE) and notes "consider 2× in parallel" — parallel MLCC fails short together and provides no improvement over a single MLCC. The safety regression is complete: MLCC is now in the BOM, and the series-pair fix is not implemented.

**Why it matters:** The DC-blocking cap is a hardware-enforced patient safety element. Its function is to prevent DC current from flowing into the patient if the current source malfunctions and outputs a DC offset. An MLCC that fails short converts this hardware protection into no protection. The original CIRCUIT-DESIGN-REVIEW document understood this clearly.

There is also a time-domain argument that makes this worse than it appears (see New Finding 4 below): the DC-blocking cap is the last line of defense during a watchdog response gap of up to 50ms, during which a crashed MCU could be delivering continuous DC at 5mA. A film cap limits the charge in this scenario; a short-circuited MLCC does not.

**Proposed fix (two options, either is acceptable):**
1. **Film cap (preferred):** Revert to ECQ-E1475KF or equivalent 10µF/100V film. Larger and more expensive but provides genuine fail-safe behavior. Volume is 18×10×15.5mm — large but manageable on a PCB where safety is paramount.
2. **MLCC with series pair:** Use 2× Samsung CL31B106KBHNNNE in series per channel = equivalent 5µF, 100V effective. One fails short: the other still blocks. Add one more in parallel to restore to 7.5µF effective (3× total: 2 series + 1 parallel with one of the series pair). Larger BOM, but workable in 1206.

The decision between these options should be explicit and documented, not accidentally resolved by a BOM update that didn't acknowledge the tradeoff.

---

### New Finding 3 — DRV8837 absolute maximum VM exceeded at top of specified operating range

**Severity: Major**

**What's wrong:** DRV8837DSGR absolute maximum VM = 11V. The V-to-I compliance requirement at 5mA into the specified maximum electrode impedance of 2.5kΩ:

```
V_opamp = I × (Z_electrode + R_sense)
        = 5mA × (2500Ω + 100Ω)
        = 13.0V
```

This is 2V above the DRV8837's absolute maximum. The spec sheet's abs max isn't a soft limit with derating — it's where the device gets damaged.

**Why it matters:** A user with high-impedance electrodes (old electrodes, dry skin, poor contact) running at 5mA would damage the H-bridge IC. The circuit doesn't naturally limit this — the op-amp faithfully tries to maintain 5mA through whatever Z it sees, driving its output to 13V, overvoltaging the DRV8837. The current source topology is specifically designed to compensate for load variation, which means it actively drives into the damage condition.

**Proposed fixes:**
1. **Op-amp output clamp (recommended):** Schottky + zener clamp at op-amp output, limiting VM to ~10V. This is transparent to normal operation (10V compliance = 2kΩ at 5mA, still 5kΩ at 2mA). Documented compliance limit: 2kΩ at 5mA. Firmware can enforce this by measuring impedance and refusing to go above 5mA into Z > 2kΩ.
2. **Firmware current limiting:** Check impedance before and during stimulation; reduce current setpoint when Z > 2kΩ to keep V_opamp below 10.5V. FIRMWARE ONLY — not acceptable as sole protection for a safety device.
3. **Accept the DRV8837's 11V cap as a de-rated compliance limit:** Document that max operating point is 5mA / 2kΩ or 2mA / 5kΩ, never combination that requires >10V compliance. This shrinks the clinical envelope but may still cover 95% of real-world use. Does NOT protect the DRV8837 from being overvoltaged by a patient with unusually high impedance.

For a patient-contact device, option 1 is the only acceptable hardware protection. Option 2 alone is insufficient.

---

### New Finding 4 — Watchdog response gap allows 250µC DC injection after MCU crash

**Severity: Major**

**What's wrong:** The heartbeat watchdog uses an RC detector with τ ≈ 50ms. If the ESP32 crashes mid-pulse (the stimulation ISR is killed), the H-bridge is stuck in one polarity. The LM339 overcurrent comparator is set to trip at 6mA — it does NOT trip because the continuous DC current (5mA setpoint) is below the threshold. The watchdog needs ~3τ ≈ 150ms to be 95% confident the heartbeat is gone. In practice, the window is likely 50–100ms.

Charge delivered during this gap:
```
Q = I × t = 5mA × 100ms = 500µC
```

The per-phase charge limit from McCreery (1990) is 25µC at this design's operating point. 500µC is 20× that limit. More relevantly, this is continuous DC, not biphasic — even small amounts of DC current cause electrochemical damage at the skin-electrode interface.

The DC-blocking capacitor is the fail-safe for this scenario. For a 10µF film cap:
```
V_cap(t) = (I/C) × t = (5mA / 10µF) × 100ms = 50V
```
The film cap would reach its 100V rating before the watchdog trips, then fail open — stopping current flow. This is the correct fail-safe behavior. An MLCC that has already failed short (see New Finding 2) provides none of this protection.

**Why it matters:** This finding directly connects New Findings 2 and 4. The DC-blocking cap's film vs. MLCC question isn't merely about normal operation — it's about what happens in the specific failure mode of MCU crash during stimulation. The film cap is a hardware fuse for exactly this scenario. The MLCC is not.

**Proposed fix:**
1. **Immediate:** Revert to film caps (or implement the 3× MLCC series/parallel approach from New Finding 2).
2. **Additionally:** Reduce watchdog RC time constant. Current τ = 50ms with 100Hz heartbeat. Alternative: τ = 5ms with 1kHz heartbeat. At τ = 5ms, watchdog response in 15ms, Q_injected = 75µC (3× limit but orders of magnitude better than 500µC). This requires faster heartbeat PWM frequency — still trivial for ESP32 Core 1.
3. **Belt-and-suspenders:** Add a dedicated hardware timer on the isolated side (simple RC + comparator) that cuts isolated power if stimulation has been on continuously for >500ms with no inter-phase gap (true DC scenario). This is independent of the heartbeat watchdog.

---

### New Finding 5 — MCP3201 MISO has no return path; contact detection is architecturally broken

**Severity: Major**

**What's wrong:** The isolation barrier has 1 reverse channel (SI8621EC, used for fault flag). The MCP3201 ADC's SPI MISO line is on the isolated side with no return path to the ESP32. The ADC is on the BOM and in the architecture, but electrically cannot communicate.

This compounds with a second problem: AGENTS.md safety architecture says "Electrode contact detection — DRV8871 IPROPI + LM339 comparator. Abort if Z > 10kΩ or Z < 100Ω." If DRV8837 is used (no IPROPI), the sole contact detection path is the MCP3201 measuring voltage across Rsense during the REST period. That path is also broken.

**The result:** Contact detection — documented as mandatory before any human use — is not achievable with the current isolation architecture regardless of which H-bridge is chosen.

**Why it matters:** "Must be implemented before any human use" is a stated safety requirement. The current schematic architecture cannot implement it.

**Proposed fix:** Add one more reverse-channel isolator. The most compact solution: replace SI8621EC (1F+1R) with a second SI8621EC and reassign: SI8621EC #1 → heartbeat (F) + fault flag (R). SI8621EC #2 → spare (F, leave unconnected) + MCP3201 MISO (R). Total: SI8380P-IU (8F) + 2× SI8621EC-B-IS (1F+1R each) = 10F + 2R. All 10 forward signals plus both reverse signals covered. BOM adds $3 and one SOIC-8 footprint.

---

### New Finding 6 — TLV70450DBVR thermal dissipation is marginal at actual load

**Severity: Major**

**What's wrong:** The TLV70450DBVR (SOT-23-5 LDO, +15V → +5V) powers the MCP4922 DAC, SI8380P-IU VDD2, SI8621EC VDD2, and MCP3201 ADC on the isolated side. The DESIGN-REVIEW.md notes "Iout max = 150mA (DAC draws ~1mA — massive margin)" but this ignores the digital isolator isolated-side supply.

Realistic load:
- MCP4922 DAC: ~2mA
- SI8380P-IU (VDD2): 8 push-pull output channels at up to 5mA each peak, ~20mA average
- SI8621EC (VDD2): ~2mA
- MCP3201 ADC: ~1mA
- Total: ~25mA typical, ~40mA at peak SPI activity

Power dissipation:
```
P = (Vin - Vout) × I = (15V - 5V) × 40mA = 400mW
```

TLV70450DBVR in SOT-23-5 has θJA = 166°C/W (per TI datasheet, JEDEC standard board). Maximum junction temperature = 125°C. At 25°C ambient:
```
ΔT = P × θJA = 400mW × 166°C/W = 66°C
T_junction = 25°C + 66°C = 91°C
```

91°C junction temperature is within limits on paper, but:
1. The device will be in a small enclosure (elevated ambient, likely 35–40°C)
2. At 40°C ambient: T_junction = 106°C — approaching the 125°C max
3. The SI8380P-IU current spec is "per channel static" — during rapid SPI transactions, peak ICC2 can be higher
4. No thermal pad in SOT-23-5; relies entirely on PCB copper spreading

**Proposed fix:** Use TLV70450DDCT (SOT-223 package, same electrical spec, θJA ≈ 28°C/W). At 400mW in SOT-223: ΔT = 11.2°C → T_junction = 36°C. Completely comfortable. BOM footprint change only. Alternatively, if PCB space is tight, place the TLV70450 with generous copper pours on all pads — 2oz copper with 50mm² area can reduce θJA to ~80°C/W (T_junction ≈ 57°C at 40°C ambient).

---

### New Finding 7 — ESP32 ADC unreliable for battery safety lockout

**Severity: Minor (but should be resolved before human use)**

**What's wrong:** AGENTS.md specifies battery voltage monitoring via a resistor divider to the ESP32-S3's internal ADC, with lockout thresholds at 3.2V (refuse start) and 3.0V (ramp down mid-session). The ESP32-S3's ADC has well-documented nonlinearity and approximately ±6% uncalibrated error on the 0–3.3V range. 

The lockout window is 200mV (3.0V to 3.2V) on a 3.0–4.2V LiPo range. The ADC error at this point (with a divider scaling 4.2V → 3.3V ADC input) is:

```
ADC error ±6% × 3.3V = ±198mV
```

The full 200mV lockout window fits within one ADC error band. In the worst case (ADC reads high), the device could allow stimulation at VBAT = 2.8V, where the ±15V rails are losing regulation and current output is unpredictable.

**Proposed fix:** The simplest reliable solution doesn't use the ADC at all. Set the TPS61023 UVLO threshold to cut out at VBAT ≤ 3.0V by selecting the UVLO resistor divider correctly. TPS61023 has a hardware UVLO input (EN pin threshold ~0.5V); a voltage divider from VBAT with UVLO trip at 3.0V:

```
R1/(R1+R2) = 0.5V / 3.0V → ratio 1:5 → e.g., 10kΩ + 51kΩ
```

This hardware UVLO feeds the same EN line as the USB charging interlock (BSS138), so both disable the boost converter simultaneously. Firmware battery monitoring via ADC can still run for UI/BLE reporting with all its inaccuracy — it just doesn't control the safety cutoff.

---

### New Finding 8 — LM339 comparator reference voltage stability on isolated side

**Severity: Minor**

**What's wrong:** The LM339DR overcurrent comparator reference voltage (0.6V, representing 6mA × 100Ω) is derived from a resistor divider off the 5V_ISO rail. The 5V_ISO comes from the TLV70450DBVR LDO, which gets its input from the +15V isolated rail, which comes from the PCN1-S5-D15-M-TR — an unregulated isolated DC-DC converter.

The PCN1-S5-D15-M-TR is unregulated. Its output voltage varies with load. Under the power budget's ~300mA total load, the PCN1's output regulation is specified as ±10% (CUI datasheet). This means +15V could be 13.5V–16.5V under load variation. The TLV70450 is a linear LDO from +15V → 5V; its output regulation is tight (±2%). So 5V_ISO is well-regulated.

The divider off 5V_ISO: V_ref = 5V × [R2/(R1+R2)]. If R1 and R2 are 1% tolerance metal film, V_ref = 0.600V ±1% = 0.594–0.606V, equivalent to trip point 5.94mA–6.06mA. That's adequate.

However, **the LM339's input offset voltage (Vio ≤ 5mV) adds ±50µA uncertainty at the trip point.** At 5.94mA – 0.05mA = 5.89mA, the comparator might trigger during a legitimate 5mA + tolerance pulse. With 6mA nominal trip and 5mA clinical maximum, the 1mA headband is only 16.7% margin. Combined with Rsense tolerance (±0.1%) and LM339 offset (±5mV / 100Ω = ±50µA), the effective margin is closer to 12%.

**Proposed fix:** This is an inherent architectural tradeoff, not something that needs fixing per se. But document it: the overcurrent trip point should be confirmed in schematic with a note about the combined tolerance stack. Consider increasing the headband to 7mA trip (0.7V reference) and adjusting the PPTC hold current to 15mA — this gives 40% headband from 5mA setpoint to comparator trip, which is more comfortable with the combined error budget. The charge density calculation still gives a 4× safety margin to McCreery's limit even at 7mA × 200µs.

---

## 4. What I Checked and Found OK

- **ADA4522-2ARZ selection:** Correct part. ±27.5V supply, zero-drift, 2.7MHz GBW, 1.7V/µs slew. Slew rate analysis for 200µs pulse confirmed adequate (7% rise time at worst case). Vos ≤10µV max → <0.1µA DC offset through 100Ω Rsense → negligible charge imbalance.

- **PCN1-S5-D15-M-TR isolation rating:** 1500VDC isolation meets 1 MOPP for battery-powered Type BF. SI8380P-IU (2500Vrms) and SI8621EC (3750Vrms) exceed this — the DC-DC is the weak link and it's already at the requirement floor. Correct analysis.

- **TPS61023 boost converter:** Correct selection for LiPo → 5V. 4MHz switching frequency at the boost output keeps ripple low for PCN1 input. Power budget math (330mA total from 3.7V LiPo) is realistic. 6-hour estimated session life is plausible.

- **ME6211C33M5G LDO:** Correct. 100mV dropout avoids the AMS1117 problem across the full LiPo range. Connected directly to LiPo (not via boost) is more efficient — valid. No concerns.

- **MCP4922-E/SL DAC placement on isolated side:** Correct decision. Analog voltage must not cross the isolation barrier. Digital SPI crossing the barrier via digital isolators is the right architecture.

- **Susumu RG2012P-101-B-T5 sense resistor:** Correct. 100Ω ±0.1% ±25PPM/°C in 0805. At ΔT=20°C, drift is 0.05Ω → ±2.5µA error at 5mA. Well within the design's tolerance.

- **Charge density safety margin:** At 5mA × 500µs = 2.5µC/cm² per phase (assuming 1cm² electrode). McCreery limit 30µC/cm². 12× margin confirmed. Even the tightest clinical preset (5mA × 500µs) is comfortably safe.

- **Clinical preset parameter table (AGENTS.md):** Anti-inflammatory at 25Hz (not 10Hz), RESET-AF unilateral left ear, Dysautonomia as 0.5mA ramp — all corrections noted. Stagger experimental label is appropriate.

- **BSS138 USB charging interlock:** Correct topology. VBUS → BSS138 gate → TPS61023 EN pulled low → boost off → ±15V off. Hardware-only path, firmware cannot override. No concerns.

- **LM339DR quad comparator BOM position (over LM393 dual):** Correct upgrade. Three comparators used with one spare is clean and gives the heartbeat watchdog its own dedicated comparator channel.

- **Core/thread assignment (FreeRTOS Core 1 = stimulation ISR, Core 0 = BLE):** Architecturally sound. The critical concern (BLE interrupting stimulation ISR) is addressed by core affinity, not just priority. Correct approach.

---

## 5. Summary Table — Issues Requiring Action

| Priority | Finding | Action Needed |
|----------|---------|---------------|
| 🔴 Critical | DRV8871 in DESIGN-REVIEW BOM contradicts AGENTS.md fix | Update DESIGN-REVIEW §1 and BOM to DRV8837DSGR |
| 🔴 Critical | MLCC DC-block cap without series pair — safety regression | Revert to film caps OR implement 3× MLCC (2 series + 1 parallel) |
| 🟠 Major | DRV8837 overvoltage above 2.2kΩ at 5mA | Add op-amp output clamp (BAT54 + 10V zener) or document/enforce compliance de-rating |
| 🟠 Major | 50ms watchdog gap → 250µC DC injection on MCU crash | Reduce watchdog τ to 5ms + revert film caps for defense-in-depth |
| 🟠 Major | MCP3201 MISO unrouted; contact detection broken | Add second SI8621EC-B-IS reverse channel for MISO |
| 🟠 Major | TLV70450DBVR thermal marginal in SOT-23-5 | Upgrade to SOT-223 footprint or add copper pours |
| 🟡 Minor | ESP32 ADC unreliable for VBAT lockout | Hardware UVLO via TPS61023 EN divider |
| 🟡 Minor | LM339 trip point tolerance stack | Increase trip to 7mA; document error budget |

---

*Review completed 2026-04-24.*

*— Ren*
