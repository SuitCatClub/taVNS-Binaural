# Electrical Design Review — OpenBinaural-taVNS

**Date:** 2026-04-24  
**Reviewer:** GitHub Copilot CLI (electrical design review)  
**Scope reviewed:** Topology, component choices, safety architecture, power chain, signal chain, isolation barrier, and the DRV8871 question, using the supplied design context plus repo review documents.  

## Scope note / uncertainty
I did **not** find KiCad schematic or PCB layout files in the repo at review time, so this is a **design-architecture review**, not a pin-for-pin ERC/layout review. That means creepage/clearance, actual latch wiring, decoupling placement, return-current routing, and connector implementation still need schematic/layout review before release.

## Executive summary
The design is pointed in the **right overall direction**: battery-powered, isolated patient domain, unipolar DAC, precision current regulation, biphasic reversal with a switching stage, hardware lockout during USB charging, and a separate isolated logic rail are all the right instincts.

The biggest remaining issue is also the most important one:

> **As currently described, the DRV8871-based signal chain is not electrically valid.**

The contradiction you flagged is real. If the op-amp output is feeding the DRV8871 **VM** pin, the bridge will sit below its 6.5 V UVLO for much of the intended operating range, and more fundamentally a motor-driver VM pin is not a good place to implement an analog servo. I would not freeze the architecture until that is redesigned.

## What looks solid

1. **ME6211 instead of AMS1117:** correct call for single-cell LiPo.
2. **Boost stage before isolated DC-DC:** mandatory and correctly recognized.
3. **ADA4522-2 instead of low-voltage zero-drift parts:** correct voltage-class fix.
4. **Dedicated isolated-side 5 V rail:** correct; the DAC and isolated-side logic need it.
5. **Separate patient domain with digital isolators + isolated power:** correct system partition.
6. **Charge-balanced biphasic intent + hardware DC blocking:** absolutely the right safety posture.
7. **USB-charge lockout via hardware EN path:** good idea and worth keeping.
8. **Compliance target (~12.5–13 V effective):** reasonable for auricular electrode loads if electrode prep is decent.

---

## Findings

### F1 — DRV8871 topology is not viable as drawn
**Severity:** critical

**Issue**  
The stated signal chain is:

`DAC -> ADA4522 -> DRV8871 VM -> H-bridge -> load -> Rsense -> op-amp feedback`

That makes the op-amp output act like the DRV8871 motor supply. The TI product page states **DRV8871 Vs(min) = 6.5 V** and includes **VM undervoltage lockout**. Your normal operating region includes much lower commanded voltages; e.g. low-current / low-impedance cases can put the servo output far below 6.5 V.

**Why it matters**  
This is not a corner case; it breaks the architecture over a large part of the operating envelope. Also, even if UVLO did not exist, a motor-driver **VM** pin expects a low-impedance decoupled supply, not a precision op-amp servo node. The bridge’s internal switching, charge-pump/gate-drive behavior, and local decoupling current pulses are a bad fit for being powered directly by the op-amp output.

**Suggested fix**  
Redesign this stage in one of these ways:

1. **Preferred:** keep the H-bridge on a fixed supply rail and regulate current with a dedicated source/sink element (transistor/op-amp controlled) rather than modulating the bridge supply pin.
2. If you want the bridge to directly steer polarity, use a topology where the bridge is a **switching element**, not the analog-controlled supply input.
3. If you stay with an integrated bridge, choose it only after the fixed-supply current-path architecture is settled.
4. If you go discrete, validate dead-time and output-state behavior on the bench.

**Bottom line**  
The DRV8871 contradiction is real, and I would treat this as an architecture blocker.

---

### F2 — DRV8871 IPROPI is not a credible 0.5–5 mA contact/impedance monitor
**Severity:** critical

**Issue**  
The design leans on **DRV8871 IPROPI + LM339** for electrode contact detection and impedance abort logic. That is very likely the wrong sensing mechanism for this current range.

**Why it matters**  
At stimulation currents of only **0.5–5 mA**, motor-driver current-report outputs are usually operating near the bottom of their useful range. Noise, offset, blanking behavior, PWM/state dependence, and gain tolerance can swamp the signal. More importantly, **current alone does not give impedance** in a current-regulated stimulator. If current is servo-controlled, you also need a voltage measurement (load voltage, compliance voltage, or both) to infer contact quality.

**Suggested fix**  
Use one of these approaches instead:

1. **Pre-session test:** inject a known sub-threshold current and measure the resulting patient-domain voltage with an ADC/INA stage.
2. **During session:** monitor the current-source compliance node (or load differential voltage) to detect open-circuit / near-short conditions.
3. Keep the comparator latch for hard faults, but do not count IPROPI alone as clinically meaningful impedance detection.

**Practical rule**  
For contact detection, measure **V and I**, not I alone.

---

### F3 — PPTC fuse is not a fast patient overcurrent protector
**Severity:** critical

**Issue**  
The 10 mA PPTC is being treated as part of the patient overcurrent safety chain.

**Why it matters**  
A PPTC is a **thermal** device. It reacts over milliseconds to seconds depending on overload magnitude and ambient temperature. It is useful as a secondary fault/fire-energy limiter, but it is **not** a precise or fast hardware current cutoff for short stimulation pulses or fault spikes.

**Suggested fix**  
Treat the PPTC as **secondary protection only**. The primary patient-current limit should be:

1. a fast analog current-limit/shutdown path, and/or
2. a comparator + latch tied to a real sense resistor in the stimulation path, and/or
3. a hardware disable that forces the output stage to Hi-Z / power-off on fault.

If the goal is “never exceed X mA at the patient,” the PPTC cannot be the primary answer.

---

### F4 — “Type BF / 1 MOPP” is not yet demonstrated by the current evidence
**Severity:** significant

**Issue**  
The review language overstates compliance confidence. A **1500 VDC isolation rating on the DC-DC converter** is not enough by itself to say the design “meets Type BF” or “meets 1 MOPP.”

**Why it matters**  
Real safety depends on the full system: converter construction, isolator ratings, barrier capacitance, PCB creepage/clearance, connector spacing, contamination assumptions, leakage-current paths, charger behavior, and what happens when USB is connected.

**Suggested fix**  
Change the project wording to something like:

> “Designed toward battery-powered BF-style patient isolation; not verified to IEC 60601-1 compliance.”

Then explicitly verify, at schematic/layout stage:

- creepage and clearance across the barrier,
- patient leakage current with USB connected and disconnected,
- barrier capacitance/common-mode coupling,
- fail-safe behavior during charger, brownout, and fault conditions.

The architecture is sensible, but the compliance claim is ahead of the evidence.

---

### F5 — The selected 10 µF / 50 V X7R DC-block capacitor is a weak point
**Severity:** significant

**Issue**  
Using a **1206 X7R MLCC** as the series patient-coupling/DC-block capacitor is possible, but I would not call it a robust choice without bench proof.

**Why it matters**  
For a safety-critical coupling capacitor, class-II MLCCs bring several annoyances:

- DC-bias capacitance derating,
- aging,
- piezoelectric/microphonic behavior,
- dielectric absorption and non-ideal pulse behavior.

A nominal 10 µF X7R may be far below 10 µF under real bias and tolerance conditions.

**Suggested fix**  
Preference order:

1. **Film/PPS solution** if board area allows,
2. otherwise **parallel MLCCs with measured effective capacitance at bias**, soft-termination if possible,
3. always keep the **bleed resistor** across the cap,
4. verify pulse droop and residual DC after a long session on a dummy load.

I am not saying the X7R is automatically unsafe; I am saying it is not the place I would optimize for size first.

---

### F6 — Stability/compliance analysis is plausible, but the quoted confidence is too high
**Severity:** significant

**Issue**  
Statements like “100 Ω sense resistor creates a zero that cancels the electrode pole” and “phase margin ~89°” are presented too confidently for the level of model detail shown.

**Why it matters**  
The real loop includes:

- op-amp output impedance and slew behavior,
- switching-stage input behavior,
- electrode resistance/capacitance,
- lead capacitance,
- output decoupling,
- protection/latch circuitry,
- any H-bridge recirculation paths.

So the topology may indeed be stable, but the exact margin is not demonstrated by the prose alone.

**Suggested fix**  
Before hardware freeze:

1. run a SPICE model across worst-case electrode R/C and cable capacitance,
2. bench-test step response into resistor, RC, open-circuit, and intermittent-contact loads,
3. verify symmetry of positive/negative phases at the maximum pulse width and current,
4. document the actual compensation parts selected from test data.

The design concept is likely salvageable; the current justification is just too hand-wavy.

---

### F7 — Safety shutdown path needs one unambiguous hardware “off” state
**Severity:** significant

**Issue**  
The documents mention a heartbeat watchdog latch, comparator aborts, USB interlock, and firmware charge accounting, which is all good. But the design still needs one plainly defined hardware truth:

> On any fault, exactly what loses power, what goes Hi-Z, and what cannot remain latched on?

**Why it matters**  
In stimulators, ambiguous shutdown behavior causes ugly failures: stale DAC code, half-enabled bridge, powered logic with unpowered analog domain, or output clamping through body diodes.

**Suggested fix**  
Define a fault tree around one hardware-off action, for example:

- remove power to the patient-domain output stage, and/or
- assert bridge-disable to a guaranteed Hi-Z state, and
- power down the DAC/output driver with the same heartbeat-qualified path.

Then bench-test these cases:

- MCU reset mid-pulse,
- USB insertion during idle and during armed state,
- battery brownout,
- broken heartbeat line,
- stuck bridge input.

The concept is right; the explicit shutdown implementation still needs to be nailed down.

---

### F8 — Power-chain details are mostly right, but a few implementation details still need closure
**Severity:** minor

**Issue**  
The LiPo -> boost -> isolated DC-DC -> ±15 V path is the correct macro-architecture, but a few practical points remain easy to miss.

**Why it matters / what to check**  
1. **TLV704 thermal load:** probably fine if isolated-side 5 V current stays modest, but calculate it from the actual VDD2 currents of the isolators plus DAC/reference load. Do not assume “150 mA available” means “thermally comfortable” in SOT-23.
2. **Battery protection:** low-battery firmware lockout is not a substitute for proper cell protection. Use a protected cell or add pack protection if the battery is bare.
3. **TP4056 + USB-C:** if the connector is USB-C, remember the **CC pull-down resistors and ESD details**; TP4056 does not make a USB-C sink implementation by itself.
4. **Charge/load behavior while plugged in:** even with stimulation disabled, make sure system load does not break charger termination assumptions.

**Suggested fix**  
Add these as explicit schematic checklist items before release.

---

## Direct answer to the DRV8871 contradiction
Yes — **the 6.5 V minimum VM is a real problem**, and in my opinion it is not the only problem.

If the intended architecture really places the op-amp servo output onto **DRV8871 VM**, then the part is not suitable there. The clean fix is **not** “hope it works anyway”; the clean fix is to **change the current-path architecture** so the H-bridge sits on a valid fixed supply and only handles polarity switching, while a dedicated analog element regulates current.

So my verdict is:

- **DRV8871 may still be usable in the project**,
- but **not in the exact role currently described**.

---

## Recommended next actions before schematic freeze

1. Redraw the output stage around a **fixed-supply H-bridge + separate current regulator**.
2. Replace “IPROPI impedance detection” with a real **voltage + current** contact-detection method.
3. Reclassify the PPTC as **secondary** protection only.
4. Downgrade the compliance language from “meets Type BF” to “designed toward BF-style isolation” until tested.
5. Revisit the series output capacitor choice with a stronger bias-derating / leakage argument.
6. Do a dedicated schematic review once KiCad files exist.

## Final verdict
This is **not a bad design**. In fact, a lot of the hard system-level choices are already better than what I usually see in DIY stimulators.

But it is **not yet electrically closed**. The output stage still has a real architecture bug, and the sensing/safety story is currently stronger in intent than in circuit evidence. Fix those, and the rest of the design looks very workable.
