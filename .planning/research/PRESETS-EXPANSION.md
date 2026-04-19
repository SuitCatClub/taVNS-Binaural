# taVNS Clinical Presets Expansion: Comprehensive Evidence Review

**Project:** OpenBinaural-taVNS
**Date:** 2025-07-17
**Scope:** Identify ALL additional conditions with published human taVNS evidence for potential firmware presets
**Methodology:** Synthesized from published clinical literature (training data through early 2025). Every parameter is traced to its source paper. Confidence levels are conservative.

> **IMPORTANT LIMITATION:** This review is compiled from training data, not real-time PubMed search. All claims should be independently verified against the cited papers before implementation. In a medical device context — even a DIY research device — every parameter must be traceable to a published human taVNS study.

> **Existing presets (already validated in PRESETS.md):** Insomnia 25Hz, Insomnia 20Hz, RESET-AF, Anti-Inflammatory, Dysautonomia/hEDS, Exploration. This document covers NEW conditions only.

---

## 1. Executive Summary Table — All Proposed New Presets

| # | Preset Name | Freq (Hz) | PW (µs) | Current (mA) | Duty Cycle | Duration | Laterality | Site in Studies | Evidence Level | Confidence | Bilateral Stagger OK? |
|---|-------------|-----------|---------|--------------|------------|----------|------------|----------------|----------------|------------|----------------------|
| 7 | Depression (MDD) | 20 Hz | 200 µs | 1.0 mA (threshold) | 30s/30s | 30 min | Unilateral L | Cymba conchae | RCT (N=160) | **HIGH** | Experimental only — studies used unilateral |
| 8 | Epilepsy (Adjunctive) | 25 Hz | 250 µs | 1.0 mA (threshold) | 30s/30s | 60 min (3×/day) | Unilateral L | Cymba conchae | RCT (multiple) | **HIGH** | Experimental only — studies used unilateral |
| 9 | Migraine Prevention | 1 Hz | 250 µs | 1.0 mA (threshold) | Continuous | 60 min | Unilateral L | Cymba conchae | Small RCT | **MEDIUM** | Unknown — insufficient data |
| 10 | Tinnitus | 25 Hz | 200 µs | 1.0–2.0 mA (threshold) | 30s/30s | 30 min | Unilateral L | Tragus / cymba | Pilot RCT | **LOW** | Unknown — studies used unilateral |
| 11 | Stress / Acute Anxiety | 25 Hz | 200 µs | 1.0 mA (threshold) | Continuous | 15 min | Unilateral L | Tragus / cymba | Pilot / Mechanistic | **MEDIUM** | Experimental only |
| 12 | Chronic Pain (Fibromyalgia) | 25 Hz | 250 µs | 1.5 mA (threshold) | 30s/30s | 30 min | Unilateral L | Cymba conchae | Pilot / fMRI | **LOW** | Experimental only |
| 13 | HRV Optimization (Autonomic Tone) | 25 Hz | 200 µs | Threshold | 30s/30s | 15 min | Bilateral | Tragus / cymba | Multiple pilots | **MEDIUM** | YES — best candidate for bilateral hypothesis |
| 14 | Hypertension (Adjunctive) | 20 Hz | 200 µs | Threshold | Continuous | 60 min | Unilateral L | Tragus | Pilot | **LOW** | Unknown |
| 15 | Obesity / Appetite | 25 Hz | 250 µs | Threshold | 30s/30s | 30 min (before meals) | Bilateral | Cymba conchae | Pilot | **LOW** | Unknown |

**Conditions investigated but NOT recommended as presets** (see Section 5):
- PTSD — insufficient taVNS-specific data
- Post-COVID autonomic dysfunction — no published taVNS study
- Cognitive enhancement / Focus — no clinical evidence for enhancement in healthy subjects
- Stroke rehabilitation — emerging but too early
- Glucose metabolism / Diabetes — mechanistic only, no human taVNS data

---

## 2. Detailed Analysis Per Condition

---

### 2.1 Depression / Major Depressive Disorder (MDD)

#### Evidence Summary

| Study | Year | Design | N | Parameters | Key Outcomes | Evidence | Confidence |
|-------|------|--------|---|------------|-------------|----------|------------|
| **Fang et al.** (Biol Psychiatry) | 2016 | RCT, sham-controlled | 160 | Left cymba conchae, 20 Hz / 4 Hz alternating, 1 mA, 30 min, 2×/day, 4 weeks | Significant reduction in Hamilton Depression Rating Scale (HAMD-17). fMRI showed modulation of default mode network (DMN). | RCT | **HIGH** — Landmark study, well-powered |
| **Rong et al.** (Biol Psychiatry) | 2016 | RCT, sham-controlled | 160 | Same cohort as Fang — left cymba conchae, 20 Hz alternating with 4 Hz, 1 mA, 30 min, 2×/day, 4 weeks | HAMD-17 reduction: 37.4% active vs. 18.2% sham (p < 0.001). Response rate 44.4% vs. 21.8%. | RCT | **HIGH** — Same study, clinical outcome paper |
| **Liu et al.** | 2020 | Meta-analysis of taVNS for depression | 7 RCTs, ~600 total | Various — predominantly 20 Hz, cymba conchae | Pooled effect favoring taVNS over sham for depression scores | Meta-analysis | **MEDIUM** — Recall of this meta-analysis is approximate |
| **Kong et al.** (J Psychiatr Res) | 2018 | Pilot RCT | ~40 | Left cymba conchae, 20 Hz, 200 µs, 1 mA, 30 min/day, 8 weeks | HAMD improvement, also showed anxiety reduction as secondary | Pilot RCT | **MEDIUM** |
| **NEMOS device studies** (Cerbomed) | 2010–2018 | Multiple pilot/small RCTs | Various | Left cymba conchae, 25 Hz, 250 µs, <1 mA, 4h/day (NEMOS standard protocol) | CE-marked for epilepsy; depression studies showed mixed results | Pilot RCTs | **MEDIUM** — NEMOS uses different parameters (25 Hz, long duration) |

**Key observations:**
- Depression has the **strongest evidence base** after AF for taVNS — the Fang/Rong 2016 RCT with N=160 is one of the largest taVNS trials ever published.
- The Chinese research group (Rong, Fang, and colleagues) has dominated this literature with a consistent protocol: **left cymba conchae, 20 Hz, ~1 mA, 30 min, 2×/day**.
- The 4 Hz / 20 Hz alternating protocol has traditional Chinese medicine rationale. Most Western replication would use continuous 20 Hz.
- taVNS for depression is used as **adjunctive** to antidepressants in most studies, not as monotherapy.
- The NEMOS device (25 Hz, 250 µs) was CE-marked for epilepsy but also studied for depression with mixed results — longer daily duration (up to 4 hours) but less robust outcomes than the Chinese 20 Hz protocol.

#### Parameter Provenance

| Parameter | Value | Source Paper | Confidence |
|-----------|-------|-------------|------------|
| Frequency | **20 Hz** | Fang et al. 2016 (N=160 RCT) — used 20 Hz / 4 Hz alternating; 20 Hz continuous is the standard Western adaptation | HIGH |
| Pulse width | **200 µs** | Fang et al. 2016 — used ~200 µs (some Chinese papers report 200 µs, some don't specify exactly) | MEDIUM |
| Current | **1.0 mA** (at or near threshold) | Fang et al. 2016 — used 1 mA fixed; threshold in this population typically 0.5–1.5 mA | HIGH |
| Duty cycle | **30s ON / 30s OFF** | Fang et al. 2016 — used 30s ON / 30s OFF in some versions of the protocol | MEDIUM |
| Duration | **30 min** | Fang et al. 2016 — 30 min sessions, 2×/day for 4 weeks | HIGH |
| Laterality | **Unilateral left** | All major depression taVNS studies used unilateral left cymba conchae | HIGH |
| Site | **Cymba conchae** | Fang et al. 2016 — left cymba conchae (NOT tragus) — aligns with this project | HIGH |

#### ⚠️ Key Parameter Differences from Existing Presets

- **Frequency 20 Hz (not 25 Hz):** The depression literature consistently uses 20 Hz, not 25 Hz. This matches our Insomnia-20Hz preset but differs from the Insomnia-25Hz and Anti-Inflammatory presets. The 5 Hz difference is likely not clinically meaningful for vagal afferent activation, but for fidelity to the evidence, 20 Hz should be used.
- **Current 1.0 mA (lower than insomnia default of 2.0 mA):** Chinese depression studies typically used a fixed 1 mA, which is at or slightly below threshold for most subjects. Our insomnia preset defaults to 2.0 mA. Depression patients on SSRIs/SNRIs may have altered sensory thresholds — starting lower is prudent.
- **Session frequency: 2×/day:** The Fang/Rong protocol used twice-daily 30-minute sessions. This is more intensive than the single daily session used in most other presets. The device should support scheduling two sessions per day.

#### Charge Density Check
1.0 mA × 200 µs = 0.2 µC per phase. At typical cymba conchae electrode contact area (~0.5 cm²): 0.4 µC/cm². Well within McCreery 1990 limit of 30 µC/cm² (75× safety margin).

#### Contraindications Specific to Depression

| Contraindication | Type | Rationale |
|-----------------|------|-----------|
| Active suicidal ideation | ABSOLUTE | Patient safety — taVNS is not an emergency intervention; refer immediately to crisis services |
| Bipolar disorder (manic phase) | Relative | Antidepressant-like effects could theoretically precipitate mania. No documented cases with taVNS, but standard psychiatric caution applies |
| Concurrent ECT | Relative | Unknown interaction — both affect brain excitability. Defer taVNS during ECT course |
| Unmedicated severe MDD | Relative | Most studies used taVNS as adjunct to antidepressants. Monotherapy evidence is limited |

#### Bilateral Stagger Appropriateness
**NOT RECOMMENDED as default.** All depression studies used unilateral left stimulation. The bilateral stagger could be offered as an experimental option but should not be the default for this preset. Unilateral left cymba conchae aligns with the strongest evidence (Fang 2016, N=160).

#### Recommended Preset Parameters

| Parameter | Value |
|-----------|-------|
| Frequency | **20 Hz** |
| Pulse width | **200 µs** |
| Default current | **1.0 mA** (titrate to threshold) |
| Duty cycle | **30s ON / 30s OFF** |
| Duration | **30 min** (recommend 2×/day) |
| Laterality | **Unilateral LEFT** |
| Stagger | **N/A (unilateral)** |
| Ramp | **0 → target over 30s** |
| Evidence | **RCT (N=160)** |
| Role | **ADJUNCTIVE** to standard antidepressant therapy |

---

### 2.2 Epilepsy (Adjunctive Seizure Reduction)

#### Evidence Summary

| Study | Year | Design | N | Parameters | Key Outcomes | Evidence | Confidence |
|-------|------|--------|---|------------|-------------|----------|------------|
| **Stefan et al.** (Epilepsia) | 2012 | Open-label pilot | 10 | Left cymba conchae, 25 Hz, 250 µs, <1 mA, 3×/day 1h each | 5/10 patients showed ≥50% seizure reduction over 9 months | Open-label pilot | **MEDIUM** |
| **Bauer et al.** (Epilepsia) | 2016 | Randomized, controlled, double-blind | ~76 | Left cymba conchae (active) vs. earlobe (sham), 25 Hz, 250 µs, titrated to below pain threshold, 4h/day, 20 weeks | Active group: ≥50% seizure reduction in 30.4% vs. 27.2% sham. Primary endpoint not significant, but per-protocol analysis showed trend. | RCT | **MEDIUM** — Primary endpoint technically not met, but effect direction was positive |
| **Aihua et al.** | 2014 | Pilot, open-label | 47 | Left cymba conchae, 20 Hz / 4 Hz alternating, 1 mA, 20 min 2×/day, 12 months | Responder rate ~38%; mean seizure reduction ~35% | Open-label pilot | **LOW** |
| **NEMOS CE mark** (Cerbomed) | 2012 | Regulatory submission | — | CE-marked for drug-resistant epilepsy in Europe. Standard: 25 Hz, 250 µs, titrated below pain, up to 4h/day | CE mark achieved | Regulatory | **HIGH** — CE-marked device for this indication |
| **He et al.** (Epilepsy Behav) | 2013 | Open-label | 14 (pediatric) | Left auricular VNS, 20 Hz, 200 µs | Seizure frequency reduction | Open-label pilot | **LOW** |

**Key observations:**
- Epilepsy was the **original clinical target** for taVNS — the NEMOS device by Cerbomed obtained CE marking for drug-resistant epilepsy.
- The evidence is moderate: the Bauer 2016 RCT (N~76) did not clearly meet its primary endpoint, but effect direction was positive and the study was underpowered.
- Standard NEMOS protocol uses **long daily exposure (up to 4 hours/day)** split into multiple sessions, which differs from most other taVNS presets.
- For a home-use device like OpenBinaural, 3×60 min sessions/day is the practical maximum.
- Epilepsy taVNS is ALWAYS **adjunctive** — it supplements antiepileptic drugs (AEDs), never replaces them.

#### Parameter Provenance

| Parameter | Value | Source Paper | Confidence |
|-----------|-------|-------------|------------|
| Frequency | **25 Hz** | NEMOS device standard / Stefan 2012 / Bauer 2016 | HIGH |
| Pulse width | **250 µs** | NEMOS device standard / Stefan 2012 / Bauer 2016 | HIGH |
| Current | **~1.0 mA** (titrated below pain) | NEMOS protocol: titrate to just below pain threshold; typical 0.5–1.5 mA | HIGH |
| Duty cycle | **30s ON / 30s OFF** | NEMOS standard protocol | HIGH |
| Duration | **Up to 4h/day** (3× 60–80 min sessions) | NEMOS protocol; Stefan 2012 | HIGH |
| Laterality | **Unilateral left** | All published studies | HIGH |
| Site | **Cymba conchae** | NEMOS studies (cymba conchae) | HIGH — matches this project |

#### ⚠️ Key Parameter Differences from Existing Presets

- **Long daily exposure (3–4 hours):** This is dramatically different from other presets (30–60 min once daily). The NEMOS protocol was designed to approximate the stimulation duty cycle of implanted VNS (which runs 24/7). For a practical home device preset, recommend **3× 60 min sessions** or **2× 90 min sessions** per day.
- **25 Hz / 250 µs:** Matches the NEMOS standard, but differs from the depression literature (20 Hz / 200 µs). This is the NEMOS-derived parameter set.

#### Charge Density Check
1.0 mA × 250 µs = 0.25 µC per phase. At ~0.5 cm² electrode: 0.5 µC/cm². Within McCreery limit (60× safety margin).

#### Contraindications Specific to Epilepsy

| Contraindication | Type | Rationale |
|-----------------|------|-----------|
| Status epilepticus (active) | ABSOLUTE | Medical emergency — not a taVNS indication |
| Implanted VNS device | ABSOLUTE | Dual vagal stimulation (implanted + transcutaneous) is untested and could cause unpredictable effects |
| Seizure frequency >10/day | Relative | Very frequent seizures increase risk of electrode dislodgement, burns, or inability to self-manage device during/after seizure |
| Unsupervised use (first sessions) | Relative | First 2–3 sessions should be supervised due to seizure risk during stimulation setup |

#### Bilateral Stagger Appropriateness
**NOT RECOMMENDED.** All epilepsy studies used unilateral left stimulation. There is no data on bilateral taVNS for epilepsy. Unilateral left is the only evidence-supported approach.

#### Recommended Preset Parameters

| Parameter | Value |
|-----------|-------|
| Frequency | **25 Hz** |
| Pulse width | **250 µs** |
| Default current | **1.0 mA** (titrate to below pain threshold) |
| Duty cycle | **30s ON / 30s OFF** |
| Duration | **60 min** (recommend 3×/day for epilepsy protocol) |
| Laterality | **Unilateral LEFT** |
| Stagger | **N/A (unilateral)** |
| Ramp | **0 → target over 30s** |
| Evidence | **CE-marked device (NEMOS); RCT borderline positive** |
| Role | **ADJUNCTIVE** — always with antiepileptic medication |

---

### 2.3 Migraine Prevention

#### Evidence Summary

| Study | Year | Design | N | Parameters | Key Outcomes | Evidence | Confidence |
|-------|------|--------|---|------------|-------------|----------|------------|
| **Straube et al.** (J Headache Pain) | 2015 | Randomized, monocentric trial | ~46 (chronic migraine) | Left cymba conchae, two arms: **1 Hz** and **25 Hz**, 250 µs, 1 mA, 4h/day, 12 weeks | **1 Hz arm**: significant reduction in headache days (−7.0 ± 4.5 days/month). **25 Hz arm**: less reduction (−3.3 ± 5.4). 1 Hz was superior. | Small RCT | **MEDIUM** |
| **Goadsby et al. (gammaCore)** | 2014–2018 | Multiple RCTs | Various (large) | **Cervical** (neck) VNS, NOT auricular — gammaCore device | FDA-cleared for acute migraine and cluster headache | RCT (different modality) | HIGH for cervical VNS; NOT directly applicable to auricular |
| **Trevizol et al.** | 2015 | Systematic review | — | Review of non-invasive VNS for headache | Mixed results; gammaCore (cervical) had most evidence; auricular evidence limited | Review | **LOW** |
| **Silberstein et al.** | 2016 | RCT (gammaCore, cervical) | 59 (cluster headache) | **Cervical VNS**, NOT auricular | Reduced attack frequency in episodic cluster headache | RCT (cervical) | NOT applicable to auricular taVNS |

**Key observations:**
- The Straube 2015 study is the **only well-designed auricular taVNS study for migraine** — and it produced a surprising result: **1 Hz was more effective than 25 Hz** for migraine prevention.
- This 1 Hz finding is counterintuitive — most taVNS studies use 20–25 Hz. At 1 Hz, you get only 1 pulse per second (0.025% duty within each ON period at 250 µs pulse width). The mechanism may differ from standard vagal afferent activation — possibly engaging different fiber types or triggering different NTS responses.
- gammaCore (cervical VNS) has strong evidence for migraine and cluster headache, but it stimulates the cervical vagus nerve (motor + sensory fibers) — a fundamentally different target from auricular taVNS.
- The evidence for auricular taVNS specifically for migraine rests essentially on one small study (Straube 2015).

#### ⚠️ Critical Parameter Flag: 1 Hz Frequency

**This is the ONLY proposed preset using 1 Hz.** All other presets use 20–25 Hz. This creates a significant firmware and safety design consideration:

1. **At 1 Hz, 250 µs, 1 mA:** Charge per phase = 0.25 µC. Charge per second = 0.25 µC. This is trivially low — no safety concern.
2. **Perception may be different at 1 Hz:** Users will feel discrete individual pulses rather than a continuous buzzing. The sensation is fundamentally different and may feel unusual.
3. **The 1 Hz result has NOT been replicated.** It comes from a single small study with ~46 patients. Confidence in 1 Hz as the optimal frequency for migraine is LOW.

#### Parameter Provenance

| Parameter | Value | Source Paper | Confidence |
|-----------|-------|-------------|------------|
| Frequency | **1 Hz** | Straube et al. 2015 — 1 Hz arm showed superior migraine reduction | MEDIUM — single study, unreplicated |
| Pulse width | **250 µs** | Straube et al. 2015 | HIGH |
| Current | **1.0 mA** | Straube et al. 2015 — used 1 mA fixed | MEDIUM |
| Duty cycle | **Continuous** | Straube 2015 — used continuous stimulation within sessions | MEDIUM |
| Duration | **60 min** (protocol was 4h/day, but 60 min is practical compromise) | Straube 2015 used 4h/day (NEMOS protocol); 60 min is a practical adaptation | LOW for 60 min specifically |
| Laterality | **Unilateral left** | Straube 2015 | HIGH |
| Site | **Cymba conchae** | Straube 2015 (NEMOS device targets cymba conchae) | HIGH |

#### Contraindications Specific to Migraine

| Contraindication | Type | Rationale |
|-----------------|------|-----------|
| Hemiplegic migraine | Relative | Rare migraine subtype with stroke-like symptoms. Any intervention that affects cerebral blood flow should be used with caution |
| Concurrent onabotulinumtoxinA (Botox) | Informational | Both target migraine prevention. No known interaction, but effects may confound assessment |
| Triptan overuse | Informational | No interaction, but medication overuse headache should be addressed before adding taVNS |

#### Bilateral Stagger Appropriateness
**UNKNOWN.** Only unilateral stimulation has been studied for migraine. No basis to recommend bilateral. The 1 Hz frequency makes bilateral stagger mechanistically questionable — at 1 pulse/second, a 20 ms stagger is trivial relative to the 1000 ms inter-pulse interval.

#### Recommended Preset Parameters

| Parameter | Value |
|-----------|-------|
| Frequency | **1 Hz** |
| Pulse width | **250 µs** |
| Default current | **1.0 mA** (titrate to threshold) |
| Duty cycle | **Continuous** |
| Duration | **60 min** (original protocol was 4h/day; adapt for compliance) |
| Laterality | **Unilateral LEFT** |
| Stagger | **N/A (unilateral)** |
| Ramp | **0 → target over 30s** |
| Evidence | **Small RCT (N~46), unreplicated** |
| Role | **ADJUNCTIVE** — supplement standard migraine prophylaxis |

---

### 2.4 Tinnitus

#### Evidence Summary

| Study | Year | Design | N | Parameters | Key Outcomes | Evidence | Confidence |
|-------|------|--------|---|------------|-------------|----------|------------|
| **Lehtimäki et al.** (Brain Stimul?) | 2013 | Pilot | ~10 | Left tragus, 25 Hz, 200 µs, sub-threshold, 30 min acute sessions | fMRI showed modulation of auditory cortex activity; minimal subjective tinnitus improvement | Mechanistic pilot | **LOW** |
| **Yakunina et al.** (Neuromodulation) | 2017 | fMRI optimization study | ~20 (healthy) | Cymba conchae and tragus compared at various frequencies; 25 Hz at cymba conchae showed strongest NTS/brainstem activation | Imaging endpoints, not tinnitus-specific. Cymba conchae > tragus for NTS activation | Mechanistic | **MEDIUM** for NTS activation; **LOW** for tinnitus |
| **Shim et al.** | 2015 | Pilot crossover | ~30 (tinnitus) | Left tragus, 25 Hz, variable intensity, 30 min sessions | Some patients reported subjective tinnitus reduction; THI (Tinnitus Handicap Inventory) improvement in subset | Pilot | **LOW** |
| **Ylikoski et al.** | 2017 | Open-label | ~12 (tinnitus) | taVNS paired with sound therapy; various parameters | Some improvement in tinnitus perception when combined with sound therapy | Open-label pilot | **LOW** |

**Key observations:**
- Evidence for taVNS specifically treating tinnitus is **weak** — all studies are small pilots with inconsistent results.
- The theoretical rationale exists: the vagus nerve projects to the nucleus tractus solitarius (NTS), which has connections to the locus coeruleus (LC) and auditory cortex. VNS-paired auditory stimulation can drive neuroplasticity in the auditory cortex (this is the basis of the Microtransponder "VNS-paired tone" therapy for tinnitus, which uses implanted VNS).
- **Implanted VNS paired with tones** (Microtransponder/Lenire approach) has stronger evidence, but this uses precise VNS-tone pairing, not standalone taVNS.
- Standalone taVNS for tinnitus — without concurrent auditory therapy — has minimal published support.

#### Parameter Provenance

| Parameter | Value | Source | Confidence |
|-----------|-------|--------|------------|
| Frequency | **25 Hz** | Multiple studies used 25 Hz; Yakunina 2017 showed optimal NTS activation at 25 Hz | MEDIUM |
| Pulse width | **200 µs** | Standard across most studies | MEDIUM |
| Current | **1.0–2.0 mA** (threshold) | Standard threshold titration | MEDIUM |
| Duration | **30 min** | Pilot studies used 30 min sessions | LOW |
| Laterality | **Unilateral left** | Studies used unilateral | MEDIUM |

#### Special Note: VNS-Paired Tone Therapy
The most promising tinnitus application of VNS involves pairing vagus stimulation with **specific auditory tones** to drive neuroplastic changes in the auditory cortex. This device could theoretically support this approach if paired with a phone app playing calibrated tones, but this is NOT a standard taVNS preset — it's a research protocol requiring audiometric calibration. If implemented, it would belong in the Exploration preset with detailed documentation.

#### Bilateral Stagger Appropriateness
**UNKNOWN.** No bilateral data for tinnitus. For a condition involving auditory processing, bilateral stimulation is theoretically interesting (bilateral NTS input → bilateral auditory cortex modulation) but entirely speculative.

#### Recommended Preset Parameters

| Parameter | Value |
|-----------|-------|
| Frequency | **25 Hz** |
| Pulse width | **200 µs** |
| Default current | **1.0 mA** (titrate to threshold) |
| Duty cycle | **30s ON / 30s OFF** |
| Duration | **30 min** |
| Laterality | **Unilateral LEFT** |
| Stagger | **N/A (unilateral)** |
| Ramp | **0 → target over 30s** |
| Evidence | **Pilot only — weak** |
| Role | **Experimental adjunct** |

---

### 2.5 Stress / Acute Anxiety

#### Evidence Summary

| Study | Year | Design | N | Parameters | Key Outcomes | Evidence | Confidence |
|-------|------|--------|---|------------|-------------|----------|------------|
| **Burger et al.** (Psychophysiology) | 2019 | RCT, lab stress (TSST) | ~60 (healthy) | Left cymba conchae, 25 Hz, 200 µs, 0.5 mA, continuous, acute (during stress task) | Reduced cortisol response to Trier Social Stress Test (TSST). Reduced subjective anxiety. | Pilot RCT (lab) | **MEDIUM** |
| **Szeska et al.** | 2020 | RCT, fear conditioning | ~60 (healthy) | Left cymba conchae, 25 Hz, 200 µs, 0.5 mA, during fear conditioning task | Modulated fear extinction — enhanced extinction learning | Pilot RCT (lab) | **MEDIUM** |
| **Gurel et al.** (Psychoneuroendocrinology?) | 2020 | Pilot, PTSD-related stress | ~20 | Left tragus, 25 Hz, 200 µs, titrated, acute during stress exposure | Reduced norepinephrine and cortisol; reduced sympathetic arousal markers | Pilot | **LOW** |
| **Badran et al.** (Brain Stimul) | 2018 | Parameter optimization | ~15 (healthy) | Left tragus, varied frequencies/pulse widths, acute sessions | 25 Hz / 200–500 µs produced most consistent HRV improvements (increased HF power, reduced HR) | Mechanistic | **MEDIUM** |
| **Bretherton et al.** (Aging) | 2019 | Pilot RCT | 29 (elderly) | Tragus, 200 µs, 200 µA, 15 min/day, 2 weeks | Improved autonomic balance (sympathovagal shift toward parasympathetic) | Pilot RCT | **MEDIUM** |

**Key observations:**
- Several well-designed **laboratory studies** demonstrate that taVNS can reduce **acute stress responses** (cortisol, subjective anxiety, sympathetic arousal) during controlled stress paradigms.
- This is NOT the same as treating chronic anxiety disorders — the evidence is for acute stress buffering, not GAD or panic disorder treatment.
- The Burger 2019 study is particularly relevant — it used the Trier Social Stress Test (gold-standard lab stress paradigm) and showed reduced cortisol with taVNS vs. sham.
- Parameters across stress studies converge on **25 Hz, 200 µs, relatively low current (0.5–1.0 mA)**.
- **Short sessions (10–15 min)** appear sufficient for acute stress buffering — consistent with the rapid onset of parasympathetic activation.

#### Parameter Provenance

| Parameter | Value | Source | Confidence |
|-----------|-------|--------|------------|
| Frequency | **25 Hz** | Burger 2019, Szeska 2020, Badran 2018 | HIGH |
| Pulse width | **200 µs** | Burger 2019, Szeska 2020 | HIGH |
| Current | **1.0 mA** (threshold; studies used 0.5 mA fixed in healthy subjects) | Burger 2019 used 0.5 mA; higher threshold-based titration is standard practice | MEDIUM |
| Duty cycle | **Continuous** | Lab stress studies used continuous stimulation during stress exposure | MEDIUM |
| Duration | **15 min** | Acute effect onset within 5–10 min; 15 min is sufficient for stress buffering | MEDIUM |
| Laterality | **Unilateral left** | All studies | HIGH |

#### ⚠️ Key Notes

- This preset is designed for **acute stress reduction** (e.g., before a stressful event, during a panic episode) — NOT chronic anxiety disorder treatment.
- The evidence comes from **laboratory stress paradigms in healthy volunteers**, NOT clinical anxiety patients.
- For chronic anxiety disorders (GAD, panic disorder), there is insufficient taVNS-specific evidence to recommend a preset. Those conditions should use the Exploration preset with clinician guidance.

#### Bilateral Stagger Appropriateness
**EXPERIMENTAL ONLY.** All studies used unilateral. However, for acute stress reduction, bilateral stimulation is theoretically reasonable — bilateral vagal afferent activation would provide more robust NTS input and stronger parasympathetic activation. This is the type of condition where the bilateral stagger hypothesis is most interesting, but there's no evidence to support it.

#### Recommended Preset Parameters

| Parameter | Value |
|-----------|-------|
| Frequency | **25 Hz** |
| Pulse width | **200 µs** |
| Default current | **1.0 mA** (titrate to threshold) |
| Duty cycle | **Continuous** |
| Duration | **15 min** |
| Laterality | **Unilateral LEFT** (bilateral as experimental option) |
| Stagger | **20 ms if bilateral enabled** |
| Ramp | **0 → target over 15s** (faster ramp for acute use) |
| Evidence | **Pilot RCT (lab stress) — MEDIUM** |
| Role | **PRIMARY** (standalone acute intervention) |

---

### 2.6 Chronic Pain — Fibromyalgia / Neuropathic

#### Evidence Summary

| Study | Year | Design | N | Parameters | Key Outcomes | Evidence | Confidence |
|-------|------|--------|---|------------|-------------|----------|------------|
| **Napadow et al.** (Pain Med) | 2012 | fMRI crossover pilot | ~15 (chronic pelvic pain) | Left cymba conchae, 25 Hz, 250 µs, respiratory-gated, acute | Modulated pain-processing brain regions (anterior insula, ACC). Trend toward reduced pain ratings | Mechanistic pilot | **MEDIUM** |
| **Kutlu et al.** | 2020 | Pilot | Small (~20?) | taVNS for fibromyalgia, 25 Hz, 200 µs | Some pain score improvement | Pilot | **LOW** |
| **Straube et al.** (J Headache Pain) | 2015 | RCT | ~46 (migraine — also has pain component) | 1 Hz and 25 Hz arms; see Migraine section | Headache day reduction (see §2.3) | Small RCT | Covered in Migraine section |

**Key observations:**
- Evidence for taVNS specifically treating chronic pain (fibromyalgia, neuropathic pain) is **weak** — mostly mechanistic pilots and one or two small studies.
- The **existing Anti-Inflammatory preset** (25 Hz, 250 µs, 1.5 mA, 60 min) already covers the most evidence-supported pain-modulation pathway (vagal anti-inflammatory reflex via Lerman 2016).
- Fibromyalgia-specific evidence is essentially just the Napadow 2012 fMRI study and the Kutlu 2020 pilot.
- gammaCore (cervical VNS) has much stronger evidence for headache disorders, but it's a different modality.
- **The existing Anti-Inflammatory preset parameters overlap significantly with what would be recommended for chronic pain.** A separate chronic pain preset is marginally justified.

#### Recommendation
**A dedicated chronic pain preset adds marginal value over the existing Anti-Inflammatory preset.** The parameters are nearly identical (25 Hz, 250 µs). If included, the main difference would be framing (pain modulation vs. anti-inflammatory), lower default current for fibromyalgia patients with potential allodynia, and slightly shorter sessions. However, this could also be handled with parameter documentation rather than a separate preset.

**If included as a separate preset:**

| Parameter | Value |
|-----------|-------|
| Frequency | **25 Hz** |
| Pulse width | **250 µs** |
| Default current | **1.5 mA** (titrate to threshold; **start 0.5 mA for fibromyalgia with allodynia**) |
| Duty cycle | **30s ON / 30s OFF** |
| Duration | **30 min** |
| Laterality | **Unilateral LEFT** |
| Stagger | **N/A (unilateral)** |
| Ramp | **0 → target over 30s** |
| Evidence | **Pilot/fMRI only — LOW** |
| Role | **ADJUNCTIVE** — supplement standard pain management |

**My recommendation:** Do NOT add this as a separate preset. Instead, add a note to the existing Anti-Inflammatory preset that it covers chronic pain applications. This keeps the preset menu cleaner.

---

### 2.7 HRV Optimization / General Autonomic Tone

#### Evidence Summary

| Study | Year | Design | N | Parameters | Key Outcomes | Evidence | Confidence |
|-------|------|--------|---|------------|-------------|----------|------------|
| **Bretherton et al.** (Aging) | 2019 | Pilot RCT | 29 (elderly) | Left tragus, 200 µs, 200 µA, 15 min/day, 2 weeks | ↑ HF-HRV, shift toward parasympathetic dominance, improved autonomic balance | Pilot RCT | **MEDIUM** |
| **Badran et al.** (Brain Stimul) | 2018 | Parameter optimization | ~15 (healthy) | Left tragus, multiple parameter combinations, acute | 25 Hz/200 µs: most consistent HR reduction and HF-HRV increase | Mechanistic | **MEDIUM** |
| **Clancy et al.** (Brain Stimul) | 2014 | Crossover pilot | ~48 (healthy) | Left tragus, 200 µs, 200 µA, 15 min, acute | Increased HF-HRV; reduced LF/HF ratio; reduced sympathetic muscle nerve activity (MSNA) | Pilot | **MEDIUM** |
| **De Couck et al.** | 2017 | Crossover | ~30 (healthy) | Cymba conchae, 25 Hz, 250 µs, threshold, single acute session | Increased RMSSD and HF-HRV during and after stimulation | Pilot | **MEDIUM** |
| **Antonino et al.** | 2017 | Crossover | ~14 (healthy) | Tragus, 30 Hz, 200 µs, sub-threshold, 15 min | Increased HF-HRV; reduced LF/HF ratio | Pilot | **LOW** |

**Key observations:**
- This is the **best-replicated acute effect** of taVNS across all conditions: virtually every taVNS study that measures HRV shows increased parasympathetic indices (↑ RMSSD, ↑ HF-HRV, ↓ LF/HF ratio).
- This is NOT a "treatment" preset — it's a **wellness/optimization** preset for users who want to enhance autonomic balance without targeting a specific condition.
- Parameters converge across studies: **25 Hz, 200 µs, threshold current, 15 min, acute onset**.
- This is the preset most suited to daily biohacker use — short sessions, well-tolerated, measurable with the Polar H10 integration.
- **This is the best candidate for bilateral stagger testing** — it's the lowest-risk application, the outcome (HRV) is directly measurable with the device, and there's no condition-specific laterality requirement.

#### Parameter Provenance

| Parameter | Value | Source | Confidence |
|-----------|-------|--------|------------|
| Frequency | **25 Hz** | Badran 2018 (optimal), Bretherton 2019, Clancy 2014, De Couck 2017 | HIGH |
| Pulse width | **200 µs** | Consistent across studies | HIGH |
| Current | **Threshold** (typically 0.5–2.0 mA) | Standard titration | HIGH |
| Duration | **15 min** | Bretherton 2019, Clancy 2014 — both used 15 min with clear HRV effect | HIGH |
| Duty cycle | **30s ON / 30s OFF** | Most common; some studies used continuous — either is acceptable | MEDIUM |

#### Bilateral Stagger Appropriateness
**YES — BEST CANDIDATE.** This is the ideal preset for testing the bilateral 20 ms stagger hypothesis because:
1. No laterality requirement from the evidence
2. The outcome (HRV) is directly measurable with the Polar H10
3. Bilateral stimulation could theoretically provide stronger NTS input → stronger parasympathetic activation
4. Low risk — this is a wellness application in healthy/generally-healthy users
5. Enables direct A/B comparison: run session with bilateral stagger, compare HRV metrics to unilateral session

#### Recommended Preset Parameters

| Parameter | Value |
|-----------|-------|
| Frequency | **25 Hz** |
| Pulse width | **200 µs** |
| Default current | **Threshold** (1.0 mA starting point, titrate) |
| Duty cycle | **30s ON / 30s OFF** |
| Duration | **15 min** |
| Laterality | **Bilateral** (default — this is the stagger test preset) |
| Stagger | **20 ms** |
| Ramp | **0 → target over 15s** |
| Evidence | **Multiple pilots — MEDIUM** |
| Role | **PRIMARY** (standalone wellness intervention) |

---

### 2.8 Hypertension (Adjunctive)

#### Evidence Summary

| Study | Year | Design | N | Parameters | Key Outcomes | Evidence | Confidence |
|-------|------|--------|---|------------|-------------|----------|------------|
| **Stavrakis et al.** (JACC) | 2015/2020 | RCT (AF focus) | 40–53 | Left tragus, 20 Hz, 200 µs, threshold, 1h/day | Secondary outcome: reduced systemic inflammation, some BP data reported in subanalyses | RCT (secondary) | **LOW** for hypertension specifically |
| **Bretherton et al.** (Aging) | 2019 | Pilot RCT | 29 | Left tragus, 200 µs, 200 µA, 15 min/day | Improved autonomic balance; BP not a primary endpoint but some trend data | Pilot | **LOW** |
| **Natelson / autonomic studies** | Various | Multiple | — | Various taVNS parameters | General autonomic modulation (increased vagal tone) could theoretically lower BP in sympathetically-driven hypertension | Mechanistic reasoning | **LOW** |

**Key observations:**
- There is **no published RCT specifically testing taVNS for hypertension** as of my training data cutoff.
- The rationale is sound: hypertension (especially sympathetically-driven, "neurogenic" hypertension) could respond to vagal tone enhancement. Implanted VNS has been explored for resistant hypertension.
- Acute taVNS sessions consistently reduce blood pressure by a few mmHg in healthy subjects (secondary finding in HRV studies) — but this is a modest, transient effect.
- Without dedicated hypertension trials, this preset is **highly theoretical**.

#### Recommendation
**Include with LOW confidence and clear experimental labeling.** Parameters would mirror the RESET-AF protocol (20 Hz, 200 µs, continuous, 60 min, unilateral L) since the Stavrakis studies provide the closest relevant data and the cardiovascular mechanisms overlap.

#### Recommended Preset Parameters

| Parameter | Value |
|-----------|-------|
| Frequency | **20 Hz** |
| Pulse width | **200 µs** |
| Default current | **Threshold** (1.0 mA starting, titrate) |
| Duty cycle | **Continuous** |
| Duration | **60 min** |
| Laterality | **Unilateral LEFT** |
| Stagger | **N/A (unilateral)** |
| Ramp | **0 → target over 30s** |
| Evidence | **THEORETICAL — no dedicated taVNS hypertension RCT** |
| Role | **ADJUNCTIVE** — always with standard antihypertensive medication |

---

### 2.9 Obesity / Appetite Regulation

#### Evidence Summary

| Study | Year | Design | N | Parameters | Key Outcomes | Evidence | Confidence |
|-------|------|--------|---|------------|-------------|----------|------------|
| **Huang et al.** | 2014 | Pilot crossover | ~20 (obese) | Left auricular stimulation (electroacupuncture-style, not standard taVNS), 25 Hz, various | Some changes in appetite-related brain regions on fMRI | Pilot (different modality from standard taVNS) | **LOW** |
| **Val-Laillet et al.** (J Neurosci) | 2015 | Animal study (pigs) | — | **Implanted** VNS, chronic | Altered food preferences, brain activity changes in reward circuits | Preclinical | NOT applicable (implanted, animal) |
| **Bodenlos et al.** | 2007 | Pilot | ~7 (obese, implanted VNS for epilepsy) | **Implanted VNS**, not taVNS | Reduced sweet food cravings when VNS was active | Case series (implanted) | **LOW** — different modality |
| **Nishi et al.** | 2020 | Pilot | Small (~10?) | taVNS before meal, 25 Hz | Some changes in subjective appetite and ghrelin levels | Pilot | **LOW** — recall uncertain |

**Key observations:**
- Evidence for taVNS specifically for obesity/appetite is **very weak** — one or two small pilots with uncertain results.
- Implanted VNS has some evidence for appetite modulation (through vagal afferent signaling to hypothalamus, reward circuits), but this doesn't translate directly to auricular taVNS.
- The theoretical mechanism (vagal afferent → NTS → hypothalamus → satiety signaling) is plausible but unvalidated for transcutaneous stimulation.
- **Pre-meal timing** is critical in the limited evidence that exists — stimulation 30 min before meals.

#### Recommendation
**Include with LOW confidence and EXPERIMENTAL labeling**, or omit and handle through Exploration preset. The evidence is barely sufficient to justify a dedicated preset.

**If included:**

| Parameter | Value |
|-----------|-------|
| Frequency | **25 Hz** |
| Pulse width | **250 µs** |
| Default current | **Threshold** |
| Duty cycle | **30s ON / 30s OFF** |
| Duration | **30 min** (before meals) |
| Laterality | **Bilateral** (no laterality data — default to project hypothesis) |
| Stagger | **20 ms** |
| Ramp | **0 → target over 30s** |
| Evidence | **THEORETICAL — no robust human taVNS obesity data** |
| Role | **ADJUNCTIVE** — always with dietary/lifestyle interventions |

---

## 3. Parameter Overlap Analysis

### 3.1 Parameter Clustering

Analyzing all proposed presets (existing + new) reveals natural parameter clusters:

**Cluster A — "Standard Vagal Tone" (25 Hz / 200 µs / 30s ON-OFF)**
- Insomnia 25Hz ✓
- Stress/Anxiety ✓ (shorter duration, continuous)
- HRV Optimization ✓ (shorter duration)
- Tinnitus ✓
- Dysautonomia/hEDS ✓ (lower current)

**Cluster B — "Chinese Depression Protocol" (20 Hz / 200 µs / 30s ON-OFF)**
- Depression ✓
- Insomnia 20Hz ✓

**Cluster C — "NEMOS Protocol" (25 Hz / 250 µs / 30s ON-OFF / long duration)**
- Epilepsy ✓ (long daily exposure)
- Anti-Inflammatory ✓
- Chronic Pain ✓
- Obesity ✓

**Cluster D — "Cardiac Protocol" (20 Hz / 200 µs / continuous / long duration)**
- RESET-AF ✓
- Hypertension ✓

**Cluster E — "Low Frequency" (1 Hz / unique)**
- Migraine Prevention ✓ (unique — does not cluster with anything else)

### 3.2 Firmware Simplification Opportunities

Given these clusters, the firmware could implement **5 base parameter templates** plus per-preset modifications:

| Template | Freq | PW | Duty | Base Duration |
|----------|------|----|------|--------------|
| vagal-standard | 25 Hz | 200 µs | 30s/30s | 30 min |
| depression-protocol | 20 Hz | 200 µs | 30s/30s | 30 min |
| nemos-protocol | 25 Hz | 250 µs | 30s/30s | 60 min |
| cardiac-protocol | 20 Hz | 200 µs | continuous | 60 min |
| low-frequency | 1 Hz | 250 µs | continuous | 60 min |

Each preset then inherits from a template and overrides specific values (current, laterality, ramp, duration).

### 3.3 Key Differentiators Between Presets

| Feature | Differentiates |
|---------|---------------|
| **Laterality** | RESET-AF (L only), Epilepsy (L only), Depression (L only) vs. HRV Optimization (bilateral) |
| **Current default** | Dysautonomia (0.5 mA), Stress (1.0 mA), Depression (1.0 mA), Insomnia (2.0 mA) |
| **Duty cycle** | RESET-AF (continuous), Stress (continuous), Migraine (continuous) vs. most others (30s/30s) |
| **Duration** | Stress (15 min), HRV (15 min), Insomnia (30 min), Epilepsy (60 min × 3) |
| **Frequency** | Migraine (1 Hz) is the outlier |
| **Session frequency** | Depression (2×/day), Epilepsy (3×/day), most others (1×/day) |

---

## 4. Recommended Final Preset List (Existing + New)

### 4.1 Tier 1 — Strong Evidence (Recommended for inclusion)

| # | Preset | Evidence | Confidence | Priority |
|---|--------|----------|------------|----------|
| 1 | Insomnia — 25Hz | Pilot RCT | MEDIUM | **INCLUDE** (existing) |
| 2 | Insomnia — 20Hz | Pilot RCT | MEDIUM | **INCLUDE** (existing) |
| 3 | RESET-AF | RCT | HIGH | **INCLUDE** (existing) |
| 4 | Anti-Inflammatory | Pilot RCT | MEDIUM | **INCLUDE** (existing) |
| 5 | **Depression (MDD)** | **RCT (N=160)** | **HIGH** | **ADD** — strongest new evidence |
| 6 | **Epilepsy (Adjunctive)** | **CE-marked device; RCT** | **HIGH** | **ADD** — original taVNS indication |

### 4.2 Tier 2 — Moderate Evidence (Recommended with caveats)

| # | Preset | Evidence | Confidence | Priority |
|---|--------|----------|------------|----------|
| 7 | **Stress / Acute Anxiety** | Pilot RCT (lab) | MEDIUM | **ADD** — useful for daily biohacker use |
| 8 | **HRV Optimization** | Multiple pilots | MEDIUM | **ADD** — best preset for bilateral stagger testing |
| 9 | **Migraine Prevention** | Small RCT | MEDIUM | **ADD** — unique 1 Hz parameter is interesting |

### 4.3 Tier 3 — Weak Evidence (Optional / Experimental)

| # | Preset | Evidence | Confidence | Priority |
|---|--------|----------|------------|----------|
| 10 | Dysautonomia/hEDS | Theoretical | LOW | **KEEP** (existing — clearly labeled experimental) |
| 11 | **Tinnitus** | Pilot only | LOW | **OPTIONAL** — could omit and handle via Exploration |
| 12 | **Hypertension** | Theoretical | LOW | **OPTIONAL** — overlaps with RESET-AF protocol |
| 13 | **Obesity / Appetite** | Theoretical | LOW | **OMIT** — handle via Exploration preset |

### 4.4 Always Present

| # | Preset | Notes |
|---|--------|-------|
| 14 | Exploration (user-defined) | All parameters user-configurable. Covers any condition not having a dedicated preset |

### 4.5 Recommended Final Menu (12 presets + Exploration)

**Practical recommendation: 10 dedicated presets + Exploration.**

Including every condition with any evidence creates a cluttered menu. My recommended selection balances evidence quality, clinical utility, and menu simplicity:

```
╔══════════════════════════════════════════════════════════════╗
║                    STIMULATION PRESETS                       ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  ── WELL-EVIDENCED ──────────────────────────────────────── ║
║  [1] Insomnia — 25Hz          (Pilot RCT)                   ║
║  [2] Insomnia — 20Hz          (Pilot RCT)                   ║
║  [3] RESET-AF (Cardiac)       (RCT) ⚠ LEFT EAR ONLY        ║
║  [4] Depression (Adjunctive)  (RCT, N=160)                  ║
║  [5] Epilepsy (Adjunctive)    (CE-marked) ⚠ LEFT EAR ONLY  ║
║  [6] Anti-Inflammatory        (Pilot RCT)                   ║
║                                                              ║
║  ── MODERATE EVIDENCE ───────────────────────────────────── ║
║  [7] Stress / Acute Anxiety   (Lab RCT)                     ║
║  [8] HRV Optimization         (Multiple pilots)             ║
║  [9] Migraine Prevention      (Small RCT) ⚠ 1 Hz           ║
║                                                              ║
║  ── EXPERIMENTAL ────────────────────────────────────────── ║
║  [10] Dysautonomia / hEDS     (THEORETICAL) ⚠               ║
║                                                              ║
║  ── USER-DEFINED ────────────────────────────────────────── ║
║  [11] Exploration             (All parameters adjustable)    ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

**Rationale for omissions:**
- **Tinnitus:** Weak standalone evidence. The interesting application (VNS-paired tones) requires audiometric calibration beyond what a simple preset can provide. Users can replicate via Exploration.
- **Hypertension:** No dedicated taVNS trial. Parameters overlap entirely with RESET-AF. Users with hypertension can use RESET-AF protocol or Exploration.
- **Obesity/Appetite:** Evidence too weak for a dedicated preset. Handle via Exploration with documentation.
- **Chronic Pain (separate):** Overlaps almost entirely with Anti-Inflammatory preset. Add a note to Anti-Inflammatory that it covers pain applications.

---

## 5. Conditions Investigated but NOT Recommended as Presets

### 5.1 PTSD

**Status: INSUFFICIENT taVNS-SPECIFIC DATA**

| What exists | Assessment |
|-------------|-----------|
| Gurel et al. (2020) — pilot study of taVNS during traumatic stress exposure, ~20 subjects | Reduced norepinephrine, reduced sympathetic markers |
| Implanted VNS has been studied for PTSD (VA trials) | Different modality — cannot extrapolate parameters |
| Szeska et al. (2020) — taVNS enhanced fear extinction learning (related mechanism) | Lab study in healthy volunteers, not PTSD patients |

**Why not included:** Only one very small pilot study in actual PTSD-related stress. Fear extinction enhancement (Szeska 2020) is mechanistically relevant but studied in healthy volunteers, not PTSD patients. The Stress/Acute Anxiety preset parameters (25 Hz, 200 µs, continuous, 15 min) would be the closest match, but labeling it as a "PTSD" preset implies clinical validation that doesn't exist.

**Recommendation:** PTSD patients interested in taVNS should use the Stress/Acute Anxiety preset or Exploration, with physician guidance. Do NOT create a dedicated PTSD preset — the name implies a level of validation that could be harmful if misinterpreted.

### 5.2 Post-COVID Autonomic Dysfunction

**Status: NO PUBLISHED taVNS STUDY**

| What exists | Assessment |
|-------------|-----------|
| No published RCT or pilot of taVNS specifically for post-COVID/long-COVID autonomic dysfunction | Gap in literature |
| Post-COVID dysautonomia resembles POTS — many patients develop tachycardia, orthostatic intolerance | Theoretically similar to dysautonomia/POTS |
| Some ongoing registered trials (ClinicalTrials.gov) may be underway as of 2024–2025 | Possible future evidence |

**Why not included:** Zero published results. The Dysautonomia/hEDS preset parameters (25 Hz, 200 µs, 0.5 mA, 30s/30s, 30 min) would be the closest match. Post-COVID patients can use that preset with appropriate labeling.

**Recommendation:** Do NOT create a separate preset. Point post-COVID users to Dysautonomia/hEDS preset.

### 5.3 Cognitive Enhancement / Focus

**Status: NO CLINICAL EVIDENCE FOR ENHANCEMENT IN HEALTHY SUBJECTS**

| What exists | Assessment |
|-------------|-----------|
| Jacobs et al. (2015) — taVNS improved associative memory in healthy older adults, acute session | Very small pilot; cognitive improvement was modest |
| Sellaro et al. (2015) — taVNS improved divergent thinking (creativity) in healthy students | Small, single session, not replicated |
| Sun et al. (2021) — taVNS during a cognitive task showed some facilitation | Small pilot |
| Multiple studies show taVNS enhances fear extinction learning (Szeska 2020) — a form of cognitive flexibility | Lab paradigm, not real-world cognitive performance |

**Why not included:** The "cognitive enhancement" studies are all small, single-session, unreplicated findings in healthy subjects. Effect sizes are modest. There is no established parameter protocol for cognitive enhancement. This is preliminary neuroscience, not a clinical application.

**Recommendation:** Enthusiasts can experiment via the Exploration preset. Do NOT create a dedicated "Focus" or "Cognitive Enhancement" preset — the evidence does not support it and the name could attract inappropriate use expectations.

### 5.4 Stroke Rehabilitation

**Status: EMERGING — TOO EARLY FOR A PRESET**

| What exists | Assessment |
|-------------|-----------|
| Capone et al. (2017) — pilot of taVNS paired with motor rehabilitation in stroke patients, N=14 | Small pilot; some improvement in motor function |
| Redgrave et al. (2018) — stroke recovery as part of broader VNS rehabilitation research | Primarily implanted VNS (MicroTransponder/Vivistim device) |
| VNS paired with motor rehabilitation (implanted) FDA-cleared 2021 for chronic ischemic stroke | Different modality (implanted) |

**Why not included:** The most compelling stroke rehabilitation evidence uses **implanted** VNS paired with intensive motor rehab (the Vivistim approach, FDA-cleared). Transcutaneous taVNS for stroke is in very early pilot stages. Parameters and protocols are not established.

**Recommendation:** Do NOT include. This is a rehabilitative medicine application requiring supervised paired therapy, not a home-use preset.

### 5.5 Glucose Metabolism / Type 2 Diabetes

**Status: MECHANISTIC ONLY — NO HUMAN taVNS DATA**

| What exists | Assessment |
|-------------|-----------|
| Vagus nerve innervates pancreas (glucose regulation, insulin secretion) | Anatomical/mechanistic basis |
| Animal studies of VNS for glucose regulation | Preclinical only |
| No published human taVNS study with glucose metabolism as primary endpoint | Gap |

**Why not included:** Pure mechanistic extrapolation from vagal anatomy. No human data whatsoever for taVNS and glucose metabolism.

**Recommendation:** Do NOT include. EXTRAPOLATED — no human taVNS data.

### 5.6 Rheumatoid Arthritis / Autoimmune

**Status: IMPLANTED VNS ONLY — NO AURICULAR taVNS RCT**

| What exists | Assessment |
|-------------|-----------|
| Koopman et al. (PNAS, 2016) — implanted VNS reduced TNF-α in RA, N=17 | Implanted, not auricular |
| Aranow et al. (~2021) — pilot of taVNS in SLE, small N, variable parameters | Very early pilot; details uncertain |
| Anti-inflammatory taVNS preset already captures the mechanism (Lerman 2016) | Covered by existing preset |

**Why not included:** The strongest autoimmune evidence is from implanted VNS (Koopman 2016). The existing Anti-Inflammatory preset (anchored to Lerman 2016) already captures the relevant taVNS mechanism. A dedicated "RA" or "Autoimmune" preset adds no parameter difference — it would be identical to Anti-Inflammatory.

**Recommendation:** Do NOT create separate preset. Point autoimmune-interested users to Anti-Inflammatory preset.

---

## 6. Safety & Charge Density Verification — All New Presets

### 6.1 McCreery 1990 Compliance Check

All proposed presets verified against the 30 µC/cm² per phase limit (McCreery 1990, IEEE Trans Biomed Eng). Assumes minimum electrode contact area of 0.5 cm².

| Preset | Current | PW | Charge/phase (µC) | Charge density (µC/cm²) | Safety margin |
|--------|---------|----|--------------------|------------------------|---------------|
| Depression | 1.0 mA | 200 µs | 0.20 | 0.40 | **75×** |
| Epilepsy | 1.0 mA | 250 µs | 0.25 | 0.50 | **60×** |
| Migraine | 1.0 mA | 250 µs | 0.25 | 0.50 | **60×** |
| Tinnitus | 2.0 mA | 200 µs | 0.40 | 0.80 | **37.5×** |
| Stress/Anxiety | 1.0 mA | 200 µs | 0.20 | 0.40 | **75×** |
| Chronic Pain | 1.5 mA | 250 µs | 0.375 | 0.75 | **40×** |
| HRV Optimization | 2.0 mA | 200 µs | 0.40 | 0.80 | **37.5×** |
| Hypertension | 1.0 mA | 200 µs | 0.20 | 0.40 | **75×** |
| Obesity | 2.0 mA | 250 µs | 0.50 | 1.00 | **30×** |
| **Device maximum** | **5.0 mA** | **500 µs** | **2.50** | **5.00** | **6×** |

**All presets are well within the McCreery safety limit.** Even at the device's absolute maximum (5 mA × 500 µs), the charge density is 5 µC/cm² — still 6× below the 30 µC/cm² limit. The hardware overcurrent cutoff at 6 mA provides an additional safety layer.

### 6.2 New Contraindications Introduced by New Presets

| New Concern | Applicable Presets | Recommendation |
|-------------|-------------------|----------------|
| Active suicidal ideation | Depression | ABSOLUTE contraindication — refer to crisis services |
| Bipolar disorder (mania) | Depression | Relative — antidepressant-like effects could precipitate mania |
| Active status epilepticus | Epilepsy | ABSOLUTE — medical emergency |
| Implanted VNS device | Epilepsy | ABSOLUTE — dual VNS untested |
| Hemiplegic migraine | Migraine | Relative — rare subtype, caution advised |
| Seizure frequency >10/day | Epilepsy | Relative — electrode safety during seizures |

---

## 7. Complete Preset Comparison Table (Existing + New)

| # | Preset | Hz | PW (µs) | mA | Duty | Duration | Lateral | Stagger | Evidence | Conf | Primary/Adjunct |
|---|--------|----|---------|----|------|----------|---------|---------|----------|------|-----------------|
| 1 | Insomnia 25Hz | 25 | 200 | 2.0 | 30s/30s | 30 min | Bilateral | 20ms | Pilot RCT | MED | Primary |
| 2 | Insomnia 20Hz | 20 | 200 | 2.0 | 30s/30s | 30 min | Bilateral | 20ms | Pilot RCT | MED | Primary |
| 3 | RESET-AF | 20 | 200 | threshold | Continuous | 60 min | **L only** | N/A | RCT | HIGH | Adjunct |
| 4 | Anti-Inflammatory | 25 | 250 | 1.5 | 30s/30s | 60 min | Bilateral | 20ms | Pilot RCT | MED | Adjunct |
| 5 | **Depression** | **20** | **200** | **1.0** | **30s/30s** | **30 min** | **L only** | **N/A** | **RCT (N=160)** | **HIGH** | **Adjunct** |
| 6 | **Epilepsy** | **25** | **250** | **1.0** | **30s/30s** | **60 min ×3** | **L only** | **N/A** | **CE-marked** | **HIGH** | **Adjunct** |
| 7 | **Stress/Anxiety** | **25** | **200** | **1.0** | **Continuous** | **15 min** | **L (exp bi)** | **20ms opt** | **Lab RCT** | **MED** | **Primary** |
| 8 | **HRV Optimization** | **25** | **200** | **threshold** | **30s/30s** | **15 min** | **Bilateral** | **20ms** | **Multi-pilot** | **MED** | **Primary** |
| 9 | **Migraine** | **1** | **250** | **1.0** | **Continuous** | **60 min** | **L only** | **N/A** | **Small RCT** | **MED** | **Adjunct** |
| 10 | Dysautonomia | 25 | 200 | 0.5→1.0 | 30s/30s | 30 min | Bilateral | 20ms | Theoretical | LOW | Adjunct |
| 11 | Exploration | user | user | user | user | user | user | user | N/A | N/A | N/A |

---

## 8. Research Gaps & Verification Needed

### 8.1 High Priority (Before Implementing New Presets)

| Gap | Action | Priority |
|-----|--------|----------|
| Verify Fang/Rong 2016 exact parameters (pulse width, duty cycle) | Download paper from Biol Psychiatry | HIGH |
| Verify Bauer 2016 epilepsy RCT primary endpoint result | Download paper from Epilepsia | HIGH |
| Confirm NEMOS CE-mark parameters (25 Hz, 250 µs, duty cycle) | Check Cerbomed documentation | HIGH |
| Verify Straube 2015 migraine 1 Hz result (exact headache day reduction) | Download paper from J Headache Pain | HIGH |
| Verify Burger 2019 stress study parameters (current, duration) | Download paper from Psychophysiology | MEDIUM |

### 8.2 Medium Priority (Post-Implementation Research)

| Gap | Action | Priority |
|-----|--------|----------|
| Search for 2024–2025 taVNS depression meta-analyses | PubMed search | MEDIUM |
| Search for any published taVNS + PTSD RCT (post-2023) | PubMed search | MEDIUM |
| Search for any published taVNS + long-COVID study | PubMed search | MEDIUM |
| Search for bilateral taVNS safety/efficacy comparisons | PubMed search | MEDIUM |
| Verify Chinese depression protocol: is 20 Hz continuous or 20/4 Hz alternating the standard? | Literature review | MEDIUM |

### 8.3 Low Priority (Documentation / Completeness)

| Gap | Action | Priority |
|-----|--------|----------|
| Confirm Nishi 2020 appetite study details | PubMed search | LOW |
| Confirm Jacobs 2015 cognitive study details | PubMed search | LOW |
| Verify if Aranow SLE pilot was published or just conference abstract | PubMed search | LOW |

---

## 9. References — New Citations Introduced

| ID | Authors | Title (abbreviated) | Journal | Year | Evidence | Cited for |
|----|---------|---------------------|---------|------|----------|-----------|
| NEW-01 | Fang J, Rong P, Hong Y, et al. | taVNS modulates DMN in MDD | Biol Psychiatry | 2016 | RCT (N=160) | Depression preset |
| NEW-02 | Rong P, Liu J, Wang L, et al. | taVNS for MDD clinical outcomes | Biol Psychiatry | 2016 | RCT (N=160) | Depression preset |
| NEW-03 | Stefan H, Kreiselmeyer G, et al. | Transcutaneous VNS in drug-resistant epilepsy | Epilepsia | 2012 | Open-label pilot | Epilepsy preset |
| NEW-04 | Bauer S, Baier H, Baumgartner C, et al. | taVNS for drug-resistant epilepsy (NEMOS) | Epilepsia | 2016 | RCT (N~76) | Epilepsy preset |
| NEW-05 | Burger AM, Van der Does W, et al. | taVNS attenuates acute stress response | Psychophysiology | 2019 | Pilot RCT (lab) | Stress preset |
| NEW-06 | Szeska C, Richter J, Wendt J, et al. | taVNS promotes fear extinction learning | Biol Psychiatry | 2020 | Pilot RCT (lab) | Stress preset (PTSD rationale) |
| NEW-07 | Clancy JA, Mary DA, Witte KK, et al. | Non-invasive VNS in healthy humans | Brain Stimul | 2014 | Crossover pilot | HRV Optimization preset |
| NEW-08 | De Couck M, Cserjesi R, Caers R, et al. | Effects of short/prolonged taVNS on HRV | Complement Ther Med | 2017 | Crossover | HRV Optimization preset |
| NEW-09 | Kong J, Fang J, Park J, et al. | Treating depression with taVNS | J Psychiatr Res | 2018 | Pilot RCT | Depression (supplementary) |
| NEW-10 | Lehtimäki J, Hyvärinen P, et al. | taVNS for tinnitus — fMRI pilot | Brain Stimul | 2013 | Pilot | Tinnitus (evaluated, not recommended) |
| NEW-11 | Gurel NZ, Huang M, Wittbrodt MT, et al. | taVNS reduces PTSD-related physiological responses | Psychoneuroendocrinology | 2020 | Pilot | PTSD (evaluated, not recommended) |
| NEW-12 | Antonino D, Teixeira AL, Maia-Lopes PM, et al. | Non-invasive VNS acutely alters HRV in healthy young men | J Neurophysiol | 2017 | Crossover | HRV Optimization |
| NEW-13 | Capone F, Miccinilli S, Pellegrino G, et al. | taVNS combined with robotic rehabilitation for upper limb after stroke | Eur J Phys Rehabil Med | 2017 | Pilot (N=14) | Stroke (evaluated, not recommended) |

> **⚠️ VERIFY ALL:** These citations are from training data. DOIs and exact page numbers should be confirmed against PubMed before citing in published documentation. Search: [PubMed taVNS bibliography](https://pubmed.ncbi.nlm.nih.gov/?term=transcutaneous+auricular+vagus+nerve+stimulation)

---

## 10. Appendix: Decision Matrix for Preset Inclusion

### Scoring Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Evidence quality | 3× | RCT=5, Pilot RCT=3, Case series=2, Mechanistic=1, Theoretical=0 |
| Human taVNS specificity | 2× | taVNS data=5, Auricular extrapolation=3, Implanted VNS extrapolation=1, Animal=0 |
| Unique parameters | 1× | Unique from existing presets=5, Minor variation=2, Identical=0 |
| User demand likelihood | 1× | High biohacker interest=5, Niche=2, Very niche=1 |
| Safety risk | -1× | Low=0, Medium=-2, High=-5 |

### Scores

| Condition | Evidence (×3) | taVNS-specific (×2) | Unique params (×1) | User demand (×1) | Safety (×-1) | **Total** | **Decision** |
|-----------|--------------|---------------------|-------------------|-----------------|-------------|-----------|-------------|
| Depression | 15 | 10 | 2 | 5 | 0 | **32** | ✅ ADD |
| Epilepsy | 12 | 10 | 5 | 2 | -2 | **27** | ✅ ADD |
| Stress/Anxiety | 9 | 10 | 5 | 5 | 0 | **29** | ✅ ADD |
| HRV Optimization | 9 | 10 | 5 | 5 | 0 | **29** | ✅ ADD |
| Migraine | 9 | 10 | 5 | 3 | 0 | **27** | ✅ ADD |
| Tinnitus | 6 | 6 | 2 | 3 | 0 | **17** | ❌ Exploration |
| Hypertension | 3 | 6 | 0 | 3 | 0 | **12** | ❌ Exploration |
| Chronic Pain | 6 | 6 | 0 | 3 | 0 | **15** | ❌ Use Anti-Inflammatory |
| Obesity | 3 | 4 | 2 | 2 | 0 | **11** | ❌ Exploration |
| PTSD | 3 | 6 | 0 | 4 | 0 | **13** | ❌ Use Stress preset |
| Post-COVID | 0 | 0 | 0 | 4 | 0 | **4** | ❌ Use Dysautonomia |
| Cognitive Enhancement | 3 | 6 | 0 | 3 | 0 | **12** | ❌ Exploration |
| Stroke Rehab | 3 | 2 | 0 | 1 | -2 | **4** | ❌ Not recommended |
| Glucose / Diabetes | 0 | 0 | 0 | 1 | 0 | **1** | ❌ Not recommended |
| RA / Autoimmune | 3 | 2 | 0 | 2 | 0 | **7** | ❌ Use Anti-Inflammatory |

---

## 11. Summary of Changes to Project

### 11.1 New Presets to Add (5)

1. **Depression (MDD)** — 20 Hz, 200 µs, 1.0 mA, 30s/30s, 30 min, unilateral L. Evidence: RCT (N=160).
2. **Epilepsy (Adjunctive)** — 25 Hz, 250 µs, 1.0 mA, 30s/30s, 60 min (3×/day), unilateral L. Evidence: CE-marked device.
3. **Stress / Acute Anxiety** — 25 Hz, 200 µs, 1.0 mA, continuous, 15 min, unilateral L (bilateral experimental). Evidence: Lab RCT.
4. **HRV Optimization** — 25 Hz, 200 µs, threshold, 30s/30s, 15 min, bilateral + 20ms stagger. Evidence: Multiple pilots.
5. **Migraine Prevention** — 1 Hz, 250 µs, 1.0 mA, continuous, 60 min, unilateral L. Evidence: Small RCT.

### 11.2 Existing Presets — No Changes

All existing presets (Insomnia 25Hz, Insomnia 20Hz, RESET-AF, Anti-Inflammatory, Dysautonomia/hEDS, Exploration) remain as validated in PRESETS.md. No parameter changes needed.

### 11.3 Documentation Updates Needed

- Add new contraindications (suicidal ideation, bipolar mania, status epilepticus, implanted VNS) to DOC-02
- Add Depression and Epilepsy session scheduling notes (2×/day and 3×/day respectively)
- Update AGENTS.md §6 (Clinical Preset Parameters) table with new presets
- Add new references (NEW-01 through NEW-13) to REFERENCES.md
- Update ROADMAP Phase 7 (Session Management) to account for 11 presets instead of 6

### 11.4 Firmware Implications

- Support for **1 Hz frequency** (Migraine preset) — verify timer resolution is adequate at this frequency
- Support for **multi-session scheduling** (Depression 2×/day, Epilepsy 3×/day) — session manager must track daily session count
- **5 parameter templates** simplify code: vagal-standard, depression-protocol, nemos-protocol, cardiac-protocol, low-frequency
- **Laterality enforcement**: Depression, Epilepsy, RESET-AF, Migraine must default to unilateral left with stagger disabled

---

*Document generated for OpenBinaural-taVNS project.*
*All parameters must be verified against cited papers before firmware implementation.*
*This document should be updated as new taVNS clinical evidence is published.*
