# Electrical Design Review — OpenBinaural-taVNS (Second Pass)

**Date:** 2026-04-24  
**Reviewer:** Kai (EE second-pass review)  
**Scope:** `AGENTS.md`, `.planning/DESIGN-REVIEW.md`, `.planning/CIRCUIT-DESIGN-REVIEW.md`, plus supporting architecture/requirements material where relevant.

## Executive summary

I agree with most of the earlier findings. The biggest blocker is still the output stage: **the DRV8871-as-drawn is not electrically valid in this topology**. I also think two things are still understated:

1. **The proposed discrete MOSFET fallback is not a drop-in fix** if the bridge supply node is still the op-amp servo node.  
2. **The 50 ms heartbeat shutdown is too slow to protect against a crashed/stuck monophasic pulse.**

That second point is the most important new issue I found.

---

## 1) Assessment of the 8 known issues

### 1. DRV8871 minimum VM = 6.5 V
**Agree — critical blocker.**

Both prior reviewers are right. If `VM = op-amp output`, the bridge spends much of the intended operating range below UVLO. I would go a step further: **even aside from UVLO, VM is the wrong place to hang an analog servo node.** A motor-driver supply pin wants a stiff, decoupled rail, not a precision-controlled compliance node.

**My take:** this is slightly **understated**. It is not just a part-selection issue; it is an **architecture issue**.

### 2. No SPI MISO reverse channel
**Agree — but broader than that.**

Yes, if you use a patient-side ADC (e.g. MCP3201), you need a reverse path. But the deeper issue is that **impedance/contact detection needs a clearly defined measurement architecture**, not just “one more isolator channel.”

**My take:** correct finding, a little **understated**.

### 3. Vref strategy undecided
**Agree — medium severity.**

The design needs one explicit reference strategy before breadboarding. A divider from 5V_ISO may be acceptable for early bench work, but for a stimulation device I would not call it “closed” until drift/noise/current-accuracy are measured. A real reference is cleaner engineering if current accuracy is meant to be trusted.

**My take:** correctly flagged, about the right severity.

### 4. PPTC fuse used like primary overcurrent protection
**Agree — strongly.**

Thermal resettable fuses are backup energy limiters, not precise patient current limiters. Primary protection still needs a **fast analog cutoff path**.

**My take:** correctly flagged.

### 5. DC-blocking capacitor: MLCC X7R vs film
**Agree with the concern.**

I do not think the MLCC argument is fully convincing yet. X7R might work, but this is a patient-coupling capacitor in a pulsed-current stimulator; that is where I would rather be conservative, not optimized for size first.

**My take:** concern is real. The “MLCC is fine” language in `DESIGN-REVIEW.md` feels **too confident**.

### 6. Emergency stop is a small tactile button
**Agree.**

This is less about circuit physics and more about actual failure use. A bedside panic stop should be large, obvious, and easy to hit blindly.

**My take:** correctly flagged.

### 7. Type BF / 1 MOPP language ahead of evidence
**Agree.**

The architecture is aiming in the right direction, but the wording is ahead of validation. Without schematic/layout/leakage testing, I would describe it as **designed toward BF-style isolation**, not compliant.

**My take:** correctly flagged.

### 8. Loop-stability confidence too high
**Agree.**

The prose analysis is not enough to justify quoted phase margin numbers. Also, because the present H-bridge implementation is not electrically valid, **the stability argument is partly built on a stage that should not be frozen yet**.

**My take:** correctly flagged, maybe slightly **understated**.

---

## 2) New findings

### New Finding A — Heartbeat shutdown is too slow for a stuck monophasic fault
**Severity:** critical  
**This is the most valuable new issue I found.**

The documents describe a patient-side heartbeat watchdog with about **50 ms** timeout before analog power is removed.

That is fine for “MCU died, eventually shut the analog down,” but it is **not fine for pulse-by-pulse charge safety**.

If the MCU crashes or the bridge control freezes **mid positive phase** with DAC nonzero, the output can remain effectively monophasic until:

- the DC-block capacitor charges toward compliance, or
- the 50 ms heartbeat timeout kills analog power.

At 5 mA with a 10 µF series capacitor:

- capacitor charging rate is `dV/dt = I/C = 5mA / 10µF = 500 V/s`
- to reach ~13 V compliance takes about **26 ms**
- delivered fault charge before compliance is roughly `Q = C × V ≈ 10µF × 13V = 130 µC`

That is **orders of magnitude above** the intended 1 µC normal phase charge.

Even if the exact number shifts with load and supply, the basic conclusion does not:  
**the series DC-block cap does not protect against a long unintended monophasic pulse quickly enough.**

#### Why this matters

The current overcurrent comparator will not trip if the fault current is still “only” 5 mA. So this is a real hole:

- normal-current
- wrong-duration
- wrong-polarity
- large net charge

#### What I would do

Add a **hardware pulse-duration limiter / output timeout** much faster than 50 ms. Examples:

- monostable that forces bridge disable if a phase command persists beyond a few hundred microseconds / low milliseconds
- hardware that requires complementary phase completion to re-arm the next pulse
- fail-safe output disable that defaults to Hi-Z unless actively retriggered every pulse

If I were breadboarding next week, **this would worry me almost as much as the DRV8871 issue**.

---

### New Finding B — The “just use discrete MOSFETs” fallback is not actually solved in the present topology
**Severity:** significant

The earlier review proposes BSS84/BSS138 as an alternative H-bridge. That may be workable in some architecture, but **not as a drop-in replacement if the op-amp output is still acting as the bridge supply/compliance node**.

Why:

- the high-side PMOS source would ride on a node that can move from near 0 V up to ~13 V
- a fixed 0/5 V logic signal from the isolated digital domain does **not** correctly drive that gate over the full range
- once the source rises above logic rail, gate-drive headroom and turn-off behavior become ambiguous

So the discrete bridge suggestion is **understated**. It is not “swap in four MOSFETs and you’re done.” It needs:

- a revised fixed-supply current-path architecture, **or**
- proper high-side level shifting / gate drive referenced to the moving source node

My conclusion:  
**the real fix is to redesign the current-path architecture first, then choose bridge implementation second.**

---

### New Finding C — Safety-critical numbers are not fully locked across documents
**Severity:** moderate

I found meaningful cross-document inconsistencies, including:

- 100 Ω sense resistor in some places vs **10 Ω / 60 mV trip** language elsewhere
- different op-amp histories (OPA388 / OPA2277 / ADA4522)
- different H-bridge stories (DRV8837 / DRV8871 / discrete)

That is understandable in an evolving design, but it means:

- comparator thresholds
- ADC scaling
- compliance calculations
- pulse/fault math

are not yet anchored to one single final electrical design.

For a stimulator, that matters. Before breadboarding, I would freeze one canonical signal chain and recompute **every threshold** from that version only.

---

## 3) What would worry me most on a breadboard next week?

### 1. The output-stage architecture itself
I would not trust the current bridge story on a breadboard until the **current regulator + polarity switch partition** is redrawn cleanly.

### 2. Fault behavior on a crashed/stuck pulse
I would want to bench-test this early:

- force MCU reset mid-phase
- force one bridge command stuck high
- freeze DAC nonzero
- watch delivered load current and cap voltage

This is the failure mode I most want scoped.

### 3. Proving contact detection with a real measurement path
I would not rely on inferred impedance from current-only mechanisms. I would want a known test current and a real patient-side voltage measurement path.

### 4. Verifying the coupling capacitor choice with actual waveforms
If MLCC stays in the design, I would immediately measure:

- effective capacitance at bias
- pulse droop
- long-session residual offset
- fault-charge behavior in stuck-output scenarios

---

## Bottom line

The system direction is good, but the design is **not electrically closed yet**.

My final read:

- I **agree** with the prior reviewers on the major known issues.
- The **DRV8871 problem is real and decisive**.
- The most important thing I think was still missed is this:

> **A 50 ms heartbeat watchdog is far too slow to protect against a stuck monophasic output; the DC-block capacitor can still allow a very large unintended fault charge before shutdown.**

If this were my bench next week, I would redesign the output stage and the fault-off behavior **before** trusting any human-contact testing.
