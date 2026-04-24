# OpenBinaural-taVNS — Electrical Design Review (Second Pass)

**Date:** 2025-07-24
**Reviewer:** Mara (AI, electrical engineering focus)
**Scope:** Full signal chain, power architecture, safety, isolation — second-pass review after two prior reviews
**Documents reviewed:** AGENTS.md, DESIGN-REVIEW.md (~38KB), CIRCUIT-DESIGN-REVIEW.md (~30KB)

---

## Approach

I read all three documents end-to-end before writing anything. I'm treating the DESIGN-REVIEW.md as the canonical design state and CIRCUIT-DESIGN-REVIEW.md as the earlier review that first identified several issues. Where they disagree, I call it out. Where they agree, I mostly move on. My goal is fresh eyes, not confirmation.

---

## Part 1: Assessment of the 8 Known Issues

### 1. DRV8871 6.5V Minimum VM — AGREE, Critical Blocker

Both prior reviewers are right. In the V-to-I topology where VM = op-amp output, the DRV8871 is fundamentally incompatible. At 0.5mA into 600Ω, the op-amp output is 0.3V. The DRV8871 needs 6.5V minimum to operate its internal charge pump and gate drivers. It simply won't turn on.

**However, the project's own documents contradict each other on this.** DESIGN-REVIEW.md §1 declares the DRV8871 "APPROVED — Correct approach" and the final BOM (§10, line U7/U8) lists two DRV8871DDARs. Meanwhile, CIRCUIT-DESIGN-REVIEW.md §2 correctly identifies it as a showstopper and recommends discrete MOSFETs or DRV8837. AGENTS.md lists DRV8871 as the chosen part.

This contradiction is itself a risk. Someone coming into this project fresh would read DESIGN-REVIEW.md §1, see "APPROVED," and proceed with the DRV8871. The BOM reinforces it. **The DRV8871 needs to be removed from the BOM and §1 needs a correction note.** The "approved" verdict in §1 is analyzing the topology (V-to-I + H-bridge), which IS correct — but it then specifically names DRV8871 as the right part, which it is not.

**My take on alternatives:**
- **DRV8837:** Pragmatic choice. 0–11V VM. Built-in dead-time and shoot-through protection. Compliance limited to ~10V, so Z_max at 5mA drops to ~2kΩ. At clinical 2mA it's 5kΩ — adequate. I'd start here for breadboard.
- **Discrete MOSFETs:** See Issue #NEW-1 below — the BSS84 high-side approach has its own showstopper that nobody caught.

### 2. No SPI MISO Reverse Channel — AGREE, Understated

The DESIGN-REVIEW.md final isolator allocation is SI8380P-IU (8 forward) + SI8621EC-B-IS (1 forward + 1 reverse). That's 9 forward + 1 reverse. The reverse channel goes to the fault flag.

**MISO has no path back.** If the MCP3201 impedance ADC is placed on the isolated side (it should be), its MISO data cannot cross the barrier. This isn't just a "nice to have" — impedance monitoring is a safety feature (electrode contact check before stimulation, abort on Z > 10kΩ). Without MISO, impedance data stays trapped on the isolated side.

Options:
- Add a second SI8621EC-B-IS (1F+1R) — gives 1 more reverse channel for MISO. Adds ~$3 and one SOIC-8.
- Use SI8622EC-B-IS (0F+2R) instead of SI8621EC — gets 2 reverse channels (fault + MISO) but loses the forward heartbeat channel. Would need to steal one SI8380P channel for heartbeat. This is cleaner: 8 forward signals on SI8380P (7 control + heartbeat), 2 reverse on SI8622EC (fault + MISO). **10 total channels, 2 ICs, no change to BOM count.**

I'd go with the SI8622EC swap. Verify the SI86XY naming is correct (Y = number of reverse channels → SI8622 = 2 reverse, 0 forward). DESIGN-REVIEW.md §5 actually notes the naming convention: "SI8622 = 0F+2R" — so this swap would work.

### 3. Vref Strategy Undecided — AGREE, More Serious Than Stated

This is worse than "undecided." The two documents actively disagree:
- CIRCUIT-DESIGN-REVIEW.md recommends **MCP1501-20E/SN** (2.048V precision reference)
- DESIGN-REVIEW.md §11A recommends a **resistor divider** from TLV70450's 5V output

And **neither is in the final BOM** (DESIGN-REVIEW.md §10). The MCP4922 has no internal reference. With no Vref, it outputs nothing. This is a functional gap, not just a design choice to make later.

**My recommendation:** Use a proper voltage reference, not a resistor divider. This is a medical current source — the Vref directly determines current accuracy. A resistor divider from the TLV70450 has:
- LDO output noise (30µVrms) passed through
- Temperature drift dependent on resistor matching (two 0.1% resistors still give ~50ppm combined TCR)
- Load regulation of the divider affected by MCP4922's Vref input current (~1µA, negligible, but still)

An MCP1501 at $1.50 gives 10ppm/°C and 0.08% initial accuracy. For a device controlling current through human tissue, $1.50 for a proper reference is the right call. Use the MCP1501-10 (1.024V) with MCP4922 in 2× gain mode → full-scale output = 2.048V. Then I_max = 2.048V/100Ω = 20.48mA, resolution = 5µA/LSB. Clinical range (0.5–5mA) uses DAC codes 100–1000 out of 4096 — plenty of resolution.

**Action: Add MCP1501-10E/SN to the BOM. Pick a Vref value and commit.**

### 4. PPTC Fuse as Primary Overcurrent — AGREE

Correctly identified. PPTC is thermal — trip time is milliseconds to seconds depending on fault current. The LM339 comparator + latch at <10µs is the right primary protection. PPTC stays as a passive backup layer for catastrophic faults (e.g., latch failure, wiring short).

One nuance: the LM339 has open-collector outputs with ~1.3µs response time. With a 4.7kΩ pull-up and ~10pF parasitic capacitance, the rising edge (fault detected → output HIGH) takes τ = 4.7kΩ × 10pF = 47ns — fast. But the latch and downstream MOSFET switch add their own delays. Total fault-to-cutoff should be verified on the bench — target <10µs is achievable but not guaranteed without measurement.

### 5. DC-Blocking Cap: MLCC X7R vs Film — PARTIALLY DISAGREE

Both reviewers flagged this. CIRCUIT-DESIGN-REVIEW.md says "Film capacitors are CORRECT and mandatory. Never ceramic." DESIGN-REVIEW.md says X7R is acceptable and analyzes the voltage coefficient concern correctly.

**I side with DESIGN-REVIEW.md on the voltage coefficient argument.** At <100mV DC bias, X7R derating is negligible (<5%). The piezoelectric concern is also negligible at these voltages. The temperature coefficient is ±15% worst-case — fine for DC blocking.

**But both reviewers missed the failure mode concern.** This is new:

MLCC capacitors fail **short-circuit** — cracked ceramic (from mechanical stress, thermal shock, or board flex) creates a conductive path through the dielectric. For a DC-blocking safety capacitor, a short-circuit failure defeats its entire purpose. The DC-blocking cap is the last line of defense ensuring zero net DC charge to the patient. If it shorts, the patient sees any DC offset from the V-to-I converter.

Film capacitors fail **open-circuit** — the metallized film vaporizes around the fault, self-healing. This is the fail-safe failure mode for DC blocking.

**Mitigation options:**
1. **Two MLCC in series** per channel. Either one can fail short and the other still blocks DC. Cost: +$0.02/channel. Space: +one 1206 footprint. This is the practical answer.
2. **Switch to film.** A Panasonic ECH-U1C103JX5 (10nF, 16V, 0805 film) — wait, can't get 10µF in film at 0805. Smallest 10µF film caps are still ~10mm+ body size.
3. **Accept the risk.** MLCC failure rates in low-stress applications (no board flex, no reflow cracking) are very low. With the LM339 comparator latch as overcurrent protection, a shorted DC-block cap would still be caught if the DC offset exceeds the 6mA trip threshold.

**My recommendation: Two MLCC in series per channel. Simple, cheap, addresses the failure mode.** This also reduces the effective capacitance to 5µF per channel (two 10µF in series), which is still adequate per the droop analysis (V_droop = 1µC/5µF = 0.2V at 5mA/200µs).

### 6. Emergency Stop Button Too Small — AGREE, Low Priority

Valid human factors concern. For a bedside device used while falling asleep, a tiny tactile button is hard to find in the dark. But for prototype/breadboard phase, it's fine. For the final PCB, consider a 12mm illuminated pushbutton (Cherry MX, E-Switch, etc.) with a red LED backlight. Cost: ~$2. This is a Phase 8 (PCB design) concern, not blocking.

### 7. Type BF / 1 MOPP Compliance Language — AGREE

The design documents are inconsistent. Some sections use "design target" (correct), others make direct compliance claims like "1500VDC meets the 1 MOPP requirement for battery-powered Type BF. No issue." (DESIGN-REVIEW.md §9, line 544).

IEC 60601-1 Type BF compliance requires far more than component ratings:
- Creepage and clearance verification (PCB-dependent)
- Patient leakage current measurement (bench test)
- Dielectric strength testing (hipot test)
- Risk analysis documentation
- Biocompatibility of patient-contact materials

Using component ratings that *target* 1 MOPP is good engineering. Claiming the design *meets* 1 MOPP before any of these tests is premature. The language should be "designed to meet" or "targeting" throughout.

### 8. Loop Stability 89° Phase Margin — AGREE It's Overstated, But Conclusion Is Correct

The stability analysis in DESIGN-REVIEW.md §8 correctly identifies the fundamental mechanism: Rsense in series with the electrode creates a zero that partially cancels the electrode capacitance pole. This IS the key insight and it's correct.

But the specific "89° phase margin" number is derived from a simplified single-pole/single-zero hand analysis that:
- Treats the electrode as a simple parallel RC (real skin impedance is a Randles circuit with Warburg element)
- Ignores higher-order op-amp poles
- Ignores PCB parasitic capacitance on traces
- Ignores the H-bridge's transient behavior during the feedback settling
- Ignores the 10nF feedback cap across Rsense (which adds another pole)

A realistic estimate is probably **50–70° phase margin** — still very stable, well above the 45° minimum. The topology IS inherently stable for a unity-gain-stable op-amp into a predominantly resistive load. But claiming 89° with prose-level analysis gives false confidence.

Also: the chopper artifact note at DESIGN-REVIEW.md §3 (line 174) contains an error. It says the 10nF feedback cap corner at 159kHz is "well below chopper." The chopper is at ~4kHz. 159kHz is well ABOVE 4kHz. At 4kHz, the 10nF cap has ~4kΩ impedance — essentially invisible against 100Ω Rsense. **The 10nF cap does not attenuate chopper artifacts.** But this doesn't matter practically — chopper artifacts are µV-level and translate to nA-level current noise, far below therapeutic relevance.

---

## Part 2: New Findings

These are issues that, as far as I can tell, neither prior review identified.

### 🔴 NEW-1: Discrete BSS84 H-Bridge — High-Side Gate Drive Failure

CIRCUIT-DESIGN-REVIEW.md recommends discrete MOSFETs (BSS84 P-channel high-side + BSS138 N-channel low-side) as the DRV8871 replacement. This is listed as the "RECOMMENDED" option for maximum compliance headroom.

**This doesn't work with logic-level gate drive.**

The BSS84 P-channel MOSFET turns OFF when Vgs > -0.8V (Vgs threshold is -0.8V to -1.2V). Its source connects to the op-amp output (the "high rail" of the H-bridge). The gate is driven by the SI8380P-IU isolator output: 0V or 5V_ISO (≈5V).

At low op-amp output (e.g., 3V for low-current stimulation):
- Gate = 5V (OFF command) → Vgs = 5V - 3V = +2V → BSS84 is OFF ✅
- Gate = 0V (ON command) → Vgs = 0V - 3V = -3V → BSS84 is ON ✅

At high op-amp output (e.g., 12V for 5mA into 2.3kΩ):
- Gate = 0V (ON command) → Vgs = 0V - 12V = -12V → BSS84 is ON ✅
- Gate = 5V (OFF command) → Vgs = 5V - 12V = **-7V → BSS84 is still ON** ❌

**The P-channel MOSFET cannot be turned off by a 5V logic signal when the op-amp output exceeds ~6V.** This is a well-known problem with P-channel high-side switches driven from logic levels. The gate voltage must be within ~1V of the source to turn off, which requires level-shifting.

**This makes the "recommended" discrete MOSFET alternative unworkable without additional gate driver circuitry** (e.g., a high-side gate driver IC or a level-shifting circuit using NPN transistors). This adds enough complexity that the DRV8837 becomes the clearly simpler path.

**Impact:** The DRV8837 (0–11V VM, built-in gate drivers and dead-time) should be promoted from "budget/breadboard alternative" to **recommended solution**. The 11V compliance limit reduces Z_max at 5mA from 2.5kΩ to ~2kΩ — but at the clinical sweet spot of 2mA, Z_max is still ~5kΩ, well above typical electrode impedance.

If full ±15V compliance headroom is truly needed, use an all-N-channel H-bridge with bootstrap gate driver (e.g., IR2104 + 4× BSS138), or a fully integrated high-voltage H-bridge with low-voltage compatibility. But this is significantly more complex.

### 🟡 NEW-2: Unbalanced Rail Loading on PCN1-S5-D15-M-TR

The PCN1-S5-D15-M-TR is an unregulated dual-output isolated DC-DC. Its two output rails (+15V and -15V) share a center-tapped transformer secondary. Cross-regulation depends on balanced loading — heavily asymmetric loads cause rail voltages to drift apart.

**Current loading estimate:**

| Rail | Load | Current |
|------|------|---------|
| +15V | ADA4522-2 quiescent (sourcing side) | ~2mA |
| +15V | Stimulation current (2 channels) | 0–10mA |
| +15V | TLV70450 LDO input (5V_ISO for DAC, isolators, comparator) | ~20mA |
| +15V | LM339DR (Vcc from +15V or 5V_ISO?) | ~1mA |
| **+15V total** | | **~33mA typical** |
| -15V | ADA4522-2 quiescent (sinking side) | ~2mA |
| -15V | Nothing else — op-amp output is always positive | ~0mA |
| **-15V total** | | **~2mA** |

**That's a ~16:1 load imbalance.** For an unregulated converter, this is severe.

Typical cross-regulation for this class of converter is ±5–10% under balanced conditions, degrading significantly with imbalance. The +15V rail could sag to +12V while -15V rises to -18V (or worse). Actual values depend on the transformer coupling coefficient and winding resistance — requires bench measurement.

**Consequences:**
- +15V sag reduces op-amp output swing → reduced compliance voltage → lower Z_max
- -15V overshoot wastes headroom but doesn't directly affect circuit function (op-amp output never goes negative in this topology)
- Combined effect: compliance drops from ~12.5V to maybe ~10V — still workable but tighter

**Mitigations:**
1. **Bleeder resistor on -15V:** A 1kΩ from GND_ISO to -15V draws 15mA, partially balancing the load. Power waste: 225mW. Impacts battery life by ~10%.
2. **Move TLV70450 input to a separate source.** If the TLV70450 (the biggest +15V consumer at ~20mA) could be powered differently, balance improves dramatically. But there's no other isolated voltage source available.
3. **Accept it and measure.** On the breadboard, measure actual rail voltages under stimulation load. If +15V stays above +12V, the design works. If it sags further, add the bleeder.
4. **Use a regulated dual-output converter.** These exist but are more expensive and harder to source in SMD with 1500VDC isolation.

**Action:** Flag for breadboard verification. Add a bleeder resistor footprint on the PCB (can be populated or left empty based on bench results).

### 🟡 NEW-3: Op-Amp Saturation During H-Bridge Dead Time

During biphasic operation, the H-bridge switches between forward and reverse through a dead-time gap (50µs per the design). During this gap, if both H-bridge outputs go Hi-Z ("coast" mode) or both go LOW ("brake" mode depends on part), the current path through the load is broken.

**When the current path opens:**
- Rsense current drops to zero → V(-) = 0V
- V(+) = Vdac ≈ 0.1–0.5V
- Op-amp sees V(+) > V(-) → drives output to positive saturation (~+14V)
- This happens every 400µs (twice per biphasic period at 25Hz)

**When the H-bridge reconnects after dead time:**
- Op-amp is saturated at +14V
- Needs to recover and settle to the correct output voltage (e.g., 5.5V for 5mA into 1kΩ)
- ADA4522-2 overload recovery time is not well-specified for zero-drift/chopper amplifiers. Typical chopper amps recover in 10–50µs.
- The 10nF feedback cap (across Rsense) is charged to 0V during dead time. On reconnection, it acts as a momentary short across Rsense, causing an initial current overshoot while it charges to the steady-state V(Rsense).

**Combined effect:** For the first ~10–50µs after each dead-time gap, the output current may overshoot significantly (potentially 2–3× the target), then settle. At 200µs pulse width, this could mean 5–25% of the pulse is at the wrong current.

**This affects charge balance.** If the overshoot is asymmetric between positive and negative phases (due to different load conditions during H-bridge direction change), there's a net DC component. The DC-blocking cap handles this, but it's an unnecessary stress.

**Mitigations:**
1. **Use "brake" mode during dead time** (both low-side FETs ON, shorting the output to GND through Rsense). This maintains a current path, V(-) = V(Rsense) ≈ 0V, and V(+) = Vdac. The op-amp output goes to V(load) + V(Rsense) — a small positive voltage, NOT saturation. Much better.
   - For DRV8837: IN1=IN2=0 → brake mode ✅
   - For DRV8871: IN1=IN2=1 → brake mode ✅
   - For discrete MOSFETs: turn on both low-side N-FETs
2. **Zero the DAC during dead time.** Set DAC to 0 during the gap → V(+) = 0V → op-amp goes to ~0V output, no saturation. But with the 1kΩ/100nF RC filter (τ=100µs) on the DAC output, the voltage at V(+) won't reach zero in 50µs. Partial mitigation at best.
3. **Remove the 10nF feedback cap** (see NEW-4 below). This eliminates the cap-charging overshoot, though the op-amp saturation recovery issue remains.

**Recommendation:** Use brake mode during dead time. This is the cleanest solution — it keeps the feedback loop closed and the op-amp in its linear region. Verify on oscilloscope.

### 🟡 NEW-4: 10nF Feedback Cap — Pulse-Edge Current Overshoot

The recommended 10nF capacitor across Rsense (DESIGN-REVIEW.md §8, C_F1/C_F2 in BOM) is intended to filter high-frequency noise. Its time constant with Rsense is τ = 100Ω × 10nF = 1µs.

**At the leading edge of each current pulse:**
1. H-bridge connects the load to the op-amp output
2. Current begins flowing through Rsense
3. The 10nF cap across Rsense is initially uncharged (from the dead-time gap)
4. The cap acts as a momentary short across Rsense → V(-) stays near 0V
5. Op-amp sees V(-) < V(+) → drives harder → current overshoots
6. Cap charges with τ = 1µs → settles in ~5µs

**Overshoot magnitude estimate:**
During the first ~1µs, the cap effectively reduces Rsense from 100Ω toward 0Ω. The op-amp is trying to force V(-) = Vdac (0.5V for 5mA), but the effective feedback resistance is near zero, so the op-amp drives maximum output. The current overshoot is limited by the op-amp's slew rate and the load impedance.

Worst-case transient current ≈ V_out(saturated) / Z_load = 14V / 1kΩ = 14mA for ~1µs. This is 2.8× the 5mA target, but only for ~1µs out of a 200µs pulse. Charge contribution: 14mA × 1µs = 14nC, vs pulse charge of 5mA × 200µs = 1000nC. The overshoot adds ~1.4% extra charge per pulse edge — minor but real.

**If the feedback cap is removed:** the high-frequency noise it was meant to filter (chopper artifacts at ~4kHz → nA-level current ripple) is negligible. The cap was recommended as "belt-and-suspenders." I'd argue the belt is causing a wardrobe malfunction.

**Recommendation:** Start without the 10nF feedback cap. Add it only if bench testing reveals actual HF oscillation or noise issues. Leave the footprint on the PCB (C_F1, C_F2 as DNP — Do Not Populate).

### 🟡 NEW-5: Missing Vref in Final BOM

As noted in issue #3 above, the DESIGN-REVIEW.md final BOM (§10) does not include a voltage reference for the MCP4922. This is a functional gap — the DAC literally cannot produce an output without Vref.

The BOM has a "Resistor dividers, pullups — Various 0603 1% — qty 15" line that presumably includes a Vref divider, but this is ambiguous and not called out as a safety-critical component.

**Add to BOM explicitly:** MCP1501-10E/SN (or MCP1501-20E/SN), SOIC-8, ~$1.50. This should have its own line item, not be buried in "misc resistors."

### ℹ️ NEW-6: Power Budget Inconsistencies Between Documents

The two design documents disagree on power consumption numbers:

| Parameter | DESIGN-REVIEW.md | CIRCUIT-DESIGN-REVIEW.md |
|-----------|-----------------|------------------------|
| PCN1/B0515D input current from 5V | 256mA (max capacity) | 62mA |
| ESP32-S3 current from 5V (via LDO) | 50mA | 80mA avg, 350mA peak |
| Total system from LiPo | 330mA | 172mA avg, 440mA peak |
| Battery life estimate | 6 hours | 8+ hours |

The DESIGN-REVIEW.md uses the PCN1's maximum rated input (1W / 5V = 200mA, plus efficiency overhead), while the CIRCUIT-DESIGN-REVIEW.md calculates actual load. Neither accounts for the TLV70450's contribution to the +15V loading (see NEW-2 above).

**A realistic estimate:** With NEW-2's load analysis (~33mA from +15V, ~2mA from -15V):
- PCN1 output power: 15V × 33mA + 15V × 2mA = 525mW
- PCN1 input at ~78% efficiency: 525mW / 0.78 = 673mW → 135mA from 5V boost
- Boost converter at ~93%: 135mA × 5V / 0.93 = 726mW from LiPo → 196mA at 3.7V
- ME6211 for ESP32: ~150mA at 3.3V → 150mA directly from LiPo
- Total from LiPo: ~346mA
- Battery life: 2000mAh / 346mA ≈ **5.8 hours** → ~11 sessions at 30min

This is slightly worse than either document estimates but still adequate. The discrepancy is because neither document fully accounts for TLV70450 loading from +15V.

**Not blocking — but the power budget should be reconciled into a single authoritative number.**

### ℹ️ NEW-7: Document Contradiction — DRV8871 "Approved" But Incompatible

(Elaborated from issue #1 assessment above.)

DESIGN-REVIEW.md contains a direct internal contradiction:
- **§1 (line 37):** "The DRV8871DDAR specifically is correct"
- **§8 (line 425):** Circuit diagram shows "DRV8871 VM pin" fed by op-amp output (the problematic topology)
- **§10 (line 575):** BOM lists 2× DRV8871DDAR
- **§12 (line 712):** Summary lists "2× DRV8871" as the resolved binaural architecture

Meanwhile CIRCUIT-DESIGN-REVIEW.md §2, issue #4, correctly explains why DRV8871 cannot work in this topology.

**The DRV8871 was approved in DESIGN-REVIEW.md based on its spec sheet (45V operating, integrated protection, IPROPI) without checking whether VM = op-amp output would violate the 6.5V minimum.** The spec sheet analysis is good — those ARE nice features. But the minimum operating voltage makes it irrelevant.

**This needs a clear correction in DESIGN-REVIEW.md and AGENTS.md.** Anyone reading the project documents right now will get contradictory guidance.

---

## Part 3: What Would Worry Me on the Breadboard

In order of what I'd verify first:

### 1. H-Bridge Selection (Must Resolve Before Breadboard)

You can't breadboard the signal chain without a working H-bridge. The DRV8871 won't work. The discrete BSS84 approach won't work without level-shifted gate drivers (NEW-1). **Start with the DRV8837.** It's available, cheap, and has the right VM range. The 11V compliance limit is fine for initial verification — you can upgrade later if bench testing shows you need more headroom.

Order 4× DRV8837DSGR (2 per channel, plus spares). SOT-23-6, easy to dead-bug on a breadboard.

### 2. PCN1-S5-D15-M-TR Cross-Regulation (Measure First Day)

Hook up the PCN1 with representative loads on both rails. Put 30mA on +15V and 2mA on -15V. Measure actual rail voltages. If +15V sags below +12V, you need a bleeder on -15V or a different power strategy. **This takes 10 minutes and could save weeks of debugging mysterious current regulation errors.**

### 3. Op-Amp Behavior During H-Bridge Transitions

Set up the ADA4522-2 with Rsense and a DRV8837 driving a 1kΩ resistor (simulating the electrode). Run biphasic pulses at 25Hz and look at:
- Current overshoot at pulse edges (especially after dead time)
- Op-amp output during dead time — is it saturating?
- Recovery waveform when H-bridge reconnects
- Effect of brake mode vs coast mode during dead time

This is the most likely source of unexpected behavior. The prose analysis says everything is fine; the scope will tell the truth.

### 4. Charge Balance Verification

Run biphasic stimulation into a 1kΩ load for 30 minutes. Measure the DC voltage across the DC-blocking cap. It should stay under ±10mV. If it drifts, there's a charge imbalance — find the source (asymmetric settling, timing errors, offset).

### 5. DAC Vref Settling and Accuracy

Whatever Vref you choose, verify the actual output voltage and noise at the MCP4922 output. Set the DAC to code 500 (should give 2.5mA at Rsense = 100Ω). Measure actual current. Compare to expected. If they differ by >2%, the Vref needs attention.

---

## Summary Table

| # | Issue | Status | Severity | Action |
|---|-------|--------|----------|--------|
| 1 | DRV8871 6.5V min VM | Agree w/ prior reviewers, documents contradict | 🔴 Critical | Remove from BOM, correct DESIGN-REVIEW.md §1 |
| 2 | No MISO reverse channel | Agree, slightly understated | 🟡 Significant | Swap SI8621EC for SI8622EC, steal 1 forward from SI8380P for heartbeat |
| 3 | Vref undecided | Agree, more serious than stated | 🟡 Significant | Add MCP1501-10E/SN to BOM. Stop deferring. |
| 4 | PPTC as primary overcurrent | Agree | ✅ Already addressed | LM339 latch is the fix. Verify on bench. |
| 5 | DC-block MLCC vs film | Partially disagree | 🟡 Significant | X7R is OK electrically, but **short-circuit failure mode** is the real concern. Use 2× in series. |
| 6 | Emergency stop too small | Agree, low priority | ℹ️ Minor | Upgrade in Phase 8 (PCB). |
| 7 | Type BF compliance language | Agree | ℹ️ Minor | Change "meets" to "targets" throughout. |
| 8 | 89° phase margin claim | Agree overstated | ℹ️ Minor | Real margin ~50–70°. Conclusion (stable) is correct. Verify with SPICE or bench. |
| **NEW-1** | **BSS84 gate drive failure** | **New finding** | **🔴 Critical** | **Discrete P-channel H-bridge unworkable at >6V. Use DRV8837 instead.** |
| **NEW-2** | **PCN1 unbalanced rail loading** | **New finding** | **🟡 Significant** | **+15V draws 16× more than -15V. Measure cross-regulation. Add bleeder pad.** |
| **NEW-3** | **Op-amp saturation in dead time** | **New finding** | **🟡 Significant** | **Use brake mode during dead time. Verify on scope.** |
| **NEW-4** | **10nF feedback cap overshoot** | **New finding** | **ℹ️ Minor** | **Start with cap unpopulated (DNP). Add if HF issues seen.** |
| **NEW-5** | **Missing Vref in BOM** | **New finding (oversight)** | **🟡 Significant** | **Add MCP1501 as explicit line item.** |
| **NEW-6** | **Power budget inconsistencies** | **New finding** | **ℹ️ Minor** | **Reconcile into single authoritative estimate (~346mA, ~5.8h).** |
| **NEW-7** | **Document contradiction on DRV8871** | **New finding** | **🟡 Significant** | **Correct DESIGN-REVIEW.md §1, §10, §12. Update AGENTS.md.** |

---

## Closing Notes

The fundamental topology — V-to-I converter with op-amp error amp, sense resistor feedback, H-bridge polarity switching, galvanic isolation — is sound. The safety architecture (7 independent layers) is well-designed and appropriately paranoid for a patient-contact device. The component selection is mostly good, with the ADA4522-2ARZ and PCN1-S5-D15-M-TR being solid choices.

The two biggest issues are both about the H-bridge: the DRV8871 can't work here, and the recommended discrete BSS84 replacement also can't work without level-shifting. The DRV8837 is the pragmatic answer — it sacrifices ~20% compliance headroom for a dramatically simpler and working design. That's a trade I'd take every day.

The unbalanced rail loading (NEW-2) and dead-time saturation (NEW-3) are the kind of things that don't show up in component-level reviews but will show up on the breadboard. They're both manageable — bleeder resistor and brake mode, respectively — but they need to be on the bringup checklist.

If I had one request: **reconcile DESIGN-REVIEW.md and CIRCUIT-DESIGN-REVIEW.md into a single canonical document.** Having two design reviews that disagree on the H-bridge, the Vref strategy, the DC-blocking cap material, and the power budget is a trap for whoever picks this project up next.

---

*Review complete. I found 7 new issues (2 critical, 3 significant, 2 minor). The most important new finding is that the recommended discrete BSS84 H-bridge alternative has the same class of problem as the DRV8871 — it doesn't work in the intended circuit. The DRV8837 emerges as the clear practical choice.*
