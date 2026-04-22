Agent completed. agent_id: tavns-clinical-review, agent_type: general-purpose, status: completed, description: Clinical and device completeness review, elapsed: 337s, total_turns: 0, model: claude-opus-4.6, duration: 331s

Now I have a thorough understanding of the project's research, design, and current state. Let me compile the full review.

---

# OpenBinaural-taVNS — Clinical & Hardware Completeness Review

## PART 1: CLINICAL PARAMETER REVIEW

---

### 1. Preset Parameter Validation

Your existing 6 presets are **well-researched and largely correct**. The corrections already applied (anti-inflammatory 10→25Hz, RESET-AF unilateral left, dysautonomia 0.5mA ramp) are the right calls. Specific notes per preset:

| Preset | Verdict | Notes |
|--------|---------|-------|
| **Insomnia 25Hz** | ✅ Correct | 25Hz/200µs matches NEMOS/Nurosym defaults. 2.0mA is a reasonable starting point but emphasize threshold titration in UI — most literature says "titrate to perception threshold," not fixed current. |
| **Insomnia 20Hz** | ✅ Correct | 20Hz matches Chinese RCTs (Jiao, Li, Fang). Good A/B test against 25Hz. |
| **RESET-AF** | ✅ Correct (post-correction) | Unilateral left, continuous, 60min — matches Stavrakis protocol. **One flag:** RESET-AF/TREAT-AF used **tragus**, not cymba conchae. Your device targets cymba conchae (~100% ABVN vs tragus ~45%). This is potentially *better* but is a deviation from the trial protocol. The UI already notes this — good. |
| **Anti-Inflammatory** | ✅ Correct (post-correction) | 25Hz/250µs matches Lerman 2016. **Minor note:** Lerman used cymba conchae at sub-threshold intensity — your 1.5mA default is reasonable but verify whether it should say "sub-threshold" rather than a fixed value. |
| **Dysautonomia/hEDS** | ✅ Acceptable as theoretical | 0.5→1.0mA ramp is prudent for hEDS sensory hypersensitivity. The 45s ramp (from PRESETS.md §4.4) is a good adjustment. **Add:** first-session tolerance test (15min only) should be enforced in firmware, not just recommended. |
| **Exploration** | See §3 below for safety limits | |

**One parameter I'd reconsider across all presets:** Your default current values (2.0mA for insomnia, 1.5mA for anti-inflammatory, etc.) are fixed values. Every clinical taVNS study uses **perception threshold titration**, not fixed current. The firmware should implement a **threshold-finding protocol** — slow automatic ramp with a "I feel it" button press — rather than starting at a preset mA value. Fixed defaults are fine as starting points, but the UI flow should guide titration.

**Charge balance verification:** At your worst-case parameters (5mA × 500µs = 2.5µC/phase, electrode area ~0.5cm² = 5.0µC/cm²), you're at 16.7% of McCreery's 30µC/cm² limit. That's a healthy margin. At clinical sweet-spot (2mA × 200µs = 0.4µC/phase = 0.8µC/cm²), you're at 2.7% — excellent.

---

### 2. Missing Presets

Your PRESETS-EXPANSION.md already identified the major gaps. The recommended final menu of 11 presets (6 well-evidenced + 3 moderate + 1 experimental + Exploration) is a good selection. I agree with the specific additions and omissions. Here is my assessment of what to add vs. skip:

**ADD (agree with PRESETS-EXPANSION.md):**

| Preset | Evidence | Why Include |
|--------|----------|-------------|
| **Depression (MDD)** | RCT, N=160 (Fang/Rong 2016) | Strongest new evidence after RESET-AF. 20Hz/200µs/1.0mA. Unilateral left. *Note: adjunctive only; 2×/day sessions.* |
| **Epilepsy (Adjunctive)** | CE-marked (NEMOS); Bauer 2016 RCT | Original taVNS indication. 25Hz/250µs. Long daily exposure (3×60min). Unilateral left. |
| **Stress/Acute Anxiety** | Pilot RCTs (Burger 2019, Szeska 2020) | Short 15-min sessions. 25Hz/200µs/continuous. Good biohacker use case. |
| **HRV Optimization** | Multiple converging pilots | **Best preset for bilateral stagger hypothesis testing.** 15min, low risk, measurable outcome via Polar H10. This is strategically the most important new addition for the project's research goal. |
| **Migraine Prevention** | Straube 2015 RCT | **1Hz is a unique and interesting parameter.** Unreplicated but worth including as an option. Firmware must handle 1Hz correctly (user will feel discrete individual pulses). |

**SKIP (agree with PRESETS-EXPANSION.md):**

| Condition | Why Skip |
|-----------|----------|
| Tinnitus | Weak evidence; interesting application (VNS-paired tones) requires audiometric calibration beyond a simple preset |
| Hypertension | No dedicated trial; parameters overlap entirely with RESET-AF |
| Obesity | Evidence too thin; handle via Exploration |
| Chronic Pain (separate) | Overlaps with Anti-Inflammatory; add a note rather than a separate preset |
| PTSD | One tiny pilot; use Stress preset instead |
| Cognitive enhancement | No clinical evidence in healthy subjects |
| Post-COVID dysautonomia | Zero publications; use Dysautonomia preset |

**One preset I'd consider that isn't in PRESETS-EXPANSION.md:**

**"Respiratory-Gated" mode** — Napadow 2012 (REF-037) found enhanced analgesic effects when taVNS was synchronized to the exhalation phase (when vagal tone is naturally highest). Your IDEAS.md already captures this (proposed by AI). If Polar H10 RR-interval data can derive respiratory phase (respiratory sinus arrhythmia), this could be a firmware feature rather than a separate preset — a toggle that gates any preset's stimulation to the exhale phase. This is a genuine differentiator no commercial device offers. Implementation: detect RR-interval lengthening (exhale) from Polar H10 stream, trigger stimulation bursts only during those periods.

---

### 3. Exploration Mode Safety Limits

These should be **non-overridable hardware+firmware enforced bounds**:

| Parameter | Hard Limit | Rationale |
|-----------|-----------|-----------|
| **Current** | ≤5.0mA (hardware cutoff at 6mA) | No taVNS protocol uses >5mA. Above this, Aδ/C pain fibers dominate. |
| **Frequency** | 1–100Hz | Below 1Hz is DC-like; above 100Hz risks nerve fatigue and pulse overlap. Warn >50Hz ("outside clinical evidence"). |
| **Pulse width** | 50–1000µs | Below 50µs is ineffective for nerve activation. Above 1000µs approaches tissue heating territory at higher currents. |
| **Charge per phase** | ≤25µC (I × PW) | 83% of McCreery 30µC/cm² limit at 0.5cm² electrode. This catches dangerous parameter combinations (e.g., 5mA × 1000µs = 5µC is fine; but if someone configures 5mA × 5000µs... blocked). |
| **Duty cycle** | ON ≤60s, OFF ≥0s (continuous allowed) | Allow continuous per RESET-AF/Stress protocols. |
| **Session duration** | 1–120min | NEMOS allows 4h/day (as 3×80min); 120min single session is generous. |
| **Pulse overlap guard** | freq × PW × 2 < 1.0 | Prevents biphasic pulse overlap at high freq + wide PW combinations. |
| **No stimulation without electrode contact** | If impedance sensing is implemented: abort if Z > 10kΩ or Z < 100Ω | See §7 below. |

**Your REQUIREMENTS.md (SAFE-06) already specifies most of these correctly.** The one addition I'd make: a **per-session charge density running total** displayed in the BLE app. This gives researchers real-time visibility into their cumulative exposure — no commercial device does this.

---

### 4. The Binaural Stagger Hypothesis — 20ms Analysis

**Is 20ms physiologically reasonable?** Yes, with caveats.

**Neuroanatomical timing:** Auricular vagal afferents (ABVN) project to the ipsilateral nucleus tractus solitarius (NTS). Conduction velocity in Aβ myelinated afferents ≈ 30–70 m/s. Ear to brainstem distance ≈ 15cm. Transit time ≈ 2–5ms. So bilateral pulses fired 20ms apart will arrive at left and right NTS with ~20ms temporal separation — each NTS will process its ipsilateral pulse in relative isolation before the contralateral pulse's effects propagate through commissural fibers.

**NTS physiology:** NTS neurons have firing rates of 5–50Hz and refractory periods of ~5–20ms. A 20ms stagger means the second pulse arrives after the refractory period of NTS neurons activated by the first pulse. This is the right ballpark to avoid saturation.

**Published literature on bilateral auricular stimulation:** Essentially zero. Nurosym offers bilateral stimulation but appears to fire synchronously. No published study compares synchronous vs. staggered bilateral taVNS. Your project's stagger is genuinely novel.

**My assessment of stagger timing:**
- **20ms (default)** = half-period at 25Hz. Reasonable — maximally separates left/right NTS input within each cycle.
- **0ms (synchronous)** = replicates commercial bilateral devices. Good control condition.
- **Configurable 0–40ms** allows exploration. At 25Hz (40ms period), offsets >20ms mean Channel B fires closer to Channel A's *next* pulse than the current one — probably diminishing returns.

**The theoretical basis is sound but speculative.** The key question — "does temporal separation of bilateral vagal afferent input produce greater parasympathetic enhancement than simultaneous input?" — can only be answered experimentally, which is exactly what this project enables via the Polar H10 HRV comparison. The HRV Optimization preset is the ideal test bed.

**Recommendation:** Keep 20ms default. The configurable range (0–40ms) is correct. Document this as the project's primary experimental hypothesis. The first experiment to run: 5 sessions unilateral left → 5 sessions bilateral synchronous → 5 sessions bilateral 20ms stagger, measuring RMSSD change. N=1 but with intra-subject controls.

---

### 5. Electrode Placement by Indication

**Cymba conchae is the correct default target.** Per Peuker & Filler 2002: ~100% ABVN innervation in cymba conchae vs ~45% in tragus vs ~0% at earlobe (sham). Yakunina 2017 fMRI confirmed: cymba conchae stimulation produced strongest NTS/brainstem activation.

| Preset | Site in Published Studies | Your Target | Assessment |
|--------|--------------------------|-------------|------------|
| Insomnia 25Hz | Mixed (tragus + cymba) | Cymba conchae | ✅ Likely superior to tragus |
| Insomnia 20Hz | Mostly cymba (Chinese) | Cymba conchae | ✅ Matches literature |
| RESET-AF | **Tragus** (Stavrakis) | Cymba conchae | ⚠️ **Divergence from trial** — potentially better due to higher ABVN density, but UI must document this |
| Anti-Inflammatory | Cymba (Lerman 2016) | Cymba conchae | ✅ Matches |
| Depression | Cymba (Fang/Rong 2016) | Cymba conchae | ✅ Matches |
| Epilepsy | Cymba (NEMOS device) | Cymba conchae | ✅ Matches |
| Stress/Anxiety | Mixed (cymba + tragus) | Cymba conchae | ✅ Consistent |
| HRV Optimization | Mixed (tragus in Bretherton/Clancy) | Cymba conchae | ✅ Likely superior |
| Migraine | Cymba (Straube 2015, NEMOS) | Cymba conchae | ✅ Matches |
| Dysautonomia | No data | Cymba conchae | ✅ Best default |

**Bottom line:** Cymba conchae is the right call for every preset. The only caveat is RESET-AF (tragus in original trials), which is already documented. No preset should specify tragus stimulation — if users want to try tragus, they can use Exploration mode with appropriate documentation.

**Anatomical note for documentation:** The cymba conchae is the upper cavity of the concha, above the crus of the helix. It is NOT the cavum conchae (lower cavity) or the tragus. Include a labeled photo/diagram in the build guide. Electrode misplacement is one of the most common causes of treatment failure in self-administered taVNS.

---

### 6. Session Duration and Frequency — Safe Usage Limits

**Published usage patterns:**

| Protocol | Sessions/Day | Duration | Total Daily Exposure | Duration (weeks) |
|----------|-------------|----------|---------------------|-------------------|
| NEMOS (epilepsy) | 3–4 | 60–80min each | **Up to 4h/day** | Months to years |
| RESET-AF | 1 | 60min | 60min/day | 6 months |
| Fang 2016 (depression) | 2 | 30min each | 60min/day | 4 weeks |
| Most insomnia RCTs | 1 | 30min | 30min/day | 4–8 weeks |
| Bretherton 2019 (elderly) | 1 | 15min | 15min/day | 2 weeks |

**Safe limits to recommend:**

| Guideline | Value | Basis |
|-----------|-------|-------|
| **Max sessions/day** | 3 (firmware warning, not hard block) | NEMOS protocol upper limit |
| **Max total daily exposure** | 4 hours | NEMOS CE-marked protocol |
| **Min inter-session rest** | 30min | No published guideline, but reasonable to prevent adaptation |
| **Session frequency** | Daily use is safe | All RCTs used daily protocols |
| **Long-term safety** | Months of daily use appears safe | RESET-AF: 6 months daily. NEMOS: years. No serious adverse events reported. |
| **Rest days** | Not required but reasonable | No data on "7 days/week vs 5 days/week" |

**Adverse events in the literature:** The most commonly reported side effects across all published taVNS studies are:
1. Tingling/prickling at electrode site (expected, not adverse)
2. Mild skin redness at electrode site (resolves within hours)
3. Mild headache (~5% of subjects, usually first 1–2 sessions)
4. Ear discomfort from electrode pressure

**No serious adverse events (SAEs)** have been reported in any published taVNS RCT. No cases of cardiac arrhythmia induction, syncope, or neurological injury attributable to taVNS. This is a good safety profile.

**Firmware recommendation:** Track cumulative daily exposure. After 60min total daily (for non-epilepsy presets) or 240min (for epilepsy), display a warning: "Daily exposure limit reached. Continue? Y/N" — advisory, not a hard block, since the user may be intentionally following a longer protocol.

---

## PART 2: HARDWARE FEATURE COMPLETENESS vs COMMERCIAL DEVICES

---

### 7. Electrode Impedance Measurement

**Do commercial devices measure impedance?** Yes — this is standard.

| Device | Impedance Feature |
|--------|-------------------|
| NEMOS (Cerbomed) | Pre-session impedance check. Won't start if out of range. |
| Nurosym | Contact quality indicator (LED/app) |
| Research-grade devices | Full impedance spectroscopy (some) |
| gammaCore (cervical) | Contact quality check |

**Your DRV8871 IPROPI output is the right approach for impedance sensing.** Here's how to use it:

**Pre-session impedance check:**
1. Set DAC to a small known current (100µA — well below perception threshold)
2. Measure voltage at IPROPI output (proportional to motor current): IPROPI = I_out × 1375µA/A for DRV8871
3. Actually, more directly: since your V-to-I converter servos current, you know I_load. Measure the op-amp output voltage (V_compliance = I_load × Z_electrode + V_Rsense). Read V_compliance via an ADC pin on the isolated side.
4. Z_electrode = (V_compliance - V_Rsense) / I_test = (V_compliance - 10mV) / 100µA

**Impedance thresholds (from your FEATURES.md TS-6, which are correct):**

| Reading | Meaning | Action |
|---------|---------|--------|
| <100Ω | Short circuit or metallic contact | ABORT — electrode not on skin |
| 100–500Ω | Very low — possible short or wet gel excess | WARN |
| 500Ω–5kΩ | **Normal range for conductive gel + skin** | GOOD — proceed |
| 5kΩ–10kΩ | High impedance — poor contact | WARN — reposition electrode |
| >10kΩ | No contact or completely dry | ABORT — electrode not placed |

**During-session monitoring:** Monitor compliance voltage continuously. If the op-amp output hits the rail (≈±13.5V for ±15V supply), the current source has lost regulation — impedance is too high. Auto-pause and alert user to check electrode position. This is already in your requirements (FW-06) — implement it.

**Hardware needed:** One ADC channel on the isolated side (can be an ADC on the ESP32 if you route the signal back through an isolator, or a simple comparator for over-limit detection). The simplest implementation: use the LM393 dual comparator that's already there for overcurrent — add a voltage divider from the op-amp output to a comparator input with a reference set at ~12V (compliance limit threshold). This gives you a digital "compliance exceeded" signal without needing an ADC on the isolated side.

---

### 8. Skin Contact Detection

**How do commercial devices verify electrode contact?** Impedance-based — same system as §7.

**Minimum implementation:**
1. **Pre-session check:** Inject 100µA test pulse. If voltage > 10V (Z > 100kΩ), no electrode detected — refuse to start.
2. **During session:** If compliance voltage hits rail for >500ms continuously, auto-pause.
3. **Electrode removal detection:** If impedance suddenly jumps from normal range to >10kΩ mid-session, that's an electrode falling off. Auto-pause within 100ms.

**The DRV8871 IPROPI output can serve as the contact detection sensor.** During stimulation, IPROPI should track the commanded current. If IPROPI drops to zero while the DAC is commanding current, the electrode has lost contact. This can be monitored by a simple comparator: "if DAC > 0 AND IPROPI < threshold, then electrode disconnected."

**Your requirements already include this (FW-06). The open question was "how to implement without additional hardware."** Answer: IPROPI on the DRV8871 is sufficient. Route it to a comparator or ADC. No additional sensors needed.

---

### 9. Sensory Threshold Detection Protocol

**Is this relevant?** Yes — it's how EVERY clinical taVNS study sets intensity.

**The NEMOS protocol:**
1. Start at 0mA
2. Slowly increase current (ramp)
3. Patient reports "I feel tingling" — this is the **perception threshold**
4. Continue increasing until "it's uncomfortable" — this is the **pain threshold**
5. Set treatment current to just below pain threshold (typically 70–80% of the way from perception to pain)

**Should you support this?** Yes, but **not automatic detection** — your REQUIREMENTS.md correctly identifies auto-detection as an anti-feature (AF-7). Manual titration is the standard.

**Firmware implementation:** A "threshold-finding mode" in the BLE app:
1. Start at 0mA with slow auto-ramp (0.1mA/second)
2. User taps "I feel it" → record perception threshold
3. Continue ramping
4. User taps "Uncomfortable" → record pain threshold
5. Auto-set treatment current to perception threshold + 60% × (pain - perception)
6. Allow manual fine-tuning ±0.1mA

This is a firmware/app feature, not hardware. Your current hardware (0.1mA resolution, soft-start ramp) already supports it. **Add a threshold-finding UI flow to Phase 6 (BLE Interface) requirements.**

---

### 10. Missing Hardware Features vs Commercial Devices

| Feature | NEMOS | Nurosym | Your Device | Assessment |
|---------|-------|---------|-------------|------------|
| **Constant current output** | ✅ | ✅ | ✅ | Match |
| **Biphasic charge-balanced** | ✅ | ✅ | ✅ | Match |
| **Impedance check** | ✅ | ✅ | Planned (FW-06) | **Must implement** |
| **Contact detection** | ✅ | ✅ | Planned (FW-06) | **Must implement** |
| **Session presets** | ✅ (locked) | ✅ (limited) | ✅ (more + configurable) | **Better** |
| **Intensity control** | Limited (↑/↓ buttons) | App control | BLE app control | **Match/Better** |
| **Session timer** | ✅ | ✅ | ✅ (planned) | Match |
| **Battery indicator** | ✅ | ✅ App | Need to add BLE characteristic | **Add** — read LiPo voltage via ADC, expose via BLE |
| **LED indicators** | Status LED | Status LED | Planned (1 LED) | Match |
| **Vibration/haptic feedback** | ❌ | ❌ | ❌ | N/A — neither has it. **Skip.** |
| **Audio tones** | Session start/end beep (some) | ❌ | ❌ | **Consider adding** — a piezo buzzer ($0.10) for session start/end/fault alert. Users stimulating before sleep may have eyes closed. Simple GPIO-driven beep. |
| **Temperature sensing** | ❌ | ❌ | ❌ | **Not needed.** At <5mA and <1% duty within each pulse, electrode heating is negligible. No commercial taVNS device includes this. Relevant for high-current TENS/FES, not taVNS. |
| **EMG/ECG during stimulation** | ❌ | ❌ | ❌ | **Skip for v1.** Your Polar H10 integration covers HRV. On-device ECG would require a dedicated AFE (ADS1292) and complicate the isolated analog design. |
| **Real-time HRV feedback** | ❌ | ❌ | ✅ (Polar H10 planned) | **Genuine advantage** — see §14 |
| **Session logging** | Basic counter | App-logged | ✅ (LittleFS + CSV) | **Better** |
| **Display** | None (LED only) | Phone app | Phone app (BLE) | Match |
| **OTA updates** | ❌ | ❌ | ✅ (planned) | **Better** |
| **Bilateral** | ❌ | ✅ (synchronous) | ✅ (staggered, configurable) | **Better** |
| **Open data export** | ❌ | ❌ | ✅ (CSV/JSON) | **Better** |
| **Charge density display** | ❌ | ❌ | ✅ (planned) | **Better** |

**Hardware additions to consider before PCB fabrication:**

1. **Battery voltage ADC** — Route LiPo voltage through a resistor divider to an ESP32 ADC pin. Expose battery % via BLE. Essential for a battery-powered wearable. **Cost: $0.01 (2 resistors). Priority: HIGH.**

2. **Piezo buzzer** — Session start/end/fault audio alert. User may be eyes-closed pre-sleep. **Cost: $0.10 (SMD piezo, GPIO-driven). Priority: MEDIUM.**

3. **Second status LED or RGB LED** — One LED with 3 states (green/blue/red) is minimum. An RGB LED allows more granular feedback (e.g., yellow = impedance warning, pulsing blue = stimulating, solid green = ready). **Cost: $0.05. Priority: LOW** (phone app provides rich UI).

4. **Accelerometer (LIS2DH12, $0.50)** — Detect if device is removed or user has fallen (safety). Also enables motion-artifact rejection for impedance sensing. **Priority: LOW** — adds complexity, marginal benefit for ear-worn device.

---

### 11. Electrode Design

**What do published trials use?**

| Study/Device | Electrode Type | Material | Contact Area |
|-------------|---------------|----------|-------------|
| NEMOS (Cerbomed) | Custom titanium spring electrodes, ear-clip | Titanium (dry contact) | ~0.5cm² per contact |
| Nurosym | Stainless steel ear clip | Stainless steel | ~0.5–1.0cm² |
| RESET-AF (Parasym device) | Ear clip with conductive rubber pad | Conductive rubber + gel | ~0.5cm² |
| Chinese RCTs | Various ear clips, often with conductive gel | Silver/AgCl or stainless steel + gel | ~0.3–1.0cm² |
| Research custom | AgCl pellet electrodes with Ten20 paste | Ag/AgCl | ~0.2–0.5cm² |

**Recommendation for your device:**

**Electrode material:** **Ag/AgCl (silver/silver chloride)** is the gold standard for bioelectric interfaces. It has:
- Low and stable electrode-skin impedance
- Minimal polarization potential (non-polarizable electrode)
- Excellent charge transfer characteristics
- Used in ECG, EEG, and most clinical neurostimulation

**Alternative:** Medical-grade stainless steel (316L) with conductive hydrogel. Simpler to source, lower cost, adequate for taVNS currents.

**Contact design for cymba conchae:**
- The cymba conchae is a concave surface inside the ear. A spring-loaded clip is impractical here (works for tragus/earlobe).
- **Best approach:** Custom-molded silicone ear insert (like an earplug) with embedded electrode contacts positioned at the cymba conchae. Or a flexible PCB with snap-on hydrogel pads that conforms to the concha.
- **DIY-friendly approach:** Modified ear clip with a gel-coated electrode pad bent to contact the cymba conchae from above. Or a 3D-printed ear scaffold that positions electrodes.
- **Published approach (NEMOS):** Titanium ball electrodes on a spring arm that press into the cymba conchae from a behind-ear housing.

**Charge density and electrode geometry:**
Your McCreery calculation assumes electrode area. **This is the weakest link in your safety math.** If a user makes a small electrode (0.1cm² instead of 0.5cm²), the charge density at 5mA × 500µs = 2.5µC/phase becomes 25µC/cm² — dangerously close to the 30µC/cm² limit.

**Mitigation:** The configurable `electrode_area` parameter in FW-07 is critical. But for open-source DIY, you can't control electrode geometry. **Recommendation:**
1. Ship a recommended electrode design (BOM + 3D print files) with known contact area (~0.5cm²)
2. Set a conservative default electrode area in firmware (0.5cm²)
3. Warn if user sets electrode area <0.3cm²
4. Hard-block if charge density exceeds 24µC/cm² (80% of McCreery limit) regardless of electrode area setting

**Include in build guide:** "Do NOT use electrodes with contact area smaller than 0.3cm². Smaller electrodes concentrate charge and risk tissue damage."

---

### 12. Session Logging — What to Record

**Your planned logging (DATA-01) is already good. Here's the complete recommended dataset, compared to commercial:**

| Data Field | NEMOS | Nurosym | Your Device | Value for Self-Experimentation |
|-----------|-------|---------|-------------|-------------------------------|
| Timestamp (start/end) | ✅ | ✅ | ✅ Planned | Essential |
| Preset name | N/A (one mode) | ✅ | ✅ Planned | Essential |
| All parameters (freq, PW, current, duty, stagger) | ❌ (fixed) | Limited | ✅ Planned | **Critical for research** |
| Impedance at start (L/R) | ❌ | ❌ | ✅ Planned | Contact quality tracking |
| Impedance at end (L/R) | ❌ | ❌ | ✅ Planned | Gel drying / drift tracking |
| Impedance samples during session | ❌ | ❌ | **Add** | One reading per minute — tracks contact quality over time |
| Fault events (overcurrent, impedance, pause) | ❌ | ❌ | ✅ Planned | Debugging + safety |
| HRV (RMSSD, pNN50, HF, LF/HF, SDNN) | ❌ | ❌ | ✅ Planned (Polar H10) | **Primary outcome measure** |
| Raw RR intervals | ❌ | ❌ | **Add** | Enables post-session reanalysis with different algorithms |
| Battery voltage at start/end | ❌ | ❌ | **Add** | Detect if low battery affected output |
| Compliance voltage events | ❌ | ❌ | **Add** | Tracks when current source was in/out of regulation |
| Subjective intensity rating (1–10) | ❌ | ❌ | **Add via app prompt** | Correlates perceived intensity with parameters |
| Session number (cumulative counter) | ✅ | ✅ | **Add** | Dose tracking |
| Firmware version | N/A | N/A | **Add** | Reproducibility |

**Logging format:** CSV is correct for interoperability. JSON as a secondary option for structured metadata. Each session = one CSV row (summary) + one file of time-series data (RR intervals, impedance samples).

**The most valuable data for self-experimentation analysis:**
1. **RMSSD change (pre vs during vs post)** — primary vagal tone biomarker
2. **Parameter set used** — enables within-subject comparison across presets
3. **Electrode impedance trend** — quality control
4. **Cumulative session count** — dose-response over weeks

---

## PART 3: WHAT'S MISSING

---

### 13. Open Safety Gaps

**⚠️ SAFETY-CRITICAL items not yet fully addressed:**

| Gap | Severity | Current Status | Recommendation |
|-----|----------|----------------|----------------|
| **No electrode contact detection** | HIGH | Planned (FW-06) but hardware path unclear | **Must be implemented before any human use.** Use DRV8871 IPROPI + comparator. This is the #1 missing safety feature. |
| **No low-battery shutoff** | MEDIUM | Not mentioned | If LiPo drops below 3.0V, boost converter output becomes unstable → unpredictable ±15V rail voltage → potential current source malfunction. **Add LiPo under-voltage lockout:** if VBAT < 3.2V, refuse to start session. If VBAT drops below 3.0V during session, ramp down and stop. |
| **No thermal monitoring on H-bridge** | LOW | Not addressed | DRV8871 has thermal shutdown at 150°C but no user warning before that. At taVNS currents (<5mA), thermal issues are unlikely. **Skip for v1** — power dissipation at 5mA is negligible. |
| **Watchdog crosses isolation barrier** | HIGH | Planned (heartbeat via isolator) | **Verify that the heartbeat/comparator latch circuit works bidirectionally.** If the ESP32 crashes, the isolated-side comparator must cut power within 100ms. The LM393 latch on the isolated side should hold the output disabled until explicitly reset by a user action (power cycle). |
| **No "session-in-progress" lockout for OTA** | MEDIUM | Mentioned in BLE-04 | **Firmware must physically disable OTA DFU service while stimulation is active.** A firmware flash mid-stimulation could crash the stimulation ISR — potentially leaving H-bridge in an asymmetric state until the DC-blocking cap catches it. |
| **PPTC fuse recovery time** | LOW | PPTC fuses specified | SMD0603B001TF has 10mA hold current. At 5mA nominal, the fuse won't trip. It trips at overcurrent faults. After tripping, PPTC fuses take **seconds to minutes** to cool and reset. During recovery, the device won't stimulate — this is actually a safety feature (forces user to investigate). **No action needed** — behavior is correct. |
| **Electromagnetic compatibility (EMC)** | LOW | Not addressed | IEC 60601-1-2 requires EMC testing. Your device is low-power and battery-operated, so emissions are likely minimal. The 4MHz boost converter is the main radiator — the CM choke in your design addresses this. **For a DIY device, formal EMC testing is not expected, but document awareness.** |

**IEC 60601-1 Type BF checklist — items you address vs gaps:**

| 60601-1 Requirement | Your Design | Gap? |
|---------------------|-------------|------|
| Patient leakage current <100µA (Type BF) | Galvanic isolation 1500V + USB interlock | ✅ Addressed |
| Earth leakage current <500µA | Battery-powered, no earth connection | ✅ N/A |
| Insulation test (2× working + 1000V) | B0515D-1WR3 rated 1500VDC | ✅ Addressed |
| Single fault safe | Hardware overcurrent + watchdog + DC-block | ✅ Addressed |
| Essential performance (output accuracy) | V-to-I feedback loop + sense resistor | ✅ Addressed |
| Means of protection (2 independent) | 1: galvanic isolation. 2: DC-block + overcurrent | ✅ Addressed |
| Risk management (ISO 14971) | Informal (pitfalls doc) | ⚠️ Not formal |
| Biocompatibility (ISO 10993) | Electrode materials not specified in BOM | ⚠️ **Specify biocompatible electrode materials in build guide** |

---

### 14. Genuine Advantages Over Commercial Devices

| Advantage | vs NEMOS | vs Nurosym | How Significant |
|-----------|----------|------------|-----------------|
| **Open-source hardware + firmware** | ❌ Proprietary | ❌ Proprietary | **Massive** — reproducibility, auditability, community improvement |
| **Configurable parameters** | ❌ (locked 25Hz/250µs) | Limited (app) | **Major** — enables protocol replication from any published study |
| **Bilateral with stagger** | ❌ (unilateral) | Bilateral (synchronous only) | **Unique** — no commercial device offers pulse-level stagger |
| **Real-time HRV integration** | ❌ | ❌ | **Major** — turns faith-based stimulation into measured-response. No sub-$500 device has this. |
| **Charge density display** | ❌ | ❌ | **Unique** — researchers can verify safety margin in real-time |
| **Open data export** | ❌ | ❌ (locked in app) | **Major** — CSV/JSON for analysis with any tool (Python, R, Excel) |
| **Multiple evidence-based presets** | 1 (epilepsy) | ~2–3 | **Better** — 11 presets covering 7+ conditions with literature citations |
| **OTA firmware updates** | ❌ | Unknown | **Better** — living device that improves over time |
| **Per-channel current** | N/A | Single control | **Better** — compensates bilateral impedance asymmetry |
| **Cymba conchae targeting** | ✅ (same) | ✅ (same) | Match |
| **Cost** | ~€500–700 | ~£300–500 | **$30–60 BOM** — 10× cheaper |
| **Waveform verification (test point)** | ❌ | ❌ | **Unique** — community can verify charge balance on oscilloscope |
| **Respiratory-gated mode** (if implemented) | ❌ | ❌ | **Unique** — no commercial device synchronizes to breathing |

**Honest assessment:** This device is potentially **better than either NEMOS or Nurosym** for a technically capable user. The configurability, HRV integration, open data, and bilateral stagger are genuine innovations. The cost advantage is enormous. The main disadvantage is that it requires technical skill to build and is not a polished consumer product.

---

### 15. Red Flags

As a reviewer, here are my concerns if a patient were going to self-experiment with this device:

**🔴 RED FLAGS (must address before human use):**

1. **Electrode contact detection is not implemented.** Without this, the device will happily deliver current to open air (compliance voltage maxes out, then reconnection causes a current spike). This is the highest-priority pre-human-use feature. **Block Phase 1 TENS hack from human testing until impedance-based contact detection is working at minimum in a basic form.**

2. **No low-battery protection.** If LiPo drops below 3.0V during a session, the boost converter output becomes unreliable → ±15V rails sag → V-to-I converter may lose regulation → unpredictable output. Add under-voltage lockout.

3. **Electrode design is unspecified.** The most dangerous component in a DIY neurostimulation device is the electrode. A user could build tiny point electrodes (high charge density), use non-biocompatible metals (nickel → skin reactions), or make poor mechanical contact. **Ship a tested electrode design with the project — 3D print files, material spec, contact area measurement. Don't leave this to the builder.**

**🟡 CAUTIONS (should address, not blockers):**

4. **The "NOT A MEDICAL DEVICE" disclaimer is necessary but insufficient.** Users WILL use this for medical conditions (that's the point). The real protection is the safety architecture — which is solid. But add to documentation: "This device is for personal research only. It does not replace medical care. If you have a cardiac condition, epilepsy, or are pregnant, consult your physician before use."

5. **Preset labels could create false confidence.** A "Depression" preset implies clinical efficacy. Label presets honestly: "Depression — Adjunctive (RCT-based parameters)" with the evidence level visible. Don't oversell.

6. **The Dysautonomia/hEDS preset has zero evidence.** The theoretical rationale is sound, but the population is medically complex (MCAS, cardiac involvement, sensory hypersensitivity). The longer ramp, lower current, and first-session tolerance test are good mitigations, but emphasize in documentation that this is the most experimental preset.

7. **No formal adverse event reporting mechanism.** Commercial devices have post-market surveillance. An open-source project should have a GitHub issue template for "Adverse Event Report" — skin irritation, dizziness, headache, cardiac symptoms, etc. This data is valuable and shows responsible design.

**🟢 THINGS THAT ARE ACTUALLY GOOD (and reduce my concern):**

- The multi-layer safety architecture (galvanic isolation + DC-block + overcurrent comparator + watchdog + USB interlock) is defense-in-depth. Any single failure is caught by another layer. This is better than most DIY projects.
- The DC-blocking capacitors provide *unconditional* charge balance regardless of firmware state. This is the most important safety feature and it's hardware-enforced.
- The 12× safety margin on charge density (at clinical parameters) is generous.
- The hardware overcurrent cutoff at 6mA (LM393 latch) is independent of firmware — correct approach.
- The FreeRTOS core pinning (stimulation ISR on Core 1, BLE on Core 0) is the right architecture to prevent BLE from disrupting stimulation timing.

---

## SUMMARY: ACTION ITEMS BY PRIORITY

| Priority | Item | Category |
|----------|------|----------|
| **P0 — Before human use** | Implement electrode contact detection (impedance-based, using DRV8871 IPROPI) | Safety |
| **P0 — Before human use** | Add low-battery shutoff (VBAT < 3.2V → refuse start; < 3.0V → ramp down and stop) | Safety |
| **P0 — Before human use** | Specify and test electrode design (material, contact area, build instructions) | Safety |
| **P1 — Before PCB fab** | Add battery voltage ADC (2-resistor divider to ESP32 ADC) | Hardware |
| **P1 — Before PCB fab** | Consider adding piezo buzzer for session start/end/fault alerts | Hardware |
| **P2 — Firmware** | Implement threshold-finding protocol (ramp + "I feel it" button) | Clinical |
| **P2 — Firmware** | Add new presets: Depression, Epilepsy, Stress, HRV Optimization, Migraine | Clinical |
| **P2 — Firmware** | Track cumulative daily exposure with advisory warning | Safety |
| **P2 — Firmware** | Log raw RR intervals, periodic impedance, battery voltage, compliance events | Data |
| **P3 — Documentation** | Create electrode placement guide with labeled anatomy photos | Clinical |
| **P3 — Documentation** | Add adverse event report template (GitHub issue template) | Safety |
| **P3 — Documentation** | Document respiratory-gated stimulation as future feature | Research |
| **P3 — Future** | Respiratory-gated mode (sync to Polar H10 exhale phase) | Differentiator |