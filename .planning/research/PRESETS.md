# Clinical Evidence Review: taVNS Stimulation Presets

**Project:** OpenBinaural-taVNS
**Researched:** 2025-07-13
**Scope:** Parameter validation for 4 firmware presets (FW-08) + contraindications (DOC-02)
**Source methodology:** Clinical training data (literature through early 2025). No live web search was available. All citations are from training data memory of published literature. Confidence levels are assigned conservatively.

> **IMPORTANT LIMITATION:** This review is compiled from training data, not real-time literature search. All HIGH confidence claims reflect well-established findings replicated across multiple published studies. MEDIUM confidence claims reflect my recall of specific papers with reasonable certainty. LOW confidence claims should be verified against PubMed/Google Scholar before finalizing firmware parameters. **In a medical device context, every parameter should be independently verified against the cited papers before implementation.**

---

## 1. Executive Summary

### Overall Evidence Quality by Preset

| Preset | Evidence Level | Confidence | Summary |
|--------|---------------|------------|---------|
| Insomnia / Sleep | Pilot study supported | MEDIUM | Several small-medium RCTs (primarily Chinese groups), no large multicenter RCT for insomnia specifically. Parameters extrapolated from cardiac/autonomic taVNS literature. |
| Chronic Pain / Anti-Inflammatory | Pilot study supported (pain); Mechanistic evidence (anti-inflammatory) | MEDIUM | Pain: small RCTs and pilot studies. Anti-inflammatory pathway: well-established for implanted VNS; emerging but limited evidence for auricular taVNS specifically. |
| Dysautonomia / hEDS | Theoretical / Experimental | LOW | No published RCTs for taVNS in POTS or hEDS. Theoretical rationale from autonomic physiology. Anecdotal community reports. Should carry strongest experimental warning. |
| RESET-AF Replication | RCT-validated | HIGH | Based on published RCTs (RESET-AF, TREAT-AF) with well-documented protocols. Best-evidenced preset. |

### Key Adjustments to Current Draft Parameters

| Preset | Parameter | Current Draft | Recommended | Reason | Severity |
|--------|-----------|---------------|-------------|--------|----------|
| Chronic Pain | Frequency | 10 Hz | **25 Hz** | ⚠️ **ADJUSTMENT NEEDED** — The 10 Hz choice lacks taVNS-specific human evidence. The strongest published taVNS anti-inflammatory data (Lerman et al. 2016) used 25 Hz. | HIGH |
| RESET-AF | Duty cycle | "continuous" | **Continuous confirmed** | No change needed, but verify against published protocol | — |
| RESET-AF | Bilateral | "synchronous" | **Unilateral (left ear)** | ⚠️ **ADJUSTMENT NEEDED** — RESET-AF used unilateral left tragus stimulation. If replicating the trial, should default to unilateral. Bilateral option can be available but is NOT the replicated protocol. | MEDIUM |
| RESET-AF | Stim site | (implied cymba) | **Tragus** | ⚠️ **DOCUMENTATION NOTE** — RESET-AF stimulated the tragus, not cymba conchae. The device targets cymba conchae by default. UI should note this discrepancy. | MEDIUM |
| Dysautonomia | Default current | 1.0 mA | **0.5 mA** (with ramp to max 1.0 mA) | ⚠️ **ADJUSTMENT NEEDED** — hEDS patients have documented sensory hypersensitivity. Lower starting intensity is prudent. | MEDIUM |
| ALL | Bilateral stagger | 20 ms | 20 ms (no change) | The bilateral asynchronous stagger is experimental and not validated by any published study. Keep as project hypothesis but document clearly. | — |

---

## 2. Preset 1: Insomnia / Sleep Quality

### 2.1 Evidence Summary

| Study / Source | Year | Design | n | Parameters | Key Outcomes | Evidence Level | Confidence |
|----------------|------|--------|---|------------|-------------|----------------|------------|
| **Jiao et al.** (likely Evid Based Complement Alternat Med or similar) | 2020 | RCT | ~60 | Left cymba conchae, 20 Hz, ~1 mA, 30 min sessions, 4 weeks | Improved PSQI scores vs. sham | Pilot RCT | MEDIUM — I recall this study but cannot verify exact journal/n |
| **Li et al.** (Chinese research group) | 2019–2021 | RCT | ~40–80 | Left ear, 20 Hz, 200 µs, 1–2 mA, 30 min/day, 4–8 weeks | Improved PSQI, sleep latency reduced | Pilot RCT | MEDIUM — Multiple Chinese studies, exact details need verification |
| **Zhao et al.** | 2019–2020 | RCT | ~60 | Left auricular branch, frequency 4/20 Hz alternating or continuous 20 Hz, 30 min | Improved sleep quality measures | Pilot RCT | LOW — Less certain about this specific study |
| **Bretherton et al.** (Aging, 2019) | 2019 | Pilot RCT | 29 (elderly) | Tragus, 200 µs, 200 µA, 15 min/day, 2 weeks | Improved autonomic balance (HRV); sleep quality was secondary outcome showing trends | Pilot | MEDIUM |
| **Nuerisym case reports** (UK, company-published) | 2023–2024 | Case series | Small | Bilateral, 25 Hz, 250 µs, variable intensity, 30–60 min | Anecdotal improvements in insomnia, anxiety, autonomic symptoms | Case series | LOW — Company-reported, not peer-reviewed independent |
| **Fang et al.** (Brain Stimulation) | 2016 | RCT | 160 | Left cymba conchae, 20 Hz/4 Hz alternating, 1 mA, 30 min, 4 weeks | Primary: depression; secondary: sleep improvement noted | Pilot RCT (insomnia as secondary) | MEDIUM — Well-known depression study, sleep improvement was a secondary finding |

**Summary of insomnia evidence:**
- There are 5–10 published studies (2016–2024) examining taVNS for insomnia or sleep quality, primarily from Chinese research groups
- Most are small-medium RCTs (n = 30–80) or pilot studies
- No large (n > 200) multicenter RCT exists specifically for taVNS and insomnia
- The most commonly used parameters across insomnia studies: **20 Hz, 200 µs, 1–2 mA, 30 min sessions, left ear**
- Several studies used a 4/20 Hz alternating protocol (traditional Chinese medicine rationale — not necessarily better than fixed 20–25 Hz)
- PSQI (Pittsburgh Sleep Quality Index) is the standard primary outcome
- Most studies show statistically significant PSQI improvement over 4–8 weeks
- Acute single-session HRV improvements (↑ RMSSD, ↑ HF power) are consistently demonstrated

### 2.2 Parameter Analysis

**Frequency: 25 Hz vs. 20 Hz**
- Most insomnia-specific studies used **20 Hz**
- The NEMOS device (Cerbomed, most-studied taVNS device) defaults to **25 Hz** — but most NEMOS studies targeted depression/epilepsy, not insomnia
- The Nuerisym device uses **25 Hz**
- Both 20 Hz and 25 Hz fall within the Aβ-fiber activation range for vagal afferents
- **Recommendation:** Either 20 Hz or 25 Hz is defensible. **Keep 25 Hz** for consistency with the NEMOS evidence base and the majority of autonomic taVNS literature. The 5 Hz difference is unlikely to be clinically meaningful for vagal afferent activation, and 25 Hz has broader support across non-insomnia taVNS studies.
- **Confidence:** MEDIUM — No head-to-head comparison of 20 vs. 25 Hz for insomnia exists.

**Pulse Width: 200 µs vs. 250 µs**
- RESET-AF and most cardiac studies used **200 µs**
- NEMOS uses **250 µs**
- Chinese insomnia studies mostly used **200 µs**
- Both are well within safe charge limits (at 2 mA: 200 µs = 0.4 µC, 250 µs = 0.5 µC — both far below McCreery limits)
- Slightly wider pulses recruit more Aβ fibers but also approach Aδ recruitment at higher intensities
- **Recommendation:** **Keep 200 µs.** This matches the majority of the insomnia-specific literature and provides a more conservative charge-per-phase.
- **Confidence:** MEDIUM

**Intensity: 2.0 mA fixed vs. threshold-based**
- All clinical studies titrate to individual perception threshold, not a fixed current
- "Perception threshold" = lowest current at which the subject reports a tingling/pulsing sensation
- Typical perception thresholds range 0.5–3 mA (varies by skin impedance, electrode contact, individual)
- Fixed 2.0 mA may be above or below threshold for different users
- **Recommendation:** **Default to 2.0 mA but UI must guide titration.** The preset should set 2.0 mA as a starting point, with clear instruction: "Adjust to comfortable tingling below pain threshold." The firmware allows 0.1 mA resolution per FW-02.
- **Confidence:** HIGH — Threshold-based titration is universal in the literature.

**Duty Cycle: 30s ON / 30s OFF**
- This is the most common duty cycle across taVNS studies
- Some insomnia studies used continuous stimulation
- The 30s/30s duty cycle is believed to prevent neural adaptation/habituation
- No head-to-head comparison of duty cycles for insomnia
- **Recommendation:** **Keep 30s ON / 30s OFF.** Well-supported across the literature.
- **Confidence:** MEDIUM

**Session Duration: 30 min**
- Most insomnia studies used **30 min**
- RESET-AF used **60 min**
- Acute HRV response (↑ RMSSD) appears within 15–20 min of stimulation
- For insomnia (nightly use, long-term), 30 min is more practical for compliance
- **Recommendation:** **Keep 30 min default.** Consider offering 60 min as an option in Exploration mode.
- **Confidence:** MEDIUM

**Bilateral vs. Unilateral**
- **Almost all published insomnia/sleep studies used unilateral (left ear) stimulation**
- Left ear preference comes from implanted VNS convention (left vagus has less direct cardiac efferent innervation) — though this concern is less relevant for auricular AFFERENT stimulation
- Bilateral auricular stimulation is not well-studied in published RCTs for any condition
- The Nuerisym device offers bilateral but published evidence is limited
- **Recommendation:** The bilateral 20 ms stagger is the project's experimental hypothesis. **Keep it, but document clearly that no published RCT validates bilateral for insomnia.** Unilateral left ear should be available as an alternative preset or toggle.
- **Confidence:** LOW for bilateral advantage; HIGH for unilateral safety

**Timing: Before Sleep**
- No RCT has specifically tested timing (30 min pre-sleep vs. morning vs. afternoon) for insomnia
- Parasympathetic enhancement is acute (onset 5–15 min, peak 15–30 min)
- Chronobiology supports evening use: promoting parasympathetic shift at natural circadian transition to sleep
- Some studies administered taVNS in the afternoon (clinical visit convenience) and still saw nightly sleep improvements — suggesting cumulative effect over weeks
- **Recommendation:** "30–60 min before bedtime" is a reasonable clinical recommendation for the UI but not evidence-mandated.
- **Confidence:** LOW — Sensible but unvalidated

### 2.3 Recommended Final Parameters

| Parameter | Value | Change from Draft? |
|-----------|-------|--------------------|
| Frequency | **25 Hz** | No change |
| Pulse width | **200 µs** | No change |
| Default current | **2.0 mA** (titrate to perception threshold) | No change (add titration guidance in UI) |
| Duty cycle | **30s ON / 30s OFF** | No change |
| Session duration | **30 min** | No change |
| Bilateral stagger | **20 ms** | No change (flag as experimental) |
| Ramp | **0 → target over 30s** | No change |

### 2.4 Evidence Level

**"Pilot Study Supported"** — Multiple small-medium RCTs demonstrate efficacy for sleep quality improvement. Parameters are well-aligned with published protocols. No large multicenter RCT for insomnia specifically.

### 2.5 UI Warning Label Recommendation

```
Insomnia / Sleep Quality
─────────────────────────
Evidence: Pilot studies support taVNS for sleep quality improvement.
Parameters based on published clinical protocols (20–25 Hz, 200 µs, sub-pain threshold).
Bilateral stimulation is experimental (published studies used unilateral left ear).

⚠ Not a medical device. Not a substitute for medical treatment of sleep disorders.
Adjust intensity to comfortable tingling below pain threshold.
Recommended: Use 30–60 minutes before bedtime.
```

### 2.6 Insomnia-Specific Contraindications

| Contraindication | Type | Rationale |
|-----------------|------|-----------|
| Untreated sleep apnea | Relative | Vagal stimulation may theoretically increase upper airway collapsibility. CPAP should remain primary treatment. No documented adverse events, but theoretical concern. |
| Benzodiazepine / z-drug concurrent use | Relative | Additive parasympathetic/sedative effects. Not dangerous but may confound self-experimentation results. |
| SSRI/SNRI concurrent use | Informational | taVNS affects serotonin pathways via NTS → dorsal raphe. No documented adverse interaction, but may confound efficacy assessment. |

---

## 3. Preset 2: Chronic Pain / Anti-Inflammatory

### 3.1 Evidence Summary

#### 3.1.1 Anti-Inflammatory Pathway Evidence

| Study / Source | Year | Design | n | Parameters | Key Outcomes | Evidence Level | Confidence |
|----------------|------|--------|---|------------|-------------|----------------|------------|
| **Tracey KJ** (Nature) | 2002 | Mechanistic review | — | Implanted VNS (animal models) | Described the "inflammatory reflex" — vagus nerve → celiac ganglion → splenic nerve → α7 nAChR on macrophages → TNF-α suppression | Foundational mechanism | HIGH — Landmark paper |
| **Tracey KJ** (J Clin Invest) | 2007 | Review | — | — | Expanded cholinergic anti-inflammatory pathway framework | Foundational mechanism | HIGH |
| **Koopman et al.** (PNAS) | 2016 | Open-label pilot | 17 (RA) | **Implanted cervical VNS**, not taVNS | Reduced TNF-α, improved DAS28 in rheumatoid arthritis | Implanted VNS only | HIGH — But different modality (implanted, not auricular) |
| **Lerman et al.** (Mol Med) | 2016 | Pilot (healthy volunteers) | ~20 | **taVNS, left cymba conchae, 25 Hz, 250 µs**, sub-threshold intensity, continuous, acute session | Reduced TNF-α production in endotoxin-stimulated whole blood | Pilot study — taVNS specific | MEDIUM — Key paper linking auricular taVNS to anti-inflammatory |
| **Stavrakis et al.** (JACC CE) | 2020 | RCT (AF) | 53 | Tragus, 20 Hz, 200 µs, threshold intensity | Reduced CRP and inflammatory markers (secondary outcomes in AF trial) | RCT (secondary outcome) | MEDIUM |
| **Aranow et al.** (Arthritis Rheumatol?) | 2021 | Pilot | Small (~15?) | taVNS in SLE (systemic lupus), parameters variable | Reduction in pain, fatigue; some biomarker changes | Pilot | LOW — Recall uncertain on exact details |

**Critical gap in the draft parameters:** The current FW-08 draft specifies **10 Hz** for the anti-inflammatory preset. However:

1. **Lerman et al. (2016)** — the only published study demonstrating taVNS-specific anti-inflammatory effects (reduced TNF-α) — used **25 Hz**, not 10 Hz.
2. **No published human taVNS study has demonstrated anti-inflammatory effects at 10 Hz.**
3. The 10 Hz figure likely comes from implanted cervical VNS animal studies, where lower frequencies (5–10 Hz) showed anti-inflammatory effects via direct efferent fiber activation. However, auricular taVNS activates **afferent** fibers, not efferent. The optimal frequency for afferent activation (to trigger the reflex arc: afferent → NTS → efferent → spleen) may differ.
4. At 10 Hz with 250 µs pulse width, only 0.5% of each second contains a stimulus pulse. At 25 Hz, this rises to 1.25% — providing more afferent drive to the NTS per unit time.

**⚠️ ADJUSTMENT NEEDED: Change frequency from 10 Hz to 25 Hz for the anti-inflammatory preset, or document 10 Hz as "theoretical, extrapolated from implanted VNS" and provide 25 Hz as the evidence-based alternative.**

#### 3.1.2 Chronic Pain Evidence

| Study / Source | Year | Design | n | Parameters | Key Outcomes | Evidence Level | Confidence |
|----------------|------|--------|---|------------|-------------|----------------|------------|
| **Napadow et al.** (Pain) | 2012 | fMRI crossover | ~15 (fibromyalgia) | taVNS at cymba conchae, 25 Hz, 250 µs, continuous, acute | Modulated pain-processing brain regions (anterior insula, ACC); trend toward reduced pain ratings | Mechanistic pilot | MEDIUM |
| **Busch et al.** (J Pain) | 2013 | Pilot | ~20 (chronic pelvic pain) | taVNS, variable parameters | Some pain reduction | Pilot | LOW — Less certain on details |
| **Straube et al.** (J Headache Pain) | 2015 | RCT | ~46 (migraine) | taVNS, cymba conchae, 1 Hz and 25 Hz arms, 250 µs, 1 mA | Reduced migraine frequency in 1 Hz arm (counterintuitively); 25 Hz arm also showed some benefit | Small RCT | MEDIUM |
| **Silberstein et al.** (various) | 2016+ | Multiple | Various | **gammaCore (non-invasive cervical VNS)** — note: this is CERVICAL, not auricular | Cluster headache and migraine prevention/acute treatment | RCT (but different modality) | Not directly applicable — cervical, not auricular |
| **Kutlu et al.** | 2020 | Pilot | Small | taVNS for fibromyalgia, standard parameters | Pain score improvements | Pilot | LOW |
| **Johnson et al.** (meta-analysis?) | 2022 | Systematic review | — | Review of VNS/taVNS for pain conditions | Modest evidence for taVNS in chronic pain; more evidence for implanted VNS | Review | LOW — Uncertain on exact citation |

**Pain evidence summary:**
- Evidence for taVNS in chronic pain is modest: several small pilot studies and one or two small RCTs
- No large RCT has demonstrated definitive efficacy of taVNS for fibromyalgia, neuropathic pain, or chronic widespread pain
- The Napadow fMRI study (2012) provides mechanistic plausibility (taVNS modulates central pain-processing circuits)
- gammaCore (cervical VNS) has FDA clearance for migraine and cluster headache, but cervical ≠ auricular
- The anti-inflammatory mechanism (vagal → splenic → α7 nAChR → TNF-α suppression) is well-established for implanted VNS but remains incompletely validated for auricular taVNS

### 3.2 Parameter Analysis

**Frequency: 10 Hz → 25 Hz (ADJUSTMENT NEEDED)**
- 10 Hz is not supported by published human taVNS anti-inflammatory data
- Lerman et al. (2016) used 25 Hz and demonstrated TNF-α reduction — this is the strongest available evidence
- For chronic pain (Napadow et al.), 25 Hz was also used
- The Straube migraine study included a 1 Hz arm that showed benefit — but this was for migraine prevention specifically and hasn't been replicated
- **Recommendation:** **Change to 25 Hz.** If the project wishes to include a 10 Hz option, it should be labeled "Experimental — extrapolated from implanted VNS studies" and available only through Exploration mode.
- **Confidence:** MEDIUM for 25 Hz being better than 10 Hz for auricular anti-inflammatory taVNS

**Pulse Width: 250 µs**
- Lerman et al. used **250 µs**
- Napadow et al. used **250 µs**
- The NEMOS device uses **250 µs**
- 250 µs provides slightly more charge per phase (at same current) — may recruit more Aβ fibers
- **Recommendation:** **Keep 250 µs.** This matches the anti-inflammatory and pain literature.
- **Confidence:** MEDIUM

**Intensity: 1.5 mA**
- Studies typically titrate to sub-pain perception threshold
- 1.5 mA is a reasonable default for chronic pain patients (who may have altered pain thresholds)
- **Recommendation:** **Keep 1.5 mA as default, with titration guidance.** Chronic pain patients may have both lower (hyperalgesia) and higher (opioid/gabapentin use) thresholds depending on medication status.
- **Confidence:** MEDIUM

**Session Duration: 60 min**
- Lerman et al. used acute single sessions
- RESET-AF used 60 min daily for 6 months
- For anti-inflammatory effects, longer exposure may be needed (the inflammatory reflex requires sustained vagal afferent input)
- **Recommendation:** **Keep 60 min.** Anti-inflammatory effects likely require longer exposure than parasympathetic HRV shifts.
- **Confidence:** LOW — No dose-response study for session duration in anti-inflammatory taVNS

**Duty Cycle: 30s ON / 30s OFF**
- Lerman et al. may have used continuous (need to verify)
- 30s/30s is the most common in the general taVNS literature
- **Recommendation:** **Keep 30s ON / 30s OFF.** Standard protocol.
- **Confidence:** MEDIUM

### 3.3 Recommended Final Parameters

| Parameter | Value | Change from Draft? |
|-----------|-------|--------------------|
| Frequency | **25 Hz** | ⚠️ **CHANGED from 10 Hz** |
| Pulse width | **250 µs** | No change |
| Default current | **1.5 mA** (titrate to perception threshold) | No change |
| Duty cycle | **30s ON / 30s OFF** | No change |
| Session duration | **60 min** | No change |
| Bilateral stagger | **20 ms** | No change (experimental) |
| Ramp | **0 → target over 30s** | No change |

### 3.4 Evidence Level

**"Pilot Study Supported (pain) / Mechanistic Evidence (anti-inflammatory)"** — Small pilot studies and fMRI data support taVNS modulation of pain processing. Anti-inflammatory pathway is well-established for implanted VNS but only pilot-level evidence for auricular taVNS. No large RCT for either pain or anti-inflammatory endpoints with auricular taVNS.

### 3.5 UI Warning Label Recommendation

```
Chronic Pain / Anti-Inflammatory
─────────────────────────────────
Evidence: Pilot studies support taVNS for pain modulation.
Anti-inflammatory mechanism established for implanted VNS; emerging evidence for auricular taVNS.
Parameters: 25 Hz, 250 µs, 60-min sessions based on published protocols.

⚠ Not a medical device. Not a substitute for prescribed pain management.
Do not discontinue prescribed medications based on taVNS use.
If on opioids or anticoagulants, consult your physician before use.
```

### 3.6 Chronic Pain-Specific Contraindications

| Contraindication | Type | Rationale |
|-----------------|------|-----------|
| Opioid use (high dose) | Relative | Opioids have vagotonic effects. Combined with taVNS vagal activation, theoretical risk of excessive bradycardia. Monitor HR during initial sessions. |
| Anticoagulant use (warfarin, DOACs) | Relative | Not a taVNS-specific interaction, but chronic pain patients on anticoagulants may have skin integrity concerns at electrode site. Ensure no broken skin. |
| NSAIDs (concurrent) | Informational | No interaction. NSAIDs act peripherally on COX pathway; taVNS acts via central vagal reflex. Safe to combine. |
| Gabapentin / pregabalin | Informational | No documented interaction. Both commonly co-used in chronic pain populations. |
| Fibromyalgia with severe allodynia | Relative | Auricular electrode contact may be uncomfortable. Start at lower current (0.5 mA) and titrate slowly. |

---

## 4. Preset 3: Dysautonomia / hEDS

### 4.1 Evidence Summary

| Study / Source | Year | Design | n | Parameters | Key Outcomes | Evidence Level | Confidence |
|----------------|------|--------|---|------------|-------------|----------------|------------|
| **No published RCT for taVNS in POTS or hEDS** | — | — | — | — | — | — | HIGH confidence that no RCT exists (as of training data cutoff) |
| **No published pilot study specifically for taVNS in hEDS** | — | — | — | — | — | — | HIGH confidence that no pilot exists |
| **Stavrakis / cardiac taVNS literature** | 2015–2023 | Various | Various | 20–25 Hz, 200 µs | Demonstrates taVNS improves cardiac autonomic balance (HRV, reduced AF burden) | RCT (different condition) | HIGH for autonomic modulation mechanism |
| **Bretherton et al.** (Aging) | 2019 | Pilot RCT | 29 (elderly) | Tragus, 200 µs, 200 µA, 15 min/day | Improved autonomic balance (shift toward parasympathetic) in older adults | Pilot (different population) | MEDIUM |
| **Nuerisym anecdotal reports** | 2023–2024 | Case reports / testimonials | Unknown | Bilateral, 25 Hz, 250 µs | Reported improvements in POTS symptoms (tachycardia, exercise tolerance, fatigue) | Anecdotal / manufacturer reports | LOW — Not peer-reviewed |
| **General taVNS → HRV literature** | 2015–2024 | Multiple studies | Multiple | Various | taVNS consistently increases RMSSD, HF power (parasympathetic indices) and reduces LF/HF ratio | Consistent across studies | HIGH for autonomic mechanism |

### 4.2 Theoretical Rationale Assessment

**Is the mechanism linking taVNS → vagal tone → POTS symptom improvement mechanistically sound?**

**YES, with caveats:**

1. **Sound part:** POTS (particularly hyperadrenergic POTS) involves sympathetic overdrive and reduced vagal tone. taVNS has been consistently shown to increase parasympathetic indices (↑ RMSSD, ↑ HF-HRV) and reduce sympathetic markers. Restoring autonomic balance is a logical therapeutic target.

2. **Caveats:**
   - POTS is heterogeneous: hyperadrenergic POTS, neuropathic POTS, and hypovolemic POTS have different underlying mechanisms. taVNS would most logically help hyperadrenergic POTS (where reducing sympathetic drive matters). It may have limited benefit in neuropathic POTS (where the problem is peripheral autonomic neuropathy, not central autonomic imbalance) or hypovolemic POTS (where the problem is inadequate blood volume).
   - **Risk of worsening:** In some POTS patients, particularly those who experience vagal episodes (vasovagal syncope-prone), increasing vagal tone could theoretically worsen presyncope symptoms. However, taVNS is a low-level afferent stimulation, not a massive vagal surge. Published studies in other populations have not reported syncope as an adverse event. The risk is theoretical but should be documented.
   - hEDS patients often have autonomic dysfunction beyond just POTS — they may have abnormal baroreceptor sensitivity, mast cell activation, and generalized connective tissue-related nerve abnormalities. taVNS would only address the autonomic regulation component.

3. **Bottom line:** The mechanism is plausible for hyperadrenergic POTS. There is zero direct clinical evidence. This preset is purely theoretical and should be labeled accordingly.

### 4.3 Safety Concerns for hEDS/POTS Population

| Concern | Assessment | Mitigation |
|---------|-----------|------------|
| **Sensory hypersensitivity** | hEDS patients frequently report allodynia, hyperalgesia, and generalized sensory sensitivity. Standard taVNS current levels may be perceived as painful. | Start at **0.5 mA** (not 1.0 mA). Use slower ramp (45–60s instead of 30s). Allow user titration down to 0.1 mA. |
| **Vasovagal syncope risk** | Some POTS patients have vasovagal tendencies. Vagal stimulation could theoretically trigger a vasovagal response in susceptible individuals. | **Use seated or recumbent position only.** Never use while standing. First session should be supervised. Include HR monitoring guidance (Polar H10). If HR drops >20 bpm during session, stop. |
| **Cardiac involvement** | hEDS patients may have: mitral valve prolapse (MVP), aortic root dilation, or other structural cardiac findings. | **MVP and mild aortic dilation are NOT contraindications to taVNS** (auricular stimulation has minimal direct cardiac effect). However, any patient with documented cardiac arrhythmia should consult cardiology first. If MVP is symptomatic or if aortic dilation is approaching surgical threshold, exclude. |
| **Skin fragility** | hEDS patients have fragile skin prone to tearing, bruising, and poor healing. Electrode contact pressure may cause tissue damage. | Use soft gel electrodes. Minimize contact pressure. Inspect ear skin before and after each session. Discontinue if any skin breakdown. |
| **Medication interactions** | hEDS/POTS patients often take: beta-blockers (HR reduction), fludrocortisone (volume expansion), midodrine (vasoconstriction), ivabradine (HR reduction). | Beta-blockers + taVNS = additive HR reduction risk. Monitor HR. See Drug Interactions section. |
| **Mast cell activation** | Some hEDS patients have MCAS (mast cell activation syndrome). Electrical stimulation could theoretically trigger mast cell degranulation in sensitized patients. | **Unknown risk.** No published data on taVNS in MCAS. Flag as theoretical concern. First session should be brief (10 min) as a tolerance test. |

### 4.4 Recommended Final Parameters

| Parameter | Value | Change from Draft? |
|-----------|-------|--------------------|
| Frequency | **25 Hz** | No change |
| Pulse width | **200 µs** | No change |
| Default current | **0.5 mA** (max 1.0 mA) | ⚠️ **CHANGED from 1.0 mA default** — start lower for sensitive population |
| Duty cycle | **30s ON / 30s OFF** | No change |
| Session duration | **30 min** (first 3 sessions: 15 min tolerance test) | ⚠️ **ADDED initial tolerance test recommendation** |
| Bilateral stagger | **20 ms** | No change (experimental) |
| Ramp | **0 → target over 45s** (longer than other presets) | ⚠️ **CHANGED from 30s to 45s** — gentler ramp for sensitive population |

### 4.5 Evidence Level

**"Theoretical / Experimental"** — No published clinical evidence for taVNS in POTS, hEDS, or dysautonomia. Mechanism is plausible based on established taVNS autonomic modulation. All parameters are extrapolated from cardiac/autonomic taVNS studies in other populations. This preset requires the strongest experimental labeling of all four.

### 4.6 UI Warning Label Recommendation

```
Dysautonomia / hEDS
────────────────────
⚠️ EXPERIMENTAL — No clinical trials support taVNS for POTS or hEDS.
Parameters are extrapolated from autonomic modulation studies in other conditions.
Mechanism is theoretically plausible but unvalidated.

🔴 SAFETY:
• Use only while seated or recumbent. Never while standing.
• First 3 sessions: 15 min only (tolerance test).
• Start at 0.5 mA. Increase only if well-tolerated.
• If you experience dizziness, presyncope, or HR drop >20 bpm: STOP immediately.
• If you have cardiac arrhythmias or structural heart disease: consult cardiologist first.
• If on beta-blockers: monitor heart rate closely (risk of excessive bradycardia).

⚠ Not a medical device. Not a substitute for POTS/hEDS medical management.
```

### 4.7 Should This Preset Carry a Stronger Warning?

**YES.** This is the only preset with zero direct clinical evidence. The other presets have at least pilot study data supporting their use case. The Dysautonomia/hEDS preset is purely theoretical. It should:

1. Be visually distinct in the BLE app (e.g., yellow/orange warning color vs. standard blue)
2. Require an explicit acknowledgment ("I understand this is experimental") before first use
3. Default to lower intensity, shorter session, longer ramp
4. Include prominent guidance to monitor HR and stop if symptomatic
5. Recommend concurrent use of Polar H10 HR monitoring (not optional for this preset)

---

## 5. Preset 4: RESET-AF Replication (Cardiac / Atrial Fibrillation)

### 5.1 Evidence Summary

| Study / Source | Year | Design | n | Parameters | Key Outcomes | Evidence Level | Confidence |
|----------------|------|--------|---|------------|-------------|----------------|------------|
| **Stavrakis et al.** (JACC Clinical Electrophysiology) — **RESET-AF** | 2020 | Single-center, double-blind, sham-controlled RCT | ~53 | **Left tragus**, 20 Hz, 200 µs, **at perception threshold intensity**, daily 1 hour, 6 months | Significant reduction in AF burden vs. sham (earlobe). Reduced inflammatory markers (TNF-α, CRP). | RCT | HIGH — Well-known landmark trial |
| **Stavrakis et al.** (JACC) — **TREAT-AF** | 2023 | Multicenter, double-blind, sham-controlled RCT | ~100+ | **Left tragus**, 20 Hz, 200 µs, perception threshold, daily 1 hour, 6 months | Primary endpoint: AF burden reduction. Results were more nuanced than RESET-AF — showed benefit on some metrics but primary endpoint results were debated. | RCT | MEDIUM — I recall the trial was published but I am not fully certain of exact primary endpoint outcome (may have been technically negative on primary endpoint with benefit on secondary endpoints) |
| **Stavrakis et al.** (Heart Rhythm) — acute study | 2015 | Acute randomized crossover | 40 (during AF ablation) | **Left tragus**, 20 Hz, 200 µs, threshold intensity, continuous during procedure | Shortened AF duration, reduced inflammatory markers acutely | RCT (acute) | HIGH |
| **Yu et al.** (various — animal studies) | 2013–2017 | Preclinical | Dogs/rats | Low-level tragus stimulation, various frequencies | Reduced AF inducibility, shortened AF duration, anti-inflammatory | Preclinical | HIGH for mechanism |

### 5.2 Detailed RESET-AF Protocol Analysis

Based on published protocol descriptions:

**Stimulation site:** LEFT TRAGUS (not cymba conchae)
- **Confidence:** HIGH
- **Implication for this project:** The project's electrode design (HW-03) targets the cymba conchae. This is a deliberate divergence from RESET-AF. The cymba conchae has higher ABVN density (Peuker & Filler 2002: ~100% ABVN innervation vs. tragus ~45%). This may actually be an improvement, but it means the preset is NOT an exact replication of RESET-AF. The UI should note this.

**Frequency:** 20 Hz
- **Confidence:** HIGH
- **Not 25 Hz** — RESET-AF specifically used 20 Hz. The current draft correctly specifies 20 Hz.

**Pulse width:** 200 µs
- **Confidence:** HIGH

**Intensity:** "At or just below perception threshold"
- **Confidence:** HIGH
- **How threshold was defined:** In published Stavrakis protocols, perception threshold is determined at the start of each session. Current is slowly increased from 0 until the patient reports a tingling/pulsing sensation. The treatment current is then set at that threshold level (some protocols specify "just below" threshold). Typical values ranged from approximately 0.5 mA to 2–3 mA depending on individual, electrode contact, and skin conditions.
- **The draft's 1.5 mA default** is reasonable as a starting point, but the essential feature is the titration process, not the fixed value.

**Duty cycle:** Continuous vs. intermittent
- This is a key question in the task. Based on my recall:
- The acute (intra-procedural) Stavrakis study (2015) used **continuous** stimulation during the cardiac procedure
- For the chronic home-use protocol (RESET-AF, TREAT-AF), I believe stimulation was also **continuous** (no ON/OFF duty cycling within the 1-hour session)
- **Confidence:** MEDIUM — I am reasonably confident this was continuous, but this specific detail should be verified against the published RESET-AF protocol paper
- **The current draft correctly specifies "continuous"**

**Laterality:** Unilateral, LEFT ear
- **Confidence:** HIGH
- RESET-AF and TREAT-AF used left ear only
- **The current draft specifies "synchronous"** — this implies bilateral synchronous stimulation, which is NOT what RESET-AF used
- **⚠️ ADJUSTMENT NEEDED:** For accurate RESET-AF replication, this should default to **unilateral left ear** (Channel A only, Channel B disabled). If bilateral is desired for exploration, it should be a user-modifiable option but NOT the default for this preset.

**Session duration:** 1 hour daily
- **Confidence:** HIGH
- The current draft correctly specifies 60 min

**Treatment duration:** 6 months (for AF outcome studies)
- Daily 1-hour sessions for 6 months
- This is for AF burden reduction — a chronic condition requiring long-term treatment
- The device doesn't need to enforce this, but documentation should note the trial duration

**Sham condition:** Earlobe stimulation (earlobe has no ABVN innervation)
- **Confidence:** HIGH — Standard sham in tragus stimulation studies

**Device used in RESET-AF:** Parasym device (Parasym Health, UK)
- Commercial taVNS device with ear clip electrodes for tragus
- **Confidence:** MEDIUM — I believe Parasym was the device used, but this should be confirmed

### 5.3 TREAT-AF Nuances

TREAT-AF (2023) is important because it was the larger, multicenter follow-up:
- My understanding is that TREAT-AF had a more complex primary endpoint and results were not as cleanly positive as RESET-AF
- Some analyses showed benefit (reduced AF episodes, improved quality of life) while the primary endpoint may not have reached statistical significance in all analyses
- This is important context: taVNS for AF is promising but not definitively proven by large trials
- **Confidence:** LOW on exact TREAT-AF primary endpoint results — I am uncertain whether it was formally positive or negative. This must be verified.

### 5.4 Recommended Final Parameters

| Parameter | Value | Change from Draft? |
|-----------|-------|--------------------|
| Frequency | **20 Hz** | No change |
| Pulse width | **200 µs** | No change |
| Default current | **1.5 mA** (titrate to perception threshold) | No change (add threshold titration in UI) |
| Duty cycle | **Continuous** (no ON/OFF) | No change |
| Session duration | **60 min** | No change |
| Laterality | **Unilateral LEFT (Channel A only)** | ⚠️ **CHANGED from "synchronous" bilateral** |
| Stagger | **N/A (unilateral)** | ⚠️ **CHANGED — not applicable in unilateral mode** |
| Ramp | **0 → target over 30s** | No change |

### 5.5 Evidence Level

**"RCT-Validated"** — Based directly on published RCT protocols (RESET-AF 2020, TREAT-AF 2023). The best-evidenced preset. However, note:
1. RESET-AF used the tragus; this device targets cymba conchae (a deliberate, potentially improved divergence)
2. TREAT-AF results may have been more mixed than RESET-AF
3. The recommended parameters are from the trial protocol; long-term AF outcome data requires medical supervision

### 5.6 UI Warning Label Recommendation

```
RESET-AF Replication (Cardiac)
──────────────────────────────
Evidence: Based on published RCTs (RESET-AF, Stavrakis et al. 2020).
Parameters: 20 Hz, 200 µs, continuous, 60 min — matches published protocol.

🔴 IMPORTANT SAFETY NOTICES:
• Atrial fibrillation is a serious cardiac condition requiring medical supervision.
• This device is NOT a substitute for prescribed AF treatment (anticoagulants,
  rate/rhythm control, ablation).
• Do NOT modify anticoagulant or antiarrhythmic medications based on taVNS use.
• If you have a cardiac pacemaker or ICD: DO NOT USE.
• If you experience chest pain, severe palpitations, or syncope: STOP and seek
  medical attention immediately.

Note: RESET-AF used tragus stimulation (left ear). This device stimulates the cymba
conchae, which has higher vagal nerve density but was not the site used in the trial.

⚠ Not a medical device. For personal research use only.
```

### 5.7 AF-Specific Contraindications

| Contraindication | Type | Rationale |
|-----------------|------|-----------|
| **Cardiac pacemaker / ICD** | **ABSOLUTE** | AF patients frequently have pacemakers or ICDs. Electrical stimulation (even auricular) poses theoretical risk of sensing interference. ALL taVNS studies exclude pacemaker/ICD patients. |
| **Anticoagulant use** (warfarin, DOACs) | Informational | Very common in AF patients. Not a contraindication to taVNS per se, but: (1) monitor for any bruising at electrode site; (2) do NOT adjust anticoagulation based on taVNS. |
| **Severe bradycardia (<50 bpm)** | Relative | taVNS increases vagal tone → can slow heart rate. AF patients on rate-control drugs (beta-blockers, diltiazem, digoxin) may already have slow ventricular rates. Combined effect could cause symptomatic bradycardia. |
| **Heart failure (NYHA III-IV)** | Relative | Severely decompensated heart failure patients are hemodynamically fragile. Vagal stimulation effects on cardiac output are unpredictable in this population. |
| **Recent cardiac surgery / ablation** | Relative (time-limited) | Allow adequate healing. No taVNS within 4 weeks of cardiac procedure. |
| **Digoxin use** | Relative | Digoxin has vagotonic effects. Combined with taVNS = enhanced risk of bradycardia and AV block. Monitor HR closely. |

### 5.8 Mandatory Disclaimer

**YES — this preset MUST include a mandatory disclaimer.** Recommended text:

> "ATRIAL FIBRILLATION REQUIRES MEDICAL SUPERVISION. This stimulation protocol replicates parameters from published research (RESET-AF trial). It is NOT a validated medical treatment. Do NOT use as a substitute for prescribed AF therapies including anticoagulation, rate control, rhythm control, or ablation. Do NOT modify any prescribed cardiac medications based on taVNS use. If you have a pacemaker or ICD, DO NOT USE this device. Consult your cardiologist before use."

This disclaimer should be:
1. Displayed in full on first use of this preset
2. Require explicit acknowledgment ("I understand and accept") before session starts
3. Summarized in the preset description shown at each session start
4. Printed in the device documentation

---

## 6. Universal Contraindications

### 6.1 Absolute Contraindications (ALL presets)

These conditions are exclusion criteria in virtually every published taVNS study. Stimulation should be blocked or strongly warned against regardless of preset.

| Contraindication | Rationale | Evidence Level | Confidence |
|-----------------|-----------|----------------|------------|
| **Cardiac pacemaker / ICD** | Theoretical risk of sensing interference, inappropriate shock delivery, or lead heating. All published taVNS studies exclude pacemaker/ICD patients. The risk is likely very low with auricular taVNS (current is local to the ear, far from the chest), but no safety data exists for this combination. | Standard exclusion | HIGH |
| **Active uncontrolled epilepsy** | Paradoxically, implanted VNS is FDA-approved FOR epilepsy. However, unsupervised taVNS in patients with frequent seizures poses risks: (1) electrode dislodgement during seizure; (2) unknown interaction with seizure threshold; (3) inability to self-manage device during post-ictal state. Well-controlled epilepsy on medication may be acceptable with physician approval. | Standard exclusion | HIGH |
| **Pregnancy** | Not studied. Vagal stimulation has theoretical effects on uterine tone and fetal heart rate. All published studies exclude pregnant subjects. | Standard exclusion | HIGH |
| **Active implanted electrical device near stimulation site** | Includes cochlear implants, deep brain stimulators, or any metallic/electronic implant in the head/neck region. Electromagnetic interference risk. | Standard exclusion | HIGH |
| **Known severe cardiac arrhythmia (uncontrolled)** | Unstable ventricular arrhythmia (VT/VF), complete heart block, sick sinus syndrome without pacemaker. taVNS affects cardiac autonomic innervation and could worsen unstable arrhythmias. Note: controlled AF is the target of Preset 4, so stable AF is not a contraindication per se — but only with appropriate disclaimers. | Standard exclusion | HIGH |

### 6.2 Relative Contraindications (ALL presets)

These conditions require caution, physician consultation, or modified parameters — but are not absolute exclusions.

| Contraindication | Rationale | Mitigation |
|-----------------|-----------|------------|
| **Cervical vagotomy history** | The vagal afferent pathway may be disrupted. Auricular branch may still be intact (it branches before the cervical level), but efficacy is uncertain. | Not dangerous, but may be ineffective. Document in session notes. |
| **Severe bradycardia (<50 bpm resting)** | taVNS increases vagal tone → further HR slowing. May cause symptomatic bradycardia (dizziness, syncope). | Monitor HR during session. Stop if HR drops below 50 bpm (or >20 bpm below pre-session baseline). Do not combine with beta-blockers or digoxin without monitoring. |
| **Active ear infection / otitis** | Electrode placement on infected tissue: pain, risk of spreading infection, poor electrode contact. | Defer until infection resolved. |
| **Ear skin lesions / dermatitis** | Electrode contact on broken skin: pain, infection risk, unpredictable current distribution. | Inspect ear before each session. Defer if skin is broken. |
| **Carotid artery stenosis (bilateral severe)** | Vagal stimulation may affect baroreceptor sensitivity and blood pressure. In patients with critical bilateral carotid stenosis, any hypotensive episode is dangerous. | Consult physician. Use recumbent position. |
| **Recent ear surgery / trauma** | Electrode placement on healing tissue. | Defer 4+ weeks post-surgery. |
| **Metal implants / piercings in or near the ear** | Piercings may alter current distribution. Metal implants could concentrate current locally. | Remove piercings before use. Metal implant in ear = exclude that ear. |

### 6.3 Non-Contraindications (Common Misconceptions)

| Condition | Why NOT a Contraindication |
|-----------|--------------------------|
| **Tinnitus** | No evidence that taVNS worsens tinnitus. Some studies have explored taVNS AS a treatment for tinnitus. |
| **Mild-moderate hypertension (controlled)** | taVNS may slightly lower BP via vagal tone increase — potentially beneficial, not harmful. Monitor BP. |
| **Depression / anxiety** | taVNS is actively being studied FOR depression. Not a contraindication. |
| **Age >65** | Bretherton et al. (2019) specifically studied taVNS in elderly. Safe and well-tolerated. |
| **Hearing aids** | Remove hearing aid from stimulated ear during session. No residual interaction. |

---

## 7. Drug Interactions

### 7.1 Clinically Significant Interactions

| Drug Class | Specific Drugs | Interaction Mechanism | Risk Level | Recommendation |
|-----------|---------------|----------------------|------------|----------------|
| **Beta-blockers** | Metoprolol, atenolol, propranolol, bisoprolol, carvedilol | Both beta-blockers and taVNS reduce heart rate. Additive negative chronotropic effect → risk of symptomatic bradycardia (HR <50, dizziness, syncope). | **MODERATE** | Monitor HR during first 3 sessions. If resting HR is already <60 bpm on beta-blocker, start taVNS at lowest current. Stop if HR drops below 50 bpm. Particularly relevant for POTS patients on propranolol and AF patients on rate-control beta-blockers. |
| **Non-dihydropyridine calcium channel blockers** | Diltiazem, verapamil | Same mechanism as beta-blockers — rate-slowing. Commonly used in AF for rate control. | **MODERATE** | Same as beta-blockers. Monitor HR. |
| **Digoxin** | Digoxin / digitalis | Digoxin has intrinsic vagotonic effects (increases vagal tone via brainstem). taVNS + digoxin = double vagotonic stimulus → risk of excessive bradycardia or AV block. | **MODERATE-HIGH** | Extra caution with this combination. Monitor HR closely. Consider shorter sessions initially. The Stavrakis RESET-AF trial may have included digoxin patients — check trial inclusion criteria for guidance. |
| **Ivabradine** | Ivabradine (Corlanor) | Selective If (funny current) inhibitor — directly slows SA node. Adding taVNS vagal tone enhancement = further HR reduction. | **MODERATE** | Common in POTS treatment. Monitor HR. May need to reduce ivabradine dose if using taVNS regularly — but this is a physician decision, NOT a device recommendation. |

### 7.2 Low-Risk / Informational Interactions

| Drug Class | Specific Drugs | Interaction Mechanism | Risk Level | Recommendation |
|-----------|---------------|----------------------|------------|----------------|
| **SSRIs** | Sertraline, fluoxetine, escitalopram, paroxetine | taVNS activates NTS → dorsal raphe nucleus (serotonergic). SSRIs increase serotonin availability. Theoretical serotonergic augmentation. | **LOW** | No documented adverse interactions in published taVNS studies. Several depression studies combined taVNS + SSRI with no safety concerns reported. Monitor for serotonergic symptoms (agitation, tremor) as a precaution. |
| **SNRIs** | Duloxetine, venlafaxine | Similar to SSRIs plus noradrenergic effects. Duloxetine is common in fibromyalgia/chronic pain (Preset 2 population). | **LOW** | Same as SSRIs. No documented interaction. |
| **Gabapentin / Pregabalin** | Gabapentin, pregabalin (Lyrica) | GABAergic/calcium channel mechanism. No known overlap with vagal afferent pathway. Very commonly co-prescribed with chronic pain. | **NEGLIGIBLE** | Safe to combine. No published interaction. |
| **Benzodiazepines** | Alprazolam, lorazepam, clonazepam, diazepam | GABAergic sedation + parasympathetic activation from taVNS = additive sedation/relaxation. | **LOW** | Not dangerous, but: (1) increased drowsiness possible — do not operate machinery; (2) may confound insomnia self-experimentation results. |
| **Z-drugs** | Zolpidem (Ambien), eszopiclone (Lunesta), zaleplon | Similar to benzodiazepines — additive sedation. | **LOW** | Same as benzodiazepines. Time taVNS session before taking sleep medication to separate effects for self-experimentation. |
| **Opioids** | Morphine, oxycodone, hydrocodone, tramadol, fentanyl | Opioids have vagotonic effects (increase vagal tone, can cause bradycardia). Combined with taVNS = theoretical additive bradycardia. Also, opioid sedation + taVNS parasympathetic enhancement = increased drowsiness. | **LOW-MODERATE** | Monitor HR in patients on high-dose opioids. Low-dose opioids (e.g., tramadol for fibromyalgia) are unlikely to cause issues. |
| **Antihypertensives** (other than beta-blockers/CCBs above) | ACE inhibitors, ARBs, thiazides | No direct interaction with vagal afferent stimulation. taVNS may mildly lower BP, which is additive but rarely clinically significant. | **NEGLIGIBLE** | Monitor BP if on multiple antihypertensives. |
| **Anticoagulants** | Warfarin, apixaban, rivarfaban, edoxaban, dabigatran | No pharmacological interaction. The concern is purely practical: anticoagulated patients may bruise more easily at electrode sites. | **NEGLIGIBLE** (pharmacological) | Inspect electrode sites for bruising. Ensure no skin breaks before applying electrodes. |
| **Antiarrhythmics** | Flecainide, amiodarone, sotalol, dronedarone | Amiodarone and sotalol have rate-slowing properties (additive with taVNS). Flecainide less so. These are relevant for AF patients (Preset 4). | **MODERATE** (amiodarone, sotalol) / **LOW** (flecainide) | Monitor HR. Amiodarone + taVNS: extra caution re: bradycardia. |

### 7.3 Drug Interaction Summary for UI

The BLE app should include a medication check prompt before first use of each preset:

```
Before your first session, check if you are taking any of these medications:
• Beta-blockers (metoprolol, atenolol, propranolol, etc.)
• Digoxin
• Ivabradine
• Diltiazem or verapamil
• Amiodarone or sotalol

If YES: Monitor your heart rate during sessions. Start with lowest intensity.
Stop if heart rate drops below 50 bpm or you feel dizzy.
Consult your physician about combining these medications with vagal stimulation.
```

---

## 8. Cross-Cutting Technical Notes

### 8.1 Bilateral Stimulation: State of Evidence

**Key finding: Almost ALL published taVNS clinical studies used UNILATERAL (left ear) stimulation.**

- Left ear preference originates from implanted cervical VNS convention (left vagus nerve has less direct efferent innervation to the SA node — reduced bradycardia risk)
- For AURICULAR taVNS, this concern is less relevant because the auricular branch is AFFERENT only (sensory). Afferent signals reach the NTS bilaterally regardless of which ear is stimulated.
- Bilateral auricular taVNS has been used in some Nuerisym protocols and is theoretically reasonable
- No published study has compared unilateral vs. bilateral auricular taVNS head-to-head
- The 20 ms asynchronous stagger between channels is a NOVEL hypothesis of this project. It is not validated by any published study.

**Recommendation:** All presets should document that bilateral stimulation is experimental. The RESET-AF preset should default to unilateral left ear. Other presets can default to bilateral (the project's experimental hypothesis) but should clearly label it.

### 8.2 Cymba Conchae vs. Tragus: Stimulation Site

| Feature | Cymba Conchae | Tragus |
|---------|--------------|-------|
| ABVN innervation density | ~100% (Peuker & Filler 2002) | ~45% (Peuker & Filler 2002) |
| Used in RESET-AF? | No | YES |
| Used in NEMOS studies? | YES | No |
| Used in Chinese insomnia RCTs? | Mostly YES | Some studies |
| Electrode access | Deeper, more difficult electrode placement | Easier, can use clip electrode |
| Patient comfort | May be less comfortable (inner ear) | Generally comfortable (clip on tragus) |

**Implication:** The project targets cymba conchae, which has higher ABVN density but was NOT the site used in RESET-AF or TREAT-AF. This means:
1. Presets 1 (Insomnia), 2 (Pain), 3 (Dysautonomia) align with NEMOS/Chinese literature that used cymba conchae → GOOD
2. Preset 4 (RESET-AF) diverges from the original trial protocol on stimulation site → DOCUMENT CLEARLY

### 8.3 Perception Threshold Titration

All published taVNS studies use individual threshold-based titration. The firmware presets provide default current values, but the UI MUST guide users through titration:

**Recommended titration protocol (for all presets):**
1. Start at 0 mA
2. Increase in 0.1 mA steps
3. Ask user: "Can you feel a tingling or pulsing sensation?" at each step
4. When tingling is first reported → that is the perception threshold
5. Set treatment current at the perception threshold (or 0.1 mA below for comfort)
6. If no tingling at the preset's max current → use the max current (electrode contact may be poor; suggest repositioning)
7. If tingling becomes uncomfortable/painful → reduce by 0.2–0.3 mA

This protocol should be built into the BLE app's session start flow for every preset, not just a documentation recommendation.

---

## 9. Recommended REQUIREMENTS.md Updates

Based on this review, the following changes to REQUIREMENTS.md FW-08 are recommended:

### 9.1 Changes Required

```
FW-08 Current:
- "Anti-Inflammatory / Chronic Pain": 10Hz, 250µs, 1.5mA, 30s ON/30s OFF, 60min, 20ms stagger

FW-08 Recommended:
- "Anti-Inflammatory / Chronic Pain": 25Hz, 250µs, 1.5mA, 30s ON/30s OFF, 60min, 20ms stagger
  Rationale: 10Hz lacks taVNS-specific human evidence; Lerman et al. (2016) used 25Hz
  for anti-inflammatory effect.
```

```
FW-08 Current:
- "RESET-AF Replication": 20Hz, 200µs, threshold (1.5mA default), continuous, 60min, synchronous

FW-08 Recommended:
- "RESET-AF Replication": 20Hz, 200µs, threshold (1.5mA default), continuous, 60min,
  UNILATERAL LEFT (Channel A only)
  Rationale: RESET-AF trial used unilateral left tragus stimulation. "Synchronous" bilateral
  does not match the trial protocol.
```

```
FW-08 Current:
- "Dysautonomia / hEDS": 25Hz, 200µs, 1.0mA, 30s ON/30s OFF, 30min, 20ms stagger

FW-08 Recommended:
- "Dysautonomia / hEDS": 25Hz, 200µs, 0.5mA (max 1.0mA), 30s ON/30s OFF, 30min,
  20ms stagger, 45s ramp
  Rationale: hEDS patients have documented sensory hypersensitivity; lower starting
  current and gentler ramp are prudent. First 3 sessions: 15 min tolerance test.
```

### 9.2 New Requirements Recommended

```
FW-10 (new): Per-preset titration flow — before each session starts, firmware/BLE app
  guides user through perception threshold titration (0 → target in 0.1 mA steps).
  The default current in each preset is a starting suggestion, not a fixed treatment current.

FW-11 (new): Per-preset safety limits — RESET-AF preset enforces Channel B = disabled
  (unilateral mode). Dysautonomia preset enforces max current = 1.0 mA, initial sessions
  max 15 min.

DOC-02 update: Add drug interaction table (Section 7 of this document) to Safety &
  Contraindications document. Include medication check prompt in BLE app.

DOC-01 update: Reference this PRESETS.md document as the clinical evidence source for
  the "Golden Parameters" table. Include evidence level labels in all documentation.
```

### 9.3 Documentation Notes for DOC-02 (Safety & Contraindications)

```
Current DOC-02:
  "absolute contraindications: cardiac pacemaker/ICD, active epilepsy, pregnancy"

Recommended DOC-02 update:
  Absolute contraindications:
  - Cardiac pacemaker / ICD
  - Active uncontrolled epilepsy (note: well-controlled epilepsy may be acceptable)
  - Pregnancy
  - Active implanted electrical device in head/neck
  - Unstable ventricular arrhythmia / complete heart block / sick sinus without pacemaker

  Relative contraindications:
  - Cervical vagotomy history
  - Severe bradycardia (<50 bpm resting)
  - Active ear infection / otitis
  - Ear skin lesions / dermatitis
  - Bilateral severe carotid stenosis
  - Recent ear surgery/trauma (<4 weeks)
  - Metal implants / piercings in ear (remove piercings; metal implant → exclude that ear)

  Drug interactions:
  - Beta-blockers: MODERATE risk (additive bradycardia) — monitor HR
  - Digoxin: MODERATE-HIGH risk (double vagotonic) — extra caution, monitor HR
  - Ivabradine: MODERATE risk (additive HR reduction) — monitor HR
  - Diltiazem/verapamil: MODERATE risk (rate-slowing) — monitor HR
  - Amiodarone/sotalol: MODERATE risk (rate-slowing) — monitor HR
  - SSRIs/SNRIs: LOW risk — no documented adverse interaction
  - Gabapentin/pregabalin: NEGLIGIBLE risk
  - Benzodiazepines/z-drugs: LOW risk (additive sedation)
  - Opioids (high-dose): LOW-MODERATE risk (additive vagotonic)
  - Anticoagulants: No pharmacological interaction (inspect skin at electrode site)
```

---

## 10. Confidence Assessment & Research Gaps

### 10.1 Overall Confidence by Domain

| Domain | Confidence | Reasoning |
|--------|-----------|-----------|
| RESET-AF protocol parameters | **HIGH** | Well-published RCTs with detailed methods sections |
| Insomnia preset parameters | **MEDIUM** | Multiple published studies, but none are large multicenter RCTs for insomnia specifically |
| Anti-inflammatory frequency (25 Hz vs. 10 Hz) | **MEDIUM** | Lerman et al. (2016) used 25 Hz; no human taVNS data at 10 Hz |
| Dysautonomia/hEDS preset | **LOW** | Zero direct clinical evidence; purely theoretical |
| Contraindications list | **HIGH** | Standard exclusion criteria across all published studies |
| Drug interactions | **MEDIUM** | Based on pharmacological reasoning + published study inclusion/exclusion criteria; no dedicated drug-interaction studies for taVNS |
| Bilateral vs. unilateral | **LOW** | No comparative studies exist |
| Binaural stagger (20 ms) | **LOW** | Novel hypothesis; no published validation |

### 10.2 Gaps That Need Phase-Specific Research

| Gap | Priority | When to Address |
|-----|----------|----------------|
| **Verify TREAT-AF (2023) primary endpoint results** | HIGH | Before finalizing RESET-AF preset documentation (DOC-01). Download actual paper from JACC. |
| **Verify Lerman et al. (2016) exact parameters** | HIGH | Before changing anti-inflammatory frequency from 10 Hz to 25 Hz in REQUIREMENTS.md. |
| **Verify RESET-AF duty cycle (continuous vs. intermittent)** | HIGH | Before finalizing RESET-AF preset. Download actual RESET-AF paper. |
| **Search for any 2024–2025 taVNS + POTS/dysautonomia studies** | MEDIUM | The field is active; new studies may have appeared since training data cutoff. |
| **Chinese insomnia RCTs: exact parameters** | MEDIUM | Download 2–3 of the most-cited Chinese taVNS insomnia papers for exact parameter extraction. |
| **Bilateral taVNS safety data** | MEDIUM | Search for any published safety data on bilateral auricular stimulation. |
| **taVNS in MCAS (mast cell activation)** | LOW | Relevant for hEDS population. Likely no data exists, but worth confirming. |

### 10.3 Sources Used

All findings in this document derive from training data knowledge of the following published literature (not exhaustive):

- Stavrakis S, et al. "Low-level tragus stimulation for the treatment of recurrent atrial fibrillation (TREAT-AF)." JACC. 2020/2023.
- Lerman I, et al. "Noninvasive transcutaneous vagus nerve stimulation decreases whole blood culture-derived cytokines and chemokines." Mol Med. 2016.
- Tracey KJ. "The inflammatory reflex." Nature. 2002.
- Peuker ET, Filler TJ. "The nerve supply of the human auricle." Clin Anat. 2002.
- Bretherton B, et al. "Effects of transcutaneous vagus nerve stimulation in individuals aged 55 years or above." Aging. 2019.
- Fang J, et al. "Transcutaneous vagus nerve stimulation modulates default mode network in major depressive disorder." Biol Psychiatry. 2016.
- Napadow V, et al. "Evoked pain analgesia in chronic pelvic pain patients using respiratory-gated auricular vagal afferent nerve stimulation." Pain Med. 2012.
- Redgrave J, et al. "Safety and tolerability of transcutaneous vagus nerve stimulation in humans; a systematic review." Brain Stimulation. 2018.
- McCreery DB, et al. "Charge density and charge per phase as cofactors in neural injury." IEEE Trans Biomed Eng. 1990.
- Shannon RV. "A model of safe levels for electrical stimulation." IEEE Trans Biomed Eng. 1992.
- Yap JYY, et al. "Critical review of transcutaneous vagus nerve stimulation." Front Neurosci. 2020.

**Confidence note:** These citations are from memory. Exact journal names, years, and co-authors should be verified against PubMed before inclusion in any formal documentation. Some may have minor inaccuracies in year or journal.

---

## 11. Summary Table: All Presets

| Parameter | Insomnia | Chronic Pain | Dysautonomia/hEDS | RESET-AF |
|-----------|----------|-------------|-------------------|----------|
| **Frequency** | 25 Hz | **25 Hz** ⚠️ | 25 Hz | 20 Hz |
| **Pulse Width** | 200 µs | 250 µs | 200 µs | 200 µs |
| **Default Current** | 2.0 mA | 1.5 mA | **0.5 mA** ⚠️ | 1.5 mA (threshold) |
| **Max Current** | 5.0 mA | 5.0 mA | **1.0 mA** ⚠️ | 5.0 mA |
| **Duty Cycle** | 30s ON/30s OFF | 30s ON/30s OFF | 30s ON/30s OFF | Continuous |
| **Session Duration** | 30 min | 60 min | 30 min (15 min initial) | 60 min |
| **Laterality** | Bilateral (experimental) | Bilateral (experimental) | Bilateral (experimental) | **Unilateral LEFT** ⚠️ |
| **Stagger** | 20 ms | 20 ms | 20 ms | N/A |
| **Ramp** | 30s | 30s | **45s** ⚠️ | 30s |
| **Evidence Level** | Pilot study | Pilot/Mechanistic | **Theoretical** | **RCT-validated** |
| **Warning Level** | Standard | Standard | **Enhanced** | **Cardiac safety** |

⚠️ = Changed from current FW-08 draft

---

*Generated: 2025-07-13 | Source: Training data clinical literature review (no live web search available)*
*Confidence: MEDIUM overall — all parameter recommendations should be verified against cited papers before firmware implementation*
*Next action: Download and verify cited papers (RESET-AF, Lerman et al., Chinese insomnia RCTs) before committing parameter changes to REQUIREMENTS.md*
