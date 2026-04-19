# taVNS Preset A/B Variant Analysis

**Project:** OpenBinaural-taVNS
**Date:** 2025-07-19
**Scope:** Identify competing valid parameter sets within each preset, propose named A/B variants where supported by published human taVNS studies
**Source data:** PRESETS.md, PRESETS-EXPANSION.md, REFERENCES.md (all in `.planning/research/`)

> **Methodology:** For each preset, every published human taVNS study cited in the project research was reviewed for parameter differences. A variant is only proposed when at least one published study supports a meaningfully different parameter set. Variants are NOT invented — if studies converge, one variant is sufficient.

---

## 1. Executive Summary Table

| # | Preset | Variants | What Differs | Default (A) | Alt (B) | Alt (C) | Rationale |
|---|--------|----------|-------------|-------------|---------|---------|-----------|
| 1 | Insomnia | **2** (existing) | Frequency | 25 Hz (NEMOS/Nuerisym) | 20 Hz (Chinese RCTs) | — | Already split; both frequencies have published support |
| 2 | RESET-AF | **1** | — | 20 Hz/200µs/cont/60min/L | — | — | All trials (Stavrakis 2015, 2020, 2023) converge on same parameters |
| 3 | Anti-Inflammatory | **2** | Pulse width, site | 25 Hz/250µs (Lerman 2016) | 25 Hz/200µs (Stavrakis cardiac) | — | Two distinct taVNS protocols showed anti-inflammatory biomarker changes |
| 4 | Dysautonomia/hEDS | **1** | — | 25 Hz/200µs/0.5mA ramp | — | — | THEORETICAL — zero taVNS data; no basis for variants |
| 5 | Depression (MDD) | **2** | Frequency, protocol style | 20 Hz/200µs (Fang/Rong 2016) | 25 Hz/250µs (NEMOS protocol) | — | Chinese 20 Hz vs NEMOS 25 Hz both studied for depression |
| 6 | Epilepsy | **2** | Frequency, session duration | 25 Hz/250µs/4h (NEMOS CE-marked) | 20 Hz/200µs/40min (Aihua 2014) | — | Western NEMOS protocol vs Chinese protocol |
| 7 | Migraine Prevention | **2** | Frequency | 1 Hz/250µs (Straube 2015 superior arm) | 25 Hz/250µs (Straube 2015 active comparator) | — | Same study tested both; 1 Hz was superior but unreplicated |
| 8 | Tinnitus | **1** | — | 25 Hz/200µs/30min | — | — | LOW evidence; all pilots converge on ~25 Hz |
| 9 | Stress/Acute Anxiety | **1** | — | 25 Hz/200µs/cont/15min | — | — | Studies converge (Burger 2019, Szeska 2020, Badran 2018) |
| 10 | Chronic Pain (Fibromyalgia) | **1** | — | 25 Hz/250µs/30min | — | — | LOW evidence; overlaps with Anti-Inflammatory |
| 11 | HRV Optimization | **2** | Laterality, duration | 25 Hz/200µs/15min/bilateral | 25 Hz/200µs/15min/unilateral L | — | Best preset for bilateral vs unilateral A/B self-experimentation |
| 12 | Hypertension | **1** | — | 20 Hz/200µs/cont/60min/L | — | — | LOW evidence; mirrors RESET-AF protocol |
| 13 | Obesity/Appetite | **1** | — | 25 Hz/250µs/30min/before meals | — | — | LOW evidence; insufficient data for variants |

**Total presets with variants: 6 out of 13 have A/B splits**
**Total preset entries (including variants): 19**

---

## 2. Per-Preset Variant Analysis

---

### 2.1 Insomnia — ALREADY SPLIT (2 variants)

**Status:** ✅ Already split into A/B in current firmware spec.

| Parameter | Variant A: Insomnia-25Hz | Variant B: Insomnia-20Hz |
|-----------|--------------------------|--------------------------|
| Frequency | **25 Hz** | **20 Hz** |
| Pulse width | 200 µs | 200 µs |
| Current | 2.0 mA (threshold) | 2.0 mA (threshold) |
| Duty cycle | 30s ON / 30s OFF | 30s ON / 30s OFF |
| Duration | 30 min | 30 min |
| Laterality | Bilateral (20ms stagger) | Bilateral (20ms stagger) |
| Source | NEMOS device standard; Nuerisym protocol; general autonomic taVNS lit | Jiao ~2020 (N≈60); Li 2019–2021 (N≈40–80); Fang 2016 (sleep as secondary, N=160) |

**Default recommendation:** **Insomnia-25Hz** as Variant A — 25 Hz has broader support across the general taVNS literature and matches the NEMOS device standard. 20 Hz is the frequency specifically tested in insomnia-focused Chinese RCTs.

**No additional variants needed.** Studies vary in duty cycle (some Chinese studies used continuous) and session frequency, but these differences are minor and not supported by head-to-head comparisons. The 25Hz/20Hz frequency split captures the most meaningful division in the literature.

---

### 2.2 RESET-AF — 1 Variant (No Split)

**Status:** ✅ One variant is sufficient.

| Parameter | Single Variant |
|-----------|---------------|
| Frequency | **20 Hz** |
| Pulse width | **200 µs** |
| Current | Perception threshold (~1.5 mA default) |
| Duty cycle | **Continuous** |
| Duration | **60 min** |
| Laterality | **Unilateral LEFT only** |
| Source | Stavrakis 2015 (REF-004, N=40), TREAT-AF 2020 (REF-005, N≈53–100+) |

**Why no split:** All Stavrakis AF trials — the 2015 acute study, RESET-AF/TREAT-AF — converge on the same parameters: 20 Hz, 200 µs, perception threshold, continuous, 60 min, unilateral left tragus. There is no competing protocol from a different research group. No other human taVNS-AF study used different parameters.

**Note:** The only potential "variant" would be stimulation site (tragus vs cymba conchae), but that's a hardware decision already made at the project level — not a parameter variant. The device targets cymba conchae, which is documented as a divergence from the RESET-AF trial protocol.

---

### 2.3 Anti-Inflammatory — 2 Variants (NEW SPLIT)

**Rationale for split:** Two distinct human taVNS protocols have demonstrated anti-inflammatory biomarker changes, with different pulse widths and stimulation contexts.

| Parameter | Variant A: Anti-Inflam-Lerman | Variant B: Anti-Inflam-Cardiac |
|-----------|-------------------------------|-------------------------------|
| Frequency | **25 Hz** | **25 Hz** |
| Pulse width | **250 µs** | **200 µs** |
| Current | 1.5 mA (sub-threshold) | Threshold (~1.5 mA) |
| Duty cycle | 30s ON / 30s OFF | **Continuous** |
| Duration | **60 min** | **60 min** |
| Laterality | Bilateral (20ms stagger) | **Unilateral LEFT** |
| Source | **Lerman et al. 2016** (REF-034) — cymba conchae, healthy volunteers, TNF-α reduction | **Stavrakis et al. 2020** (REF-005) — tragus, AF patients, CRP/TNF-α reduction as secondary outcome |

**Default recommendation:** **Variant A (Lerman)** as default — Lerman 2016 is the only study designed with an anti-inflammatory primary endpoint for auricular taVNS specifically. Stavrakis showed anti-inflammatory effects but as a secondary outcome of an AF trial.

**Key differences:**
- **Pulse width** (250µs vs 200µs): Lerman used the NEMOS-standard 250µs; Stavrakis used the cardiac-standard 200µs. Both are safe. 250µs provides slightly more charge per pulse.
- **Duty cycle** (intermittent vs continuous): Lerman used intermittent ON/OFF; Stavrakis used continuous (cardiac protocol). For anti-inflammatory pathway activation, the sustained afferent drive of continuous stimulation may differ from intermittent.
- **Laterality**: Lerman's exact laterality protocol is less precisely documented than Stavrakis (always unilateral left).

**Honest caveat:** This split is marginally justified. The Stavrakis anti-inflammatory finding was a secondary outcome, not the primary goal. Users primarily interested in anti-inflammatory effects should use Variant A. Users who also have cardiac concerns (AF, arrhythmia tendency) may prefer Variant B since it matches a full cardiac RCT protocol.

---

### 2.4 Dysautonomia/hEDS — 1 Variant (No Split)

**Status:** ✅ One variant is sufficient.

| Parameter | Single Variant |
|-----------|---------------|
| Frequency | **25 Hz** |
| Pulse width | **200 µs** |
| Current | **0.5 mA → 1.0 mA** (ramp, max 1.0 mA) |
| Duty cycle | 30s ON / 30s OFF |
| Duration | 30 min (first 3 sessions: 15 min) |
| Laterality | Bilateral (20ms stagger) |
| Ramp | 45s (longer for sensory hypersensitivity) |
| Source | **THEORETICAL** — extrapolated from Bretherton 2019 (REF-035, autonomic balance in elderly) and general taVNS HRV literature |

**Why no split:** There are **zero published taVNS studies for POTS, hEDS, or dysautonomia**. You cannot create evidence-based variants from zero evidence. The current parameters are already a best-guess extrapolation from autonomic modulation studies. Creating arbitrary variants would imply a level of evidence that does not exist.

**Note:** If future studies emerge testing specific parameters for dysautonomia, variants should be revisited.

---

### 2.5 Depression (MDD) — 2 Variants (NEW SPLIT)

**Rationale for split:** Two distinct research traditions have studied taVNS for depression with meaningfully different protocols.

| Parameter | Variant A: MDD-Chinese | Variant B: MDD-NEMOS |
|-----------|------------------------|----------------------|
| Frequency | **20 Hz** | **25 Hz** |
| Pulse width | **200 µs** | **250 µs** |
| Current | 1.0 mA (fixed) | <1.0 mA (sub-threshold) |
| Duty cycle | 30s ON / 30s OFF | 30s ON / 30s OFF |
| Duration | **30 min, 2×/day** | **Up to 4h/day** (practical: 60 min 2×/day) |
| Laterality | Unilateral LEFT | Unilateral LEFT |
| Treatment course | 4 weeks | 8–12 weeks |
| Source | **Fang/Rong et al. 2016** (NEW-01/02, N=160 RCT) — left cymba conchae; **Kong et al. 2018** (NEW-09, pilot RCT, N≈40) | **NEMOS device depression studies** (Cerbomed, CE-marked for epilepsy, depression studies mixed); 25 Hz/250 µs standard NEMOS protocol |

**Default recommendation:** **Variant A (Chinese 20 Hz)** as default — the Fang/Rong 2016 RCT (N=160) is the largest and most rigorous taVNS depression study published. HAMD-17 reduction was 37.4% active vs 18.2% sham (p < 0.001). This is substantially stronger evidence than the NEMOS depression data, which showed mixed results.

**Key differences:**
- **Frequency** (20 Hz vs 25 Hz): The 5 Hz difference is unlikely to be clinically meaningful for vagal afferent activation, but for fidelity to the strongest evidence, 20 Hz should be preferred.
- **Pulse width** (200µs vs 250µs): Follows the frequency — Chinese protocol uses 200µs, NEMOS uses 250µs.
- **Session duration/frequency**: The Chinese protocol (30 min 2×/day) is more practical than the NEMOS approach (up to 4h/day). User compliance is a major factor.

**Honest caveat:** The NEMOS depression evidence is weaker than the Chinese 20 Hz data. Variant B exists primarily for completeness and for users who want to match the NEMOS device protocol. If forced to choose only one, keep Variant A.

---

### 2.6 Epilepsy — 2 Variants (NEW SPLIT)

**Rationale for split:** The Western NEMOS protocol and the Chinese research protocol differ meaningfully in frequency, pulse width, and session structure.

| Parameter | Variant A: Epilepsy-NEMOS | Variant B: Epilepsy-Chinese |
|-----------|---------------------------|----------------------------|
| Frequency | **25 Hz** | **20 Hz** |
| Pulse width | **250 µs** | **200 µs** |
| Current | ~1.0 mA (below pain) | ~1.0 mA |
| Duty cycle | 30s ON / 30s OFF | 30s ON / 30s OFF |
| Duration | **60 min, 3×/day** (NEMOS: up to 4h/day) | **20 min, 2×/day** |
| Laterality | Unilateral LEFT | Unilateral LEFT |
| Treatment course | 20+ weeks | 12 months |
| Source | **Stefan 2012** (NEW-03, N=10 open-label); **Bauer 2016** (NEW-04, N≈76 RCT); NEMOS CE mark | **Aihua et al. 2014** (N=47, open-label); **He et al. 2013** (N=14, pediatric) |

**Default recommendation:** **Variant A (NEMOS)** as default — the NEMOS protocol has CE-mark regulatory backing and the largest RCT (Bauer 2016, N≈76). The CE mark for drug-resistant epilepsy represents the most formal validation of any taVNS protocol for any condition.

**Key differences:**
- **Session structure**: NEMOS demands very long daily exposure (3–4 hours). The Chinese protocol is less intensive (40 min/day total). For a home-use device, the Chinese protocol is far more practical for compliance, but the NEMOS protocol has regulatory backing.
- **Frequency** (25 Hz vs 20 Hz): NEMOS standardized on 25 Hz; Chinese studies often used 20 Hz or 20/4 Hz alternating. Both fall within the vagal afferent activation range.

**Honest caveat:** The Bauer 2016 RCT (NEMOS) did NOT clearly meet its primary endpoint — the active group showed 30.4% responder rate vs 27.2% sham. The per-protocol analysis trended positive. This is borderline evidence. Including two variants may overstate the strength of evidence for either protocol.

---

### 2.7 Migraine Prevention — 2 Variants (NEW SPLIT)

**Rationale for split:** The Straube 2015 RCT (REF-041) directly compared two frequencies within the same study — a rare head-to-head comparison.

| Parameter | Variant A: Migraine-1Hz | Variant B: Migraine-25Hz |
|-----------|-------------------------|--------------------------|
| Frequency | **1 Hz** | **25 Hz** |
| Pulse width | 250 µs | 250 µs |
| Current | 1.0 mA (threshold) | 1.0 mA (threshold) |
| Duty cycle | Continuous | Continuous |
| Duration | 60 min | 60 min |
| Laterality | Unilateral LEFT | Unilateral LEFT |
| Source | **Straube et al. 2015** (REF-041, N≈46) — 1 Hz arm showed **−7.0 ± 4.5 headache days/month** | **Straube et al. 2015** (REF-041) — 25 Hz arm showed **−3.3 ± 5.4 headache days/month** |

**Default recommendation:** **Variant A (1 Hz)** as default — 1 Hz showed statistically superior migraine prevention in the Straube study. However:

**Critical caveats:**
- This is a **single small study (N≈46)** that has NOT been replicated.
- The 1 Hz result is counterintuitive — most taVNS effects are attributed to Aβ fiber activation at 20–25 Hz. At 1 Hz, users feel discrete individual pulses, not a continuous buzz. The mechanism may be fundamentally different.
- 25 Hz is the "standard" taVNS frequency backed by broader literature. A user who doesn't respond to 1 Hz may benefit from trying 25 Hz.
- **Both arms showed headache day reduction** — the 25 Hz arm was less effective but not ineffective.

**Firmware note:** 1 Hz requires the firmware timer to support extremely low frequencies. At 1 Hz with 250 µs pulse width, only 0.025% of each second contains a stimulus pulse. Verify timer resolution.

---

### 2.8 Tinnitus — 1 Variant (No Split)

**Status:** ✅ One variant is sufficient.

| Parameter | Single Variant |
|-----------|---------------|
| Frequency | **25 Hz** |
| Pulse width | **200 µs** |
| Current | 1.0–2.0 mA (threshold) |
| Duty cycle | 30s ON / 30s OFF |
| Duration | 30 min |
| Laterality | Unilateral LEFT |
| Source | Lehtimäki 2013 (NEW-10, N≈10 pilot); Shim 2015 (N≈30 pilot); Yakunina 2017 (REF-006, NTS activation imaging) |

**Why no split:** All tinnitus taVNS pilots used approximately the same parameters (~25 Hz, tragus or cymba conchae, ~30 min). The evidence base is too thin (all small pilots with inconsistent results) to justify variants. Standalone taVNS for tinnitus without paired tone therapy has minimal published support.

---

### 2.9 Stress / Acute Anxiety — 1 Variant (No Split)

**Status:** ✅ One variant is sufficient.

| Parameter | Single Variant |
|-----------|---------------|
| Frequency | **25 Hz** |
| Pulse width | **200 µs** |
| Current | 1.0 mA (threshold) |
| Duty cycle | **Continuous** |
| Duration | **15 min** |
| Laterality | Unilateral LEFT (bilateral as experimental option) |
| Source | Burger et al. 2019 (NEW-05, N≈60 RCT-lab); Szeska et al. 2020 (NEW-06, N≈60); Badran et al. 2018 (REF-007, N≈15 optimization) |

**Why no split:** Stress/anxiety studies show strong convergence. Burger 2019 (25 Hz, 200 µs, 0.5 mA, cymba conchae), Szeska 2020 (25 Hz, 200 µs, 0.5 mA, cymba conchae), and Badran 2018 (25 Hz optimal for HRV shift) all point to the same parameter set. The only variation is current intensity (0.5 mA in healthy volunteers vs 1.0 mA as a practical threshold default) — not enough for a named variant.

**Potential future variant:** If acute stress studies at different durations (5 min vs 15 min vs 30 min) emerge showing dose-dependent effects, a "Quick Stress Relief" variant at 5 min could be justified. Currently no data supports this.

---

### 2.10 Chronic Pain (Fibromyalgia) — 1 Variant (No Split)

**Status:** ✅ One variant is sufficient.

| Parameter | Single Variant |
|-----------|---------------|
| Frequency | **25 Hz** |
| Pulse width | **250 µs** |
| Current | 1.5 mA (threshold; start 0.5 mA for allodynia patients) |
| Duty cycle | 30s ON / 30s OFF |
| Duration | 30 min |
| Laterality | Unilateral LEFT |
| Source | Napadow et al. 2012 (REF-037, N≈15 fMRI pilot); Kutlu ~2020 (N≈20 pilot) |

**Why no split:** Evidence is too weak (two small pilots) to support multiple variants. Parameters substantially overlap with the Anti-Inflammatory preset — PRESETS-EXPANSION.md recommends NOT including this as a separate preset at all, instead noting on the Anti-Inflammatory preset that it covers chronic pain applications.

**If kept as a separate preset:** The main differentiator from Anti-Inflammatory is the lower starting current (for fibromyalgia allodynia) and shorter session duration. These are population-appropriate modifications, not evidence-based variants.

---

### 2.11 HRV Optimization — 2 Variants (NEW SPLIT)

**Rationale for split:** This is the ideal preset for A/B self-experimentation on the project's bilateral stagger hypothesis, because HRV is directly measurable with the Polar H10.

| Parameter | Variant A: HRV-Bilateral | Variant B: HRV-Unilateral |
|-----------|--------------------------|---------------------------|
| Frequency | 25 Hz | 25 Hz |
| Pulse width | 200 µs | 200 µs |
| Current | Threshold (~1.0 mA) | Threshold (~1.0 mA) |
| Duty cycle | 30s ON / 30s OFF | 30s ON / 30s OFF |
| Duration | 15 min | 15 min |
| Laterality | **Bilateral (20ms stagger)** | **Unilateral LEFT** |
| Source | **Project hypothesis** — no published bilateral taVNS study; theoretical rationale: bilateral NTS input → stronger parasympathetic drive | Bretherton 2019 (REF-035, N=29); Clancy 2014 (NEW-07, N≈48); De Couck 2017 (NEW-08, N≈30); Badran 2018 (REF-007) — all used unilateral |

**Default recommendation:** **Variant A (Bilateral)** as default — this is the project's experimental showcase. The HRV Optimization preset is explicitly identified in PRESETS-EXPANSION.md as the **best candidate for testing the bilateral 20ms stagger hypothesis** because:
1. No laterality requirement from the evidence
2. HRV outcome is directly measurable with Polar H10
3. Low risk (wellness application in generally healthy users)
4. Enables direct n-of-1 A/B comparison within the same user

**Variant B (Unilateral)** exists as the control condition for self-experimentation: run Variant A one week, Variant B the next, compare RMSSD/HF-HRV metrics.

**Honest caveat:** This is the only split where the rationale is experimental methodology rather than competing published evidence. Both "variants" use the same core parameters — only laterality differs. This is a deliberate design choice for the project's novel bilateral hypothesis testing.

---

### 2.12 Hypertension — 1 Variant (No Split)

**Status:** ✅ One variant is sufficient.

| Parameter | Single Variant |
|-----------|---------------|
| Frequency | **20 Hz** |
| Pulse width | **200 µs** |
| Current | Threshold (~1.0 mA) |
| Duty cycle | **Continuous** |
| Duration | **60 min** |
| Laterality | Unilateral LEFT |
| Source | **THEORETICAL** — extrapolated from Stavrakis AF trials (cardiovascular mechanism overlap); Bretherton 2019 (autonomic balance shift) |

**Why no split:** No published RCT specifically tests taVNS for hypertension. Parameters mirror the RESET-AF protocol because the cardiovascular mechanisms overlap (autonomic rebalancing, reduced sympathetic tone). Creating variants from zero direct evidence would be dishonest.

**Note:** PRESETS-EXPANSION.md questions whether this preset should exist at all, given its near-total parameter overlap with RESET-AF. It could be handled as a documentation note on RESET-AF: "This protocol may also benefit sympathetically-driven hypertension."

---

### 2.13 Obesity/Appetite — 1 Variant (No Split)

**Status:** ✅ One variant is sufficient.

| Parameter | Single Variant |
|-----------|---------------|
| Frequency | **25 Hz** |
| Pulse width | **250 µs** |
| Current | Threshold |
| Duty cycle | 30s ON / 30s OFF |
| Duration | **30 min (before meals)** |
| Laterality | Bilateral (20ms stagger) |
| Source | Huang 2014 (N≈20 pilot, electroacupuncture-style); Nishi ~2020 (N≈10 pilot, ghrelin changes) |

**Why no split:** Evidence is extremely weak — two tiny pilots with uncertain methodology. One (Huang 2014) used electroacupuncture-style stimulation rather than standard taVNS. Insufficient data to propose competing parameter sets. The pre-meal timing is the only parameter with any theoretical specificity to this indication.

---

## 3. Total Preset Count

### 3.1 By Category

| Category | Presets | Variants | Total Entries |
|----------|--------|----------|---------------|
| **Existing (already in spec)** | 5 + Exploration | 2 (Insomnia already split) | 6 + Exploration |
| **New — with variants** | 4 (Anti-Inflam, Depression, Epilepsy, Migraine) | 4 new B-variants | 8 entries |
| **New — single variant** | 5 (Tinnitus, Stress, Pain, HRV, Hypertension, Obesity) | 0 | 5 entries |
| **HRV bilateral/unilateral split** | 1 | 1 new B-variant | 2 entries |

### 3.2 Complete Preset Menu (All Variants)

```
╔══════════════════════════════════════════════════════════════════════════╗
║                      STIMULATION PRESETS — FULL LIST                    ║
╠══════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║  ── WELL-EVIDENCED ───────────────────────────────────────────────────  ║
║  [1A] Insomnia — 25Hz               25Hz/200µs/30s-30s/30min/bi       ║
║  [1B] Insomnia — 20Hz               20Hz/200µs/30s-30s/30min/bi       ║
║  [2]  RESET-AF (Cardiac)            20Hz/200µs/cont/60min/L only      ║
║  [3A] Depression — Chinese Protocol  20Hz/200µs/30s-30s/30min 2×/L    ║
║  [3B] Depression — NEMOS Protocol    25Hz/250µs/30s-30s/60min 2×/L    ║
║  [4A] Epilepsy — NEMOS Protocol      25Hz/250µs/30s-30s/60min 3×/L    ║
║  [4B] Epilepsy — Chinese Protocol    20Hz/200µs/30s-30s/20min 2×/L    ║
║  [5A] Anti-Inflammatory — Lerman     25Hz/250µs/30s-30s/60min/bi      ║
║  [5B] Anti-Inflammatory — Cardiac    25Hz/200µs/cont/60min/L only     ║
║                                                                        ║
║  ── MODERATE EVIDENCE ────────────────────────────────────────────────  ║
║  [6]  Stress / Acute Anxiety         25Hz/200µs/cont/15min/L          ║
║  [7A] HRV Optimization — Bilateral   25Hz/200µs/30s-30s/15min/bi      ║
║  [7B] HRV Optimization — Unilateral  25Hz/200µs/30s-30s/15min/L       ║
║  [8A] Migraine — 1Hz (Straube sup.)  1Hz/250µs/cont/60min/L           ║
║  [8B] Migraine — 25Hz (active comp.) 25Hz/250µs/cont/60min/L          ║
║                                                                        ║
║  ── LOW EVIDENCE / EXPERIMENTAL ──────────────────────────────────────  ║
║  [9]  Dysautonomia / hEDS            25Hz/200µs/30s-30s/30min/bi ⚠    ║
║  [10] Tinnitus                       25Hz/200µs/30s-30s/30min/L  ⚠    ║
║  [11] Chronic Pain (Fibromyalgia)    25Hz/250µs/30s-30s/30min/L  ⚠    ║
║  [12] Hypertension                   20Hz/200µs/cont/60min/L     ⚠    ║
║  [13] Obesity / Appetite             25Hz/250µs/30s-30s/30min/bi ⚠    ║
║                                                                        ║
║  ── USER-DEFINED ─────────────────────────────────────────────────────  ║
║  [14] Exploration                    All parameters user-adjustable    ║
║                                                                        ║
╚══════════════════════════════════════════════════════════════════════════╝

Total: 21 preset entries (13 conditions × variants + Exploration)
```

### 3.3 Recommended Reduced Menu (Practical)

Per PRESETS-EXPANSION.md recommendations, Tinnitus, Chronic Pain, Hypertension, and Obesity could be omitted (handled via Exploration or existing presets). This yields:

**Practical menu: 16 preset entries + Exploration = 17 total**

| # | Entry | Evidence Tier |
|---|-------|--------------|
| 1A | Insomnia-25Hz | Well-evidenced |
| 1B | Insomnia-20Hz | Well-evidenced |
| 2 | RESET-AF | Well-evidenced |
| 3A | Depression-Chinese | Well-evidenced |
| 3B | Depression-NEMOS | Well-evidenced |
| 4A | Epilepsy-NEMOS | Well-evidenced |
| 4B | Epilepsy-Chinese | Well-evidenced |
| 5A | Anti-Inflam-Lerman | Well-evidenced |
| 5B | Anti-Inflam-Cardiac | Well-evidenced |
| 6 | Stress/Anxiety | Moderate |
| 7A | HRV-Bilateral | Moderate |
| 7B | HRV-Unilateral | Moderate |
| 8A | Migraine-1Hz | Moderate |
| 8B | Migraine-25Hz | Moderate |
| 9 | Dysautonomia/hEDS | Experimental |
| — | Exploration | User-defined |

---

## 4. Parameter Overlap Analysis

### 4.1 Identical Parameter Sets

These variant pairs share **identical stimulation parameters** and differ only in labeling/context:

| Group | Presets | Shared Parameters | What Actually Differs |
|-------|---------|-------------------|-----------------------|
| **A** | Insomnia-20Hz, Depression-Chinese | 20 Hz, 200 µs, 30s/30s, 30 min | Current (2.0 vs 1.0 mA), laterality (bi vs L), session frequency (1× vs 2×/day) |
| **B** | RESET-AF, Hypertension | 20 Hz, 200 µs, continuous, 60 min, L only | Nothing — parameters are identical. Hypertension is a rebranded RESET-AF. |
| **C** | Anti-Inflam-Cardiac, RESET-AF | 20 Hz (Anti-Inflam-Cardiac uses 25 Hz) | Frequency differs (25 vs 20 Hz); otherwise overlap is high |
| **D** | Epilepsy-NEMOS, Anti-Inflam-Lerman | 25 Hz, 250 µs, 30s/30s, 60 min | Current (1.0 vs 1.5 mA), laterality (L vs bi), session frequency (3×/day vs 1×/day) |
| **E** | HRV-Unilateral, Stress/Anxiety | 25 Hz, 200 µs, L only | Duty cycle (30s/30s vs continuous), duration (15 min — same) |
| **F** | Insomnia-25Hz, HRV-Bilateral | 25 Hz, 200 µs, 30s/30s, bilateral | Duration (30 min vs 15 min), current (2.0 vs threshold) |

### 4.2 Parameter Template Clustering (Updated with Variants)

With variants, the 5 base templates from PRESETS-EXPANSION.md still hold, but the assignment is clearer:

| Template | Hz | PW | Duty | Presets Using It |
|----------|----|----|------|-----------------|
| **vagal-standard** | 25 | 200 | 30s/30s | Insomnia-25Hz, HRV-Bilateral, HRV-Unilateral, Stress (continuous override), Tinnitus, Dysautonomia |
| **depression-protocol** | 20 | 200 | 30s/30s | Insomnia-20Hz, Depression-Chinese |
| **nemos-protocol** | 25 | 250 | 30s/30s | Epilepsy-NEMOS, Anti-Inflam-Lerman, Depression-NEMOS, Chronic Pain, Obesity |
| **cardiac-protocol** | 20 | 200 | continuous | RESET-AF, Hypertension, Anti-Inflam-Cardiac |
| **low-frequency** | 1 | 250 | continuous | Migraine-1Hz |

**Additional override:** Migraine-25Hz uses nemos-protocol with continuous duty cycle override.

### 4.3 Firmware Simplification Opportunity

With templates, each preset is defined as:
```
preset = template + overrides (current, duration, laterality, ramp, session_frequency)
```

This means:
- **5 base templates** define Hz/PW/duty
- **Per-preset overrides** handle current defaults, duration, laterality enforcement, ramp time, and session scheduling
- **Variants within a condition** typically switch templates (e.g., Depression-Chinese uses depression-protocol, Depression-NEMOS uses nemos-protocol)

This reduces the firmware complexity from 21 independent parameter sets to 5 templates + override tables.

### 4.4 Presets That Could Be Merged

Based on overlap analysis, the following merges are defensible:

| Candidate Merge | Rationale | Recommendation |
|-----------------|-----------|----------------|
| Hypertension → RESET-AF | Identical parameters; hypertension has no unique taVNS data | **MERGE** — add note to RESET-AF: "Also applicable to sympathetically-driven hypertension" |
| Chronic Pain → Anti-Inflam-Lerman | Nearly identical (same Hz/PW/duty; differ only in current and duration) | **MERGE** — add note to Anti-Inflammatory: "Covers chronic pain applications; start at 0.5 mA for fibromyalgia" |
| Depression-NEMOS → Epilepsy-NEMOS | Same template (25 Hz/250 µs) but different durations and session counts | **DO NOT MERGE** — clinically distinct conditions with different dosing |
| HRV-Bilateral → Insomnia-25Hz | Same Hz/PW/duty but different duration and purpose | **DO NOT MERGE** — different use cases; HRV is 15 min wellness, Insomnia is 30 min therapeutic |

After merges: **Practical menu drops to 15 entries + Exploration = 16 total**

---

## 5. Summary of Variant Decisions

| Preset | Split? | Variant Count | Confidence in Split |
|--------|--------|--------------|---------------------|
| Insomnia | ✅ Yes (existing) | 2 | **MEDIUM** — no head-to-head; both frequencies have published support |
| RESET-AF | ❌ No | 1 | **HIGH** — all trials converge |
| Anti-Inflammatory | ✅ Yes (new) | 2 | **LOW-MEDIUM** — Variant B anti-inflammatory data is secondary outcome only |
| Dysautonomia | ❌ No | 1 | **HIGH** — zero evidence = zero variants |
| Depression | ✅ Yes (new) | 2 | **MEDIUM** — two distinct research traditions with different protocols |
| Epilepsy | ✅ Yes (new) | 2 | **MEDIUM** — NEMOS vs Chinese protocol; both have limited evidence |
| Migraine | ✅ Yes (new) | 2 | **MEDIUM-HIGH** — same study tested both frequencies head-to-head |
| Tinnitus | ❌ No | 1 | **HIGH** — too little evidence for variants |
| Stress/Anxiety | ❌ No | 1 | **HIGH** — studies converge |
| Chronic Pain | ❌ No | 1 | **HIGH** — too little evidence; overlaps Anti-Inflammatory |
| HRV Optimization | ✅ Yes (new) | 2 | **MEDIUM** — split is for bilateral hypothesis testing, not competing evidence |
| Hypertension | ❌ No | 1 | **HIGH** — zero direct evidence = zero variants |
| Obesity | ❌ No | 1 | **HIGH** — too little evidence for variants |

**Grand total: 6 conditions split × 2 variants each + 7 single-variant conditions = 19 condition-preset entries + Exploration = 20 total**

---

## 6. References Cited in This Analysis

All references below are indexed in `.planning/research/REFERENCES.md` (REF-xxx) or `.planning/research/PRESETS-EXPANSION.md` (NEW-xx).

| Ref | Authors | Year | Journal | Used For |
|-----|---------|------|---------|----------|
| REF-004 | Stavrakis et al. | 2015 | JACC | RESET-AF parameters |
| REF-005 | Stavrakis et al. | 2020 | JACC Clin EP | TREAT-AF, Anti-Inflam-Cardiac variant |
| REF-006 | Yakunina et al. | 2017 | Neuromodulation | Tinnitus — NTS activation |
| REF-007 | Badran et al. | 2018 | Brain Stimul | Stress, HRV — parameter optimization |
| REF-034 | Lerman et al. | 2016 | Mol Med | Anti-Inflam-Lerman variant (25 Hz anchor) |
| REF-035 | Bretherton et al. | 2019 | Aging | HRV, Dysautonomia — autonomic balance |
| REF-036 | Fang et al. | 2016 | Biol Psychiatry | Depression-Chinese variant (N=160) |
| REF-037 | Napadow et al. | 2012 | Pain Med | Chronic Pain — fMRI pilot |
| REF-041 | Straube et al. | 2015 | J Headache Pain | Migraine — 1 Hz vs 25 Hz |
| NEW-01 | Fang et al. | 2016 | Biol Psychiatry | Depression-Chinese variant (fMRI) |
| NEW-02 | Rong et al. | 2016 | Biol Psychiatry | Depression-Chinese variant (clinical) |
| NEW-03 | Stefan et al. | 2012 | Epilepsia | Epilepsy-NEMOS variant |
| NEW-04 | Bauer et al. | 2016 | Epilepsia | Epilepsy-NEMOS variant (RCT) |
| NEW-05 | Burger et al. | 2019 | Psychophysiology | Stress — cortisol reduction |
| NEW-06 | Szeska et al. | 2020 | Biol Psychiatry | Stress — fear extinction |
| NEW-07 | Clancy et al. | 2014 | Brain Stimul | HRV — unilateral baseline |
| NEW-08 | De Couck et al. | 2017 | Complement Ther Med | HRV — acute RMSSD increase |
| NEW-09 | Kong et al. | 2018 | J Psychiatr Res | Depression — supplementary |

---

*Document generated for OpenBinaural-taVNS project.*
*All parameters must be verified against cited papers before firmware implementation.*
*This document should be updated as new taVNS clinical evidence is published.*
