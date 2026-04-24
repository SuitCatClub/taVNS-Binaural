# OpenBinaural-taVNS — Electrical Design Review (Third Pass)

**Date:** 2026-04-24
**Reviewer:** Ren (claude-sonnet-4.6 — warm review, full document read)
**Documents reviewed:** AGENTS.md, DESIGN-REVIEW.md (~38KB), CIRCUIT-DESIGN-REVIEW.md (~30KB), REVIEW-OPUS-WARM-2026-04-24.md, REVIEW-GPT54-WARM-2026-04-24.md, REVIEW-OPUS-WARM-V2-2026-04-24.md, REVIEW-GPT54-WARM-V2-2026-04-24.md, DESIGN-ACTIONS-2026-04-24.md

---

## Approach

I read everything before writing. I treated the DESIGN-ACTIONS document as the current synthesis of prior reviewer findings, which means anything listed there I don't need to rediscover — I just need to verify it or push back on it. My job is the delta: things that are new, things that are wrong in the existing analysis, and anything the prior four reviewers each individually missed but might become visible when you read them all together.

---

## Part 1 — On the 8 Known Issues

### 1. DRV8871 minimum VM — Agree, critical. One new angle.

All four prior reviewers agree on this. The contradict in DESIGN-REVIEW.md §1 ("DRV8871 APPROVED") vs. the rest of the documents is correctly flagged and the DESIGN-ACTIONS document resolves it as DRV8837. I won't re-argue this.

**My angle no one mentioned:** DESIGN-REVIEW.md §1 approves DRV8871 based on "45V operating / 50V abs max → handles full ±15V compliance voltage." This is analyzing the WRONG rating. The 45V abs max is the maximum supply voltage that can be applied to VM — not the minimum. The reviewer in DESIGN-REVIEW.md §1 confused the voltage range ceiling with the floor, then built a whole confidence argument on it. This is worth noting explicitly so nobody re-introduces DRV8871 via the same reasoning path.

### 2. No SPI MISO reverse channel — Agree. But there's a deeper measurement problem I'll address in Part 2.

The SI8622EC-B-IS swap (getting 2 reverse channels, reallocating heartbeat to SI8380P channel 8) is the right fix. Clean, no BOM count change. Agree with Mara-Opus V2 on this. **But**: having a MISO path back doesn't solve the problem unless the ADC is measuring the right thing. See New Finding B below.

### 3. Vref strategy — Agree, DESIGN-ACTIONS resolution (MCP1501-10E/SN) is correct.

MCP1501-10 with MCP4922 in 2× gain → full-scale 2.048V → I_max = 20.48mA → 5µA/LSB. At 5mA clinical max: DAC code ≈ 1000/4096 (24% of range). Mara-Opus V1 notes you only use 24% of the DAC range. This is fine — 10-bit effective resolution at 5µA/step is more than adequate for a stimulator where the user titrates by feel.

One addition nobody mentioned: **the MCP4922 Vref input pin has an input resistance of approximately 40kΩ (from the datasheet).** At Vref = 1.024V, this draws ~25µA from the MCP1501. The MCP1501 output rating is ±10mA, so there's no loading issue. But if someone tries the resistor-divider approach, this 25µA load shifts the divider output and needs to be accounted for in the resistor selection.

### 4. PPTC fuse as primary overcurrent — Agree. Correctly flagged and addressed.

The LM339 comparator latch as primary fast protection (< 10µs) with PPTC as passive backup is the right layered approach. The DESIGN-ACTIONS resolution is correct. Nothing new to add here.

### 5. DC-blocking cap MLCC vs film — Agree with the DESIGN-REVIEW analysis on electrical performance. Agree with Mara-Opus V2's resolution (2 MLCCs in series).

The dual series MLCC approach (two Samsung CL31B106KBHNNNE in series = 5µF effective per channel) handles the fail-short failure mode concern without requiring film caps. 5µF still passes the droop analysis: V_droop = 1µC/5µF = 0.2V at 5mA/200µs — acceptable.

**I want to add a concern nobody raised:** X7R MLCC capacitors exhibit piezoelectric behavior (the inverse effect — voltage → mechanical deformation). At 20–25Hz stimulation frequency with any ripple voltage on the cap, these capacitors will vibrate mechanically. The sound pressure is negligible for electrical use, but this is a **sleep device used in a quiet bedroom**. A 1206 MLCC physically vibrating at 20Hz on a circuit board can generate an audible hum or tick. This is not a safety issue, but it is a user experience issue that could make the device annoying to use for the exact application it's designed for. Film capacitors do not have this problem. I'm not saying the Samsung MLCC will definitely be audible — I'm saying this should be tested during breadboard validation. If audible, it's a genuine reason to find a compact film cap, or to evaluate C0G (zero piezo effect) at reduced capacitance.

### 6. Emergency stop switch — Agree. Low priority for breadboard, real concern for final PCB.

Mara-Opus V1's recommendation of a 12mm+ illuminated pushbutton or slider is correct. I'd add: if the device will be used in a dark bedroom, **the emergency stop needs to be findable by touch alone without visual reference.** A physically raised, textured button on the top face of the enclosure is more useful than any switch size spec.

### 7. Type BF / 1 MOPP language — Agree. Change to "designed toward BF-style isolation."

Nothing to add beyond what was already said. The component ratings aim at the right numbers, but compliance requires schematic review, hipot testing, leakage measurement, and risk management documentation.

### 8. Loop stability 89° claim — Agree it's overstated. But I want to flag a specific error.

Mara-Opus V2 already caught one error in the stability section (the chopper note says "159kHz — well below chopper" when 159kHz is well *above* the ~4kHz chopper clock). I want to add a second issue with the stability analysis:

The zero-pole cancellation argument (Rsense creates a zero at 15.9kHz that nearly cancels the electrode pole at 1.6kHz, leaving 89° margin at 2.7MHz crossover) assumes these are the only two reactive elements in the loop. In a real board:

- The DC-blocking cap (~10µF or 5µF after the series pair) is in the *current path* from H-bridge output through electrode to Rsense. It adds another RC element to the loop.
- At 2.7MHz crossover, a 5µF cap has ~12mΩ impedance — essentially a short at that frequency, so it doesn't add a pole in the GBW calculation. Fine.
- But the **lead wires** from the device to the ear (expected 30–60cm) add inductance (5–15nH typical for electrode cables). At 2.7MHz, L = 10nH → Z_L = 2π × 2.7MHz × 10nH ≈ 170mΩ. Also negligible.

So the 89° calculation is wrong in precision but probably right in direction: the loop is stable with a realistic phase margin somewhere in the 50–70° range as Mara-Opus V2 estimates. The broader concern (no SPICE) stands.

---

## Part 2 — New Findings

These are things I don't see explicitly addressed in any prior review or the DESIGN-ACTIONS document.

### 🔴 NEW-A: DRV8837 VM absolute maximum is a component damage risk, not just a compliance limit

This is different from what prior reviewers said, and it matters.

Prior reviewers flagged the DRV8837's 11V VM maximum as a "compliance ceiling" — meaning you can't deliver more than ~2kΩ load at 5mA without compliance limiting. That framing treats 11V as a design limit to work around.

**The actual issue is more serious: exceeding 11V on DRV8837 VM is an absolute maximum rating violation that can permanently damage the IC.**

The feedback loop works like this: the op-amp adjusts VM upward until V(Rsense) = V(DAC). If electrode impedance is higher than the DRV8837 can handle at the set current, the op-amp keeps increasing its output toward +13V trying to force the required current. When VM exceeds 11V, the DRV8837 is over-spec. The op-amp does not know this and will not self-limit.

The current impedance abort threshold in AGENTS.md is **Z > 10kΩ → abort**. At 5mA and Z = 10kΩ, the required VM would be 5mA × (10,000 + 100)Ω = 50.5V. That's well beyond DRV8837 limits and would actually be limited by op-amp compliance first. The mismatch between the impedance abort threshold (10kΩ) and the DRV8837 damage threshold (~2kΩ at 5mA) means there's a zone — Z between 2kΩ and 10kΩ — where the firmware considers it a valid operating condition but the DRV8837 is being damaged.

**Required action:** The impedance abort thresholds must be differentiated by current setpoint:

```
At I = 5mA: Z_max = (11V - V_sat) / 5mA ≈ 10.4V / 5mA ≈ 2.0 kΩ → abort if Z > 2kΩ
At I = 2mA: Z_max = 10.4V / 2mA ≈ 5.2 kΩ → abort if Z > 5kΩ
At I = 1mA: Z_max = 10.4V / 1mA ≈ 10.4 kΩ → abort if Z > 10kΩ
```

These are not the same as the "bad electrode contact" threshold (10kΩ) — they're tighter and current-dependent. The firmware must check impedance against a current-scaled limit before enabling stimulation, and AGENTS.md should document this table rather than a single 10kΩ threshold.

If discrete MOSFETs with full ±15V are eventually used, the Z_max at 5mA becomes ~2.4kΩ (12.5V/5mA - Rsense). Still tighter than 10kΩ. The 10kΩ threshold in AGENTS.md is not meaningful unless the current is very low.

---

### 🟡 NEW-B: Impedance measurement requires compliance voltage, not just sense resistor current

This is a measurement architecture gap that the MISO-reverse-channel discussion doesn't fully address.

The design places MCP3201 ADC on the isolated side to measure impedance. The current design has the ADC measuring voltage across Rsense. That gives: `I = V(Rsense) / 100Ω`. It does **not** give impedance.

To compute `Z = V_load / I`, you need **V_load separately**. In this topology:

```
V_load = V_opamp_output - V_Rsense - V_Hbridge_drop
```

None of those signals are currently connected to the ADC input. The MCP3201 on Rsense tells you current only.

**How impedance is actually computed in practice:**

Option 1 — Measure compliance voltage: Add an ADC input to the op-amp output node (before the H-bridge). Then V_load = V_compliance - I × Rsense - V_drop. Requires a second ADC channel or multiplexed input.

Option 2 — Use the servo equation: If the DAC setpoint (Vdac) is known and the current is servo-controlled, then at steady-state: V_opamp = I × (Z + Rsense). Since I = Vdac/Rsense, we have V_opamp = Vdac/Rsense × (Z + Rsense). The firmware knows Vdac, can read I from ADC, and therefore: `Z = (V_opamp/I) - Rsense`. But V_opamp still needs to be measured.

Option 3 — Apply a sub-threshold test pulse at a known current, then vary current and measure V(Rsense) at two levels. If the system is current-regulated, V(Rsense) is constant regardless of Z. This doesn't work directly.

Option 4 — Open-circuit voltage measurement: briefly interrupt the current (H-bridge coast), let V(Rsense) → 0. While open-circuit, the compliance voltage tells you the electrode open-circuit potential. Then close the circuit, measure V(Rsense). This gives a two-point impedance estimate.

**The simplest working solution:** Add a second MCP3201 ADC channel measuring the op-amp output (compliance voltage node) via a resistor divider (the 13V max needs to be scaled to the ADC's 5V_ISO range). This adds one more ADC input trace on the isolated side. MCP3201 is single-ended single-channel, so you'd need a dual-channel ADC (MCP3202 is the drop-in dual-channel equivalent) or a multiplexer.

This is a concrete gap that needs a concrete solution before the impedance sensing feature is designed. The ACTIONS document has it as "add MISO channel" but that's necessary, not sufficient.

---

### 🟡 NEW-C: TPS61023 switching frequency — documented inconsistency affects noise argument

DESIGN-REVIEW.md claims 4MHz switching frequency and uses this to justify choosing TPS61023 over alternatives:
> "The 4MHz switching frequency is the decisive factor. At 4MHz, output ripple is 10–20× lower than 300–600kHz alternatives."

CIRCUIT-DESIGN-REVIEW.md states 1.8MHz.

DESIGN-REVIEW.md then notes on its own: "Design for ~4 MHz switching frequency (verify from TI datasheet — distributor listing showed 1 MHz which may be an error)."

This self-contradiction in the same document suggests the 4MHz number is unverified. The DESIGN-ACTIONS document flags this as a documentation inconsistency to merge, but the actual frequency matters for noise analysis.

**Why it matters:** The CM choke (Würth 744232601, 600Ω @ 100MHz) and the 47µF + 100nF decoupling on the PCN1 input were sized expecting a 4MHz switching signal. At 1.8MHz, the filtering effectiveness is lower:
- CM choke impedance at 1.8MHz: roughly 50–150Ω (from the frequency-vs-impedance curve) vs. ~600Ω at 100MHz. The choke is less effective at the switching fundamental.
- The LC filter resonance with 47µF: f_res ≈ 1/(2π√(L_choke × 47µF)). Without knowing L_choke, can't calculate exactly.

**Action:** Verify TPS61023 switching frequency from TI datasheet before finalizing noise filtering components. If 1.8MHz (not 4MHz), the justification for choosing TPS61023 over alternatives still holds (it's lower than the audio range and the isolation barrier separates the digital noise from the analog domain), but the specific ripple comparison in the design document is wrong and should be corrected.

---

### 🟡 NEW-D: LM339 comparator latch and power-up state — implementation not specified

DESIGN-ACTIONS item #10 says "verify latch power-up default is 'safe' (no stimulation)." This is correct as a requirement. But the implementation is not specified anywhere.

The SR latch is typically implemented with a flip-flop IC (SN74LVC1G80 or similar). Power-up state of CMOS SR latches is indeterminate unless there's a reset mechanism. For a fail-safe design:

- **Required behavior:** Latch powers up in the SET state (Q=1 = "fault asserted" = H-bridge disabled). Stimulation only starts after firmware explicitly clears the latch.
- **Implementation:** Add a power-on-reset (POR) signal from the ESP32 (or from a dedicated RC reset circuit on 5V_ISO) to the latch RESET input — but active HIGH on RESET drives Q=0 (armed), which is the WRONG default. Need to ensure the reset signal holds RESET LOW at power-up and releases after initial diagnostics pass.

A simple implementation: add a pull-down resistor on the SR latch's RESET input (keeps RESET LOW at power-up → latch stays SET), and have the firmware explicitly pulse RESET HIGH after completing impedance verification and safety checks.

This is a small circuit detail but it's the difference between a device that safely refuses to stimulate on startup versus one that starts in an armed state. Nobody has specified the implementation, only the requirement.

---

### 🟡 NEW-E: TP4056X charging current — PROG resistor value not specified anywhere

The TP4056X charging IC requires an external PROG resistor to set the charging current. The formula is `IPROG = 1000 / RPROG` (A), so:
- RPROG = 1kΩ → 1A charging
- RPROG = 2kΩ → 500mA charging

For a 2000mAh LiPo:
- 1C = 2A (too fast, reduces cycle life)
- 0.5C = 1A (appropriate, RPROG = 1kΩ)
- 0.25C = 500mA (conservative, RPROG = 2kΩ)

This value appears nowhere in AGENTS.md, DESIGN-REVIEW.md, or the BOM. It's a small resistor but it determines whether the battery charges in 2 hours or 4 hours and affects long-term battery health.

**Action:** Add `R_PROG = 1kΩ 0603 1%` to the BOM with an explicit note that it sets 1A charging for the 2000mAh cell. If hand-soldered boards vary, wrong RPROG could undercharge (frustrating) or stress the battery. Minor but should be explicit.

---

### ℹ️ NEW-F: SI8380P-IU QSOP-20 package — hand soldering difficulty for a DIY build

The SI8380P-IU is in a QSOP-20 package (20 pins, 0.635mm pitch). This is a fine-pitch SMD package that requires a steady hand, flux, and solder wick to hand-solder reliably. Most DIY builders can manage SOIC (1.27mm pitch) without issues; QSOP-20 at 0.635mm is a meaningful step harder.

This doesn't mean it's wrong — SI8380P-IU is the best single-IC solution for 8 forward channels. But for breadboard prototyping (Phase 2), the QSOP-20 is difficult to use on a standard breadboard adapter. If a breakout board isn't available, an alternative for the breadboard phase would be 2× SI8642EC (4-channel, SOIC-16, 1.27mm pitch) + 1× SI8622EC (2-channel, SOIC-8). More ICs but easier to prototype. Switch to SI8380P-IU for the PCB phase.

---

### ℹ️ NEW-G: Bleed resistor change to 100kΩ — check leakage headroom

DESIGN-ACTIONS item #12 suggests changing the bleed resistor from 1MΩ to 100kΩ for faster DC-blocking cap discharge (τ from 10s to 1s). This is reasonable for inter-session charge clearance.

**What to verify:** The bleed resistor creates a DC leakage path from the op-amp output through the electrode to ground. At any residual voltage across the cap (e.g., 1V from drift) and with a 100kΩ bleed: I_leak = 1V / 100kΩ = 10µA. IEC 60601-1 Type BF patient leakage limit is 100µA in normal conditions. 10µA is within spec.

But at the compliance rail voltage (if the cap ever charges toward 13V in a fault), leakage through 100kΩ = 130µA, which exceeds the 100µA limit. The bleed resistor is not in parallel with the full compliance voltage in normal operation (normal operation keeps cap near 0V DC), but fault analysis should include this path.

**Recommendation:** Keep 1MΩ if the 10s discharge time is acceptable. If 100kΩ is used, explicitly document that fault analysis at 13V worst-case gives 130µA and ensure this is covered by the comparator latch cutting power before extended cap charging can occur.

---

## Part 3 — What Would Worry Me Most if Breadboarding Next Week

In order of concern:

### 1. DRV8837 VM ceiling and the impedance abort logic (NEW-A)

If I'm breadboarding next week, I'll be connecting real ear clips to real ears. Electrode impedance in practice varies widely: poor contact → Z > 5kΩ is common at first placement. At 5mA into Z = 5kΩ, the op-amp would try to drive VM = 5mA × 5.1kΩ = 25.5V. The DRV8837 sees 11V abs max. I'd damage the IC on my first session with bad electrode contact.

The impedance check must be implemented before ANY current is applied. The check itself must use a sub-mA test current to keep VM below 11V during the check. Before breadboarding: wire in the MCP3201 ADC, verify the impedance measurement works, and set current-dependent abort thresholds as described in NEW-A.

### 2. Op-amp saturation during dead time (from prior review, Mara-Opus V2 NEW-3)

At every inter-phase gap (50µs), if the H-bridge coasts (Hi-Z), the op-amp sees V(-) = 0 and immediately ramps to +14V output. When the next phase starts, there's a current overshoot. This is a systematic charge imbalance source that affects every single biphasic cycle.

The fix (brake mode: DRV8837 IN1=IN2=0 during dead time) is simple in firmware but **must be verified on a scope during first bringup**. Put a current probe in series with the electrode and look at the first 50µs of each pulse. You want to see a clean rising edge, not a 14mA spike followed by settling. If you see the spike, the overshoot is real and brake mode is essential.

### 3. PCN1 unbalanced rail loading (from prior review, Mara-Opus V2 NEW-2)

+15V loaded at ~33mA, -15V at ~2mA. I have no idea what the actual rail voltages will be until I measure them. For the first bringup, measure +15V and -15V before connecting anything to the patient domain. If +15V is significantly below 15V (say, 12V), the compliance headroom drops. If the TLV70450 on +15V has only 12V input instead of 15V, the LDO can't maintain 5V_ISO with 3V headroom (TLV70450 dropout is ~150mV). Actually dropout isn't a problem here: (12-5)/5=1.4V headroom >> 150mV. Fine. But if +15V sags to 8V, the DAC power rail would still be OK (5V from 8V), but op-amp compliance drops to ~6.5V. At 2mA therapeutic current, Z_max = 6.5V / 2mA = 3.25kΩ — still fine for well-prepared electrodes, but the headroom is eaten away.

Before patient use, add a 1kΩ bleeder on the -15V rail and measure rail voltages under typical load. Adjust if needed.

### 4. The impedance measurement architecture gap (NEW-B)

If I can't compute Z from just V(Rsense), the pre-session contact check doesn't give me actual impedance. I can compute I from Rsense, but I can't compute Z without the compliance voltage. For breadboard phase, I'd wire a second ADC input to the op-amp output (through a voltage divider to scale from 13V to 5V range) to get V_compliance. Then Z = (V_compliance - V_Rsense) / I.

---

## Summary Table

| # | Finding | Severity | New? |
|---|---------|----------|------|
| 1 | DRV8871 minimum VM — prior analysis misidentified the relevant spec | 🔴 | Confirms prior, adds new angle |
| 2 | SPI MISO reverse channel | 🟡 | Confirms prior |
| 3 | Vref — MCP1501-10E/SN correct, plus Vref input loading note | 🟡 | Confirms prior, minor addition |
| 4 | PPTC as primary OCP | 🟡 | Confirms prior |
| 5 | MLCC DC-block — series redundancy correct; adds acoustic noise concern | 🟡 | Confirms prior, adds acoustic risk |
| 6 | Emergency stop | 🟡 | Confirms prior |
| 7 | Type BF language | ℹ️ | Confirms prior |
| 8 | Loop stability 89° — second analysis error found | 🟡 | Confirms prior, adds to |
| A | DRV8837 VM = component damage risk, not just compliance loss | 🔴 | **New** |
| B | Impedance measurement needs compliance voltage, not just Rsense current | 🟡 | **New** |
| C | TPS61023 frequency discrepancy (4MHz claimed vs 1.8MHz) affects noise argument | 🟡 | **New** |
| D | LM339 latch power-up implementation unspecified | 🟡 | **New** |
| E | TP4056X PROG resistor value never specified | ℹ️ | **New** |
| F | SI8380P-IU QSOP-20 not practical for breadboard phase | ℹ️ | **New** |
| G | Bleed resistor 100kΩ — check leakage at fault voltage | ℹ️ | **New** |

---

## What I Didn't Find

I looked for issues with the binaural stagger timing (LDAC shared between channels), the ADA4522 chopper clock interaction with stimulation frequency, and the TLV70450's ability to handle unregulated PCN1 output variations. None of these are problems in my read:
- LDAC timing for 20ms stagger is handled correctly in firmware (H-bridge timing controls the stagger, DAC codes are pre-loaded)
- Chopper clock (~4kHz) is 160× above stimulation frequencies; any current ripple it causes is negligible
- TLV70450 input (up to ~18V at PCN1 no-load) is within its 24V abs max; power dissipation stays reasonable

I also looked at the safety-shutdown path ambiguity (GPT-5.4's F7). The prior reviews flagged it correctly. I don't have more to add beyond noting that Mara-Opus V2's brake-mode recommendation is actually the cleanest answer to "define the H-bridge state in every condition" — coast mode is the ambiguous state, brake mode is the defined one.

---

*End of review. Key new contributions: NEW-A (DRV8837 damage risk), NEW-B (impedance measurement architecture), NEW-C (TPS61023 frequency), NEW-D (latch power-up state), NEW-E (PROG resistor), NEW-F (QSOP-20 solderability), NEW-G (bleed resistor leakage check).*
