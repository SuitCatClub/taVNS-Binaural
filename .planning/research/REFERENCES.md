# taVNS Clinical & Technical Reference Library

> Living bibliography for the taVNS-Binaural open-source project.
> Last updated: 2025-07-14
> DOI links verified against training data (cutoff ~2024). URLs flagged with ⚠️ need manual verification.
> All PubMed search URLs are stable and will find the paper even if PMID changes.

## How to Use This File

- **REF-IDs** are used inline in all `.planning/research/` documents
- **URLs** link to canonical source (DOI preferred, PubMed fallback, search URL last resort)
- **Evidence Level** assigned per modified GRADE: RCT > Cohort > Case Series > Expert Opinion > Theoretical
- **Cited In** column shows which research documents reference each paper
- **⚠️ VERIFY** flag means the DOI/PMID is from training data and should be confirmed before publication

---

## A. Foundational Neuroscience

Vagal anatomy, brainstem pathways, auricular branch of vagus nerve (ABVN), nucleus tractus solitarius (NTS) projections.

| ID | Authors | Title | Journal | Year | URL | Evidence Level | Cited In |
|----|---------|-------|---------|------|-----|----------------|----------|
| REF-001 | Peuker ET, Filler TJ | The nerve supply of the human auricle | Clin Anat | 2002 | [DOI: 10.1002/ca.1089](https://doi.org/10.1002/ca.1089) ⚠️ VERIFY | Anatomical study — HIGH | FEATURES, SUMMARY, PITFALLS, ARCHITECTURE |
| REF-002 | Tracey KJ | The inflammatory reflex | Nature | 2002 | [DOI: 10.1038/nature01321](https://doi.org/10.1038/nature01321) ⚠️ VERIFY | Review — HIGH | Implicitly via α7 nAChR pathway in REQUIREMENTS (FW-08 anti-inflammatory preset) |
| REF-003 | Tracey KJ | Physiology and immunology of the cholinergic antiinflammatory pathway | J Clin Invest | 2007 | [DOI: 10.1172/JCI30555](https://doi.org/10.1172/JCI30555) ⚠️ VERIFY | Review — HIGH | Implicitly via α7 nAChR pathway in REQUIREMENTS (FW-08 anti-inflammatory preset) |

**Notes:**
- **REF-001** is the foundational reference for cymba conchae electrode placement. Peuker & Filler found ~100% ABVN innervation in the cymba conchae vs. ~45% in the tragus. This single paper underpins the electrode site decision for the entire project.
- **REF-002/003** (Tracey) are not explicitly named in the source files but are the foundational references for the "Anti-Inflammatory / Chronic Pain" preset in REQUIREMENTS.md, which targets the "vagal anti-inflammatory reflex (α7 nAChR pathway)." These are the seminal papers establishing that pathway.

---

## B. taVNS Clinical Trials

Human RCTs, pilot studies, and parameter optimization studies for transcutaneous auricular vagus nerve stimulation.

| ID | Authors | Title | Journal | Year | URL | Evidence Level | Cited In |
|----|---------|-------|---------|------|-----|----------------|----------|
| REF-004 | Stavrakis S, Humphrey MB, Scherlag BJ, et al. | Low-level transcutaneous electrical vagus nerve stimulation suppresses atrial fibrillation | J Am Coll Cardiol (JACC) | 2015 | [DOI: 10.1016/j.jacc.2014.12.026](https://doi.org/10.1016/j.jacc.2014.12.026) ⚠️ VERIFY | RCT (pilot, N=40) — HIGH | FEATURES, SUMMARY, PROJECT |
| REF-005 | Stavrakis S, Stoner JA, Humphrey MB, et al. | TREAT AF (Transcutaneous Electrical Vagus Nerve Stimulation to Suppress Atrial Fibrillation): A Randomized Clinical Trial | JACC Clin Electrophysiol | 2020 | [DOI: 10.1016/j.jacep.2019.11.008](https://doi.org/10.1016/j.jacep.2019.11.008) ⚠️ VERIFY | RCT — HIGH | FEATURES, SUMMARY |
| REF-006 | Yakunina N, Kim SS, Nam EC | Optimization of Transcutaneous Vagus Nerve Stimulation Using Functional MRI | Neuromodulation | 2017 | [DOI: 10.1111/ner.12541](https://doi.org/10.1111/ner.12541) ⚠️ VERIFY | fMRI study — MEDIUM | PITFALLS |
| REF-007 | Badran BW, Mithoefer OJ, Summer CE, et al. | Short trains of transcutaneous auricular vagus nerve stimulation (taVNS) have parameter-specific effects on heart rate | Brain Stimul | 2018 | [DOI: 10.1016/j.brs.2018.04.004](https://doi.org/10.1016/j.brs.2018.04.004) ⚠️ VERIFY | Parameter optimization — MEDIUM | PITFALLS |

### ⚠️ CRITICAL DISCREPANCY: "RESET-AF" Trial Name

The source files (FEATURES.md, SUMMARY.md, PROJECT.md) consistently reference **"RESET-AF (Stavrakis et al., JACC 2020)"** as using 20 Hz, 200µs, tragus, 1h parameters.

**Issue:** The author (Stavrakis) and parameters match, but the trial name "RESET-AF" could not be confirmed in training data. The known Stavrakis taVNS-AF trials are:
- **2015 pilot** (REF-004): Low-level tragus stimulation, JACC 2015 — uses 20 Hz, described as proof-of-concept
- **TREAT-AF** (REF-005): The full RCT, JACC Clin EP 2020 — the larger confirmatory trial

**Possible explanations:**
1. "RESET-AF" may be the name of a 2020 sub-study or extension not in my training data
2. The source files may have conflated two trials (2015 pilot parameters + 2020 TREAT-AF publication year)
3. There may be a separate "RESET-AF" trial published 2020-2024 that post-dates my training data

**Resolution needed:** Search ClinicalTrials.gov for "RESET-AF Stavrakis" and PubMed for the exact trial name. Until verified, treat REF-004 (2015) and REF-005 (2020 TREAT-AF) as the canonical Stavrakis taVNS-AF references.

The source files also reference **"TREAT-AF (Stavrakis et al., 2023)"** as a follow-up confirming RESET-AF parameters in a larger cohort. This may refer to:
- A 2023 long-term follow-up publication of the TREAT-AF trial
- Or the 2020 TREAT-AF RCT with misattributed year

| ID | Ref in Source Files | Best Match | Action |
|----|--------------------|-----------:|--------|
| REF-004/005 | "RESET-AF (Stavrakis, JACC 2020)" | Likely REF-004 (2015 pilot) or REF-005 (2020 TREAT-AF) | [Search PubMed](https://pubmed.ncbi.nlm.nih.gov/?term=Stavrakis+RESET-AF+atrial+fibrillation) |
| — | "TREAT-AF (Stavrakis, 2023)" | Likely REF-005 or a follow-up publication | [Search PubMed](https://pubmed.ncbi.nlm.nih.gov/?term=Stavrakis+TREAT-AF+2023+vagus+nerve+stimulation) |

---

## C. VNS Foundational (Implanted) — Cited to Support taVNS Rationale

Only included because they establish the mechanistic basis for vagus nerve stimulation.

| ID | Authors | Title | Journal | Year | URL | Evidence Level | Cited In |
|----|---------|-------|---------|------|-----|----------------|----------|
| REF-002 | Tracey KJ | The inflammatory reflex | Nature | 2002 | See Section A | Review — HIGH | REQUIREMENTS |
| REF-003 | Tracey KJ | Physiology and immunology of the cholinergic antiinflammatory pathway | J Clin Invest | 2007 | See Section A | Review — HIGH | REQUIREMENTS |

**Note:** REF-002 and REF-003 appear in both Section A and Section C because they bridge foundational neuroscience and implanted VNS rationale. Tracey's work on the cholinergic anti-inflammatory pathway via the vagus nerve is the mechanistic basis for both implanted VNS and transcutaneous taVNS approaches.

---

## D. Target Condition Evidence

### D1. Insomnia / Sleep

| ID | Authors | Title | Journal | Year | URL | Evidence Level | Cited In |
|----|---------|-------|---------|------|-----|----------------|----------|
| REF-008 | — (Nuerisym case reports) | taVNS for insomnia — bedtime use case reports | — | ~2023-2024 | [Search PubMed](https://pubmed.ncbi.nlm.nih.gov/?term=transcutaneous+auricular+vagus+nerve+stimulation+insomnia) | Case series — LOW | FEATURES, PROJECT |

**⚠️ GAP: No specific insomnia taVNS RCT is cited in the source files.** The golden parameters for insomnia (25 Hz, 200µs, 30s ON/OFF, 30 min) are extrapolated from autonomic/cardiac taVNS trials. FEATURES.md explicitly acknowledges: *"For insomnia specifically, the evidence base is smaller (mostly pilot studies and case series, not large RCTs). The parameters are extrapolated from autonomic/cardiac trials because the mechanism (parasympathetic enhancement via vagal afferents → NTS → dorsal motor nucleus) is the same."*

**Recommended additional references (not cited in source files but relevant):**
- Search: [taVNS insomnia RCT on PubMed](https://pubmed.ncbi.nlm.nih.gov/?term=transcutaneous+auricular+vagus+nerve+stimulation+insomnia+randomized)
- Search: [taVNS sleep quality on PubMed](https://pubmed.ncbi.nlm.nih.gov/?term=taVNS+sleep+quality+randomized+controlled+trial)

### D2. Chronic Pain / Anti-inflammatory

| ID | Authors | Title | Journal | Year | URL | Evidence Level | Cited In |
|----|---------|-------|---------|------|-----|----------------|----------|
| REF-002 | Tracey KJ | The inflammatory reflex | Nature | 2002 | See Section A | Mechanistic review — HIGH | REQUIREMENTS |
| REF-003 | Tracey KJ | Cholinergic antiinflammatory pathway | J Clin Invest | 2007 | See Section A | Mechanistic review — HIGH | REQUIREMENTS |

**Note:** The "Anti-Inflammatory / Chronic Pain" preset (REQUIREMENTS.md FW-08) uses 10 Hz, 250µs, targeting the α7 nAChR pathway established in REF-002/003. No specific taVNS-for-pain clinical trial is cited in the source files.

### D3. Dysautonomia / POTS / hEDS

**⚠️ GAP: No specific paper is cited.** REQUIREMENTS.md (FW-08) includes a "Dysautonomia / hEDS" preset labeled "⚠️ experimental, low clinical evidence." This is consistent — no foundational reference was provided because the evidence base is sparse.

**Recommended search:**
- [taVNS POTS dysautonomia on PubMed](https://pubmed.ncbi.nlm.nih.gov/?term=transcutaneous+auricular+vagus+nerve+stimulation+POTS+dysautonomia)
- [taVNS Ehlers-Danlos on PubMed](https://pubmed.ncbi.nlm.nih.gov/?term=vagus+nerve+stimulation+Ehlers-Danlos+hypermobility)

### D4. Atrial Fibrillation

| ID | Authors | Title | Journal | Year | URL | Evidence Level | Cited In |
|----|---------|-------|---------|------|-----|----------------|----------|
| REF-004 | Stavrakis S, et al. | Low-level transcutaneous electrical vagus nerve stimulation suppresses atrial fibrillation | JACC | 2015 | See Section B | RCT pilot — HIGH | FEATURES, SUMMARY, PROJECT |
| REF-005 | Stavrakis S, et al. | TREAT AF: A Randomized Clinical Trial | JACC Clin Electrophysiol | 2020 | See Section B | RCT — HIGH | FEATURES, SUMMARY |

**ClinicalTrials.gov:**
- TREAT-AF: [Search ClinicalTrials.gov](https://clinicaltrials.gov/search?term=TREAT+AF+vagus+nerve+stimulation+Stavrakis)
- RESET-AF: [Search ClinicalTrials.gov](https://clinicaltrials.gov/search?term=RESET-AF+vagus+nerve+stimulation)

---

## E. HRV Methodology & Validation

Standards for heart rate variability measurement, Polar H10 validation, and HRV metrics methodology.

| ID | Authors | Title | Journal / Body | Year | URL | Evidence Level | Cited In |
|----|---------|-------|----------------|------|-----|----------------|----------|
| REF-009 | Task Force of the ESC and NASPE | Heart rate variability: Standards of measurement, physiological interpretation, and clinical use | Circulation | 1996 | [DOI: 10.1161/01.CIR.93.5.1043](https://doi.org/10.1161/01.CIR.93.5.1043) ⚠️ VERIFY | Guidelines — GOLD STANDARD | FEATURES, SUMMARY, PITFALLS |
| REF-010 | Laborde S, Mosley E, Thayer JF | Heart Rate Variability and Cardiac Vagal Tone in Psychophysiological Research — Recommendations for Experiment Planning, Data Analysis, and Data Reporting | Front Psychol | 2017 | [DOI: 10.3389/fpsyg.2017.00213](https://doi.org/10.3389/fpsyg.2017.00213) ⚠️ VERIFY | Practice guidelines — HIGH | FEATURES |
| REF-011 | Shaffer F, Ginsberg JP | An Overview of Heart Rate Variability Metrics and Norms | Front Public Health | 2017 | [DOI: 10.3389/fpubh.2017.00258](https://doi.org/10.3389/fpubh.2017.00258) ⚠️ VERIFY | Review — HIGH | PITFALLS |
| REF-012 | Gilgen-Ammann R, Schweizer T, Wyss T | RR interval signal quality of a heart rate monitor and an ECG Holter at rest and during exercise | Eur J Appl Physiol | 2019 | [DOI: 10.1007/s00421-019-04142-5](https://doi.org/10.1007/s00421-019-04142-5) ⚠️ VERIFY | Validation study — MEDIUM | Not explicitly cited; supports Polar H10 as "gold standard consumer HRV" claim in PROJECT.md |

**Notes:**
- **REF-009** is THE standard for HRV measurement. Defines RMSSD, SDNN, pNN50, LF/HF ratio. Every HRV computation in this project follows these definitions.
- **REF-010** provides practical experimental design recommendations that directly inform the HRV measurement protocol (PITFALLS C-03).
- **REF-012** is a Polar H10 validation study. While not explicitly cited in the source files, PROJECT.md calls the Polar H10 "gold standard consumer HRV" — this paper supports that claim. **LOW confidence on exact DOI; verify.**

---

## F. Safety & Biocompatibility Standards

Charge density limits, electrical safety standards, and stimulation safety models.

| ID | Authors / Body | Title | Publication | Year | URL | Cited In |
|----|----------------|-------|-------------|------|-----|----------|
| REF-013 | McCreery DB, Agnew WF, Yuen TG, Bullara L | Charge density and charge per phase as cofactors in neural injury induced by electrical stimulation | IEEE Trans Biomed Eng | 1990 | [DOI: 10.1109/10.102812](https://doi.org/10.1109/10.102812) ⚠️ VERIFY | FEATURES, SUMMARY, PITFALLS, PROJECT |
| REF-014 | Shannon RV | A model of safe levels for electrical stimulation | IEEE Trans Biomed Eng | 1992 | [DOI: 10.1109/10.126616](https://doi.org/10.1109/10.126616) ⚠️ VERIFY | FEATURES, SUMMARY, PITFALLS |
| REF-015 | Merrill DR, Bikson M, Jefferys JGR | Electrical stimulation of excitable tissue: Design of efficacious and safe protocols | J Neurosci Methods | 2005 | [DOI: 10.1016/j.jneumeth.2004.10.020](https://doi.org/10.1016/j.jneumeth.2004.10.020) ⚠️ VERIFY | PITFALLS |
| REF-016 | IEC | IEC 60601-1:2005+AMD1:2012 — Medical electrical equipment — Part 1: General requirements for basic safety and essential performance | International Electrotechnical Commission | 2012 | [IEC Webstore](https://webstore.iec.ch/en/publication/2606) | FEATURES, SUMMARY, ARCHITECTURE, PITFALLS |
| REF-017 | IEC | IEC 60601-2-10 — Particular requirements for nerve and muscle stimulators | International Electrotechnical Commission | — | [IEC Webstore](https://webstore.iec.ch/en/publication/2622) | FEATURES, PITFALLS |
| REF-018 | ANSI/AAMI | ANSI/AAMI NS4:2024 — Transcutaneous electrical nerve stimulators | AAMI | 2024 | [AAMI Store](https://www.aami.org/standards/ns4) | SUMMARY |
| REF-019 | ISO | ISO 13485 — Medical devices — Quality management systems | International Organization for Standardization | 2016 | [ISO](https://www.iso.org/standard/59752.html) | REQUIREMENTS, FEATURES (AF-10) |

### ⚠️ CRITICAL DISCREPANCY: McCreery Year

The source files consistently reference **"McCreery et al. (2010)"** for the 30 µC/cm² charge density safety limit. However:

- The **charge density safety limit (30 µC/cm²)** was established in the **1990 paper** (REF-013): McCreery DB, Agnew WF, Yuen TG, Bullara L. "Charge density and charge per phase as cofactors in neural injury induced by electrical stimulation." IEEE Trans Biomed Eng. 1990;37(10):996-1001.
- McCreery **did** publish a 2010 paper: McCreery D, Pikov V, Troyk PR. "Neuronal loss due to prolonged controlled-current stimulation with chronically implanted microelectrodes in the cat cerebral cortex." J Neural Eng. 2010;7(3):036005. — but this is about prolonged microstimulation in cortex, not the foundational charge density limit.

**The 30 µC/cm² limit cited throughout the project comes from the 1990 paper, not 2010.** This discrepancy should be corrected in all `.planning/research/` documents. The DOI for the canonical paper is REF-013 above.

---

## G. Hardware & Component Technical References

Technical papers and application notes cited in the architecture and pitfalls documents.

| ID | Authors / Publisher | Title | Publication | Year | URL | Cited In |
|----|---------------------|-------|-------------|------|-----|----------|
| REF-020 | Pease R (National Semiconductor / Analog Devices) | A Comprehensive Study of the Howland Current Pump | Application Note AN-1515 | 2008 | [TI AN-1515 (PDF)](https://www.ti.com/lit/an/snoa474a/snoa474a.pdf) | PITFALLS |
| REF-021 | Texas Instruments | Improved Howland Current Pump (SLOA097) | TI Application Report | 2013 | [TI SLOA097 (PDF)](https://www.ti.com/lit/an/sloa097/sloa097.pdf) | PITFALLS, ARCHITECTURE |
| REF-022 | Texas Instruments | Precision Voltage-to-Current Converter/Transmitter (SBAA290) | TI Application Report | 2018 | [TI SBAA290 (PDF)](https://www.ti.com/lit/an/sbaa290/sbaa290.pdf) ⚠️ VERIFY | PITFALLS |
| REF-023 | Microchip Technology | MCP4901/MCP4911/MCP4921/MCP4902/MCP4912/MCP4922 — 8/10/12-Bit DAC with SPI Interface (DS21897) | Datasheet | — | [Microchip DS21897](https://ww1.microchip.com/downloads/en/DeviceDoc/21897B.pdf) | ARCHITECTURE, PITFALLS |
| REF-024 | Texas Instruments | DRV8871 — 3.6A Brushed DC Motor Driver (SLVSCY4) | Datasheet | — | [TI DRV8871](https://www.ti.com/lit/ds/slvscy4d/slvscy4d.pdf) ⚠️ VERIFY | ARCHITECTURE |
| REF-025 | Silicon Labs | Si862x — High-Speed Digital Isolators | Datasheet | — | [Silicon Labs Si862x](https://www.silabs.com/interface/digital-isolators/si862x) | ARCHITECTURE |
| REF-026 | Horowitz P, Hill W | The Art of Electronics | Cambridge University Press | 2015 (3rd ed.) | [Publisher](https://artofelectronics.net/) | ARCHITECTURE |
| REF-027 | Franco S | Design with Operational Amplifiers and Analog Integrated Circuits | McGraw-Hill | 2015 (4th ed.) | [Publisher](https://www.mheducation.com/) | ARCHITECTURE |
| REF-028 | Espressif Systems | ESP32-S3 Technical Reference Manual | — | 2023 | [Espressif Docs](https://www.espressif.com/sites/default/files/documentation/esp32-s3_technical_reference_manual_en.pdf) | PITFALLS |

---

## H. Software & Analysis Tools

| ID | Tool / Library | Version | Purpose | URL | Cited In |
|----|----------------|---------|---------|-----|----------|
| REF-029 | NeuroKit2 | 0.2.13 | Python HRV analysis (RMSSD, pNN50, SDNN, LF/HF, DFA) | [GitHub](https://github.com/neuropsychology/NeuroKit) / [PyPI](https://pypi.org/project/neurokit2/) | SUMMARY, REQUIREMENTS |
| REF-030 | NimBLE-Arduino | ≥2.4.0 (rec: 2.5.0) | ESP32-S3 BLE dual-role stack | [GitHub](https://github.com/h2zero/NimBLE-Arduino) | SUMMARY, ARCHITECTURE, PITFALLS |
| REF-031 | Arduino-ESP32 | 3.3.8 | ESP32-S3 Arduino framework (wraps ESP-IDF 5.5.4) | [GitHub](https://github.com/espressif/arduino-esp32) | SUMMARY |
| REF-032 | PlatformIO (espressif32) | 6.13.0 | Build system | [PlatformIO Registry](https://registry.platformio.org/platforms/platformio/espressif32) | SUMMARY |
| REF-033 | KiCad | 9.0.8 | PCB design (GPL-compatible) | [KiCad.org](https://www.kicad.org/) | SUMMARY |

---

## I. Commercial Devices Referenced (for comparison)

Not academic citations, but product references used for the competitive comparison matrix in FEATURES.md.

| Device | Manufacturer | Type | Electrode Site | Key Specs | Product Page | Cited In |
|--------|-------------|------|---------------|-----------|-------------|----------|
| NEMOS | cerbomed (now tVNS Technologies) | Unilateral taVNS, CE-marked | Cymba conchae | 25 Hz, 250µs, 0.1–5mA | [cerbomed.com](https://cerbomed.com/) | FEATURES, SUMMARY, PITFALLS |
| Parasym | Parasym Ltd | Unilateral taVNS | Left tragus | 0–25 mA range (tragus/TENS hybrid) | [parasym.co](https://parasym.co/) | FEATURES, SUMMARY |
| Nuerisym | Nuerisym (Parasym-related) | Bilateral taVNS | Cymba conchae | Bilateral synchronous, ~0.5–5 mA | [nuerisym.com](https://nuerisym.com/) | FEATURES, SUMMARY, PITFALLS, PROJECT |
| gammaCore | electroCore | Transcervical VNS | Neck/cervical | Non-auricular — cited only as anti-pattern | [gammacore.com](https://www.gammacore.com/) | PITFALLS |
| Polar H10 | Polar Electro | BLE chest strap HRM | N/A | BLE HRS profile, RR intervals | [polar.com](https://www.polar.com/en/sensors/h10-heart-rate-sensor) | PROJECT, REQUIREMENTS, ARCHITECTURE |

---

## Missing / Unresolved References

References mentioned in the source files or task scope that could NOT be resolved to a specific paper with URL.

| What Was Referenced | Where Cited | Why Unresolved | Resolution Path |
|---------------------|-------------|----------------|-----------------|
| **"RESET-AF" trial name** | FEATURES, SUMMARY, PROJECT | Cannot confirm "RESET-AF" as an actual trial name. May be confused with Stavrakis 2015 pilot or TREAT-AF 2020. | Search [ClinicalTrials.gov](https://clinicaltrials.gov/search?term=RESET-AF+vagus+nerve+Stavrakis) and [PubMed](https://pubmed.ncbi.nlm.nih.gov/?term=RESET-AF+Stavrakis+atrial+fibrillation) |
| **Stavrakis 2023 TREAT-AF follow-up** | FEATURES | Source files reference a 2023 publication confirming TREAT-AF results. May exist as a follow-up or sub-analysis not in training data. | [Search PubMed](https://pubmed.ncbi.nlm.nih.gov/?term=Stavrakis+transcutaneous+vagus+atrial+fibrillation+2023) |
| **"25Hz / 200µs insomnia protocol"** specific study | FEATURES, PROJECT | No specific RCT cited. Parameters extrapolated from autonomic/cardiac trials. | [Search PubMed](https://pubmed.ncbi.nlm.nih.gov/?term=transcutaneous+auricular+vagus+nerve+stimulation+insomnia+25Hz) |
| **ABATIS trial** | Task scope (not in source files) | Not referenced in any source file. May be a planned/unpublished trial or different domain. | [Search ClinicalTrials.gov](https://clinicaltrials.gov/search?term=ABATIS+vagus+nerve+stimulation) |
| **taVNS bilateral vs. unilateral comparison study** | Task scope (not in source files) | No specific study cited. FEATURES.md acknowledges bilateral taVNS evidence is LOW. | [Search PubMed](https://pubmed.ncbi.nlm.nih.gov/?term=bilateral+transcutaneous+auricular+vagus+nerve+stimulation+comparison) |
| **Nuerisym insomnia case reports** | FEATURES, PROJECT | Referenced as "Nuerisym case reports suggest bedtime use for insomnia" — no specific citation. | Contact Nuerisym or search [PubMed](https://pubmed.ncbi.nlm.nih.gov/?term=Nuerisym+insomnia+vagus+nerve+stimulation) |
| **Dysautonomia / POTS / hEDS taVNS evidence** | REQUIREMENTS | FW-08 preset is labeled "⚠️ experimental, low clinical evidence" — no supporting paper cited. | [Search PubMed](https://pubmed.ncbi.nlm.nih.gov/?term=transcutaneous+vagus+nerve+stimulation+POTS+dysautonomia) |
| **Polar H10 as "gold standard consumer HRV"** validation | PROJECT | Claim made without citation. REF-012 (Gilgen-Ammann 2019) supports this but was not explicitly referenced. | Verify REF-012 DOI and add to PROJECT.md |
| **McCreery (2010) vs McCreery (1990)** date discrepancy | All safety sections | Source files cite "McCreery et al. (2010)" for 30µC/cm² limit, but this limit is from the 1990 paper (REF-013). | Correct year to 1990 in all documents. See Section F discrepancy note. |

---

## Parameter Discrepancy Audit

Cross-checking stimulation parameters cited in research files against known paper parameters.

| Parameter | Value in Source Files | Paper Cited | Actual Value in Paper | Discrepancy? |
|-----------|----------------------|-------------|----------------------|--------------|
| RESET-AF frequency | 20 Hz | "RESET-AF (Stavrakis, JACC 2020)" | Stavrakis 2015 (REF-004): 20 Hz ✓ | **Trial name uncertain, parameters match Stavrakis 2015 pilot** |
| RESET-AF pulse width | 200 µs | Same | Stavrakis 2015: 500 µs in some reports | **⚠️ NEEDS VERIFICATION — some Stavrakis papers report 500µs, not 200µs** |
| RESET-AF electrode site | Tragus | Same | Stavrakis 2015: Tragus ✓ | No discrepancy |
| TREAT-AF duty cycle | 30s ON / 30s OFF | "TREAT-AF (Stavrakis, 2023)" | TREAT-AF 2020 (REF-005): Continuous or intermittent — verify | **⚠️ NEEDS VERIFICATION** |
| McCreery charge density limit | 30 µC/cm² | "McCreery et al. (2010)" | McCreery 1990 (REF-013): 30 µC/cm² ✓ | **Year discrepancy: paper is from 1990, not 2010** |
| Shannon k-factor | k=1.75 (implied by 30µC/cm² limit) | "Shannon (1992)" | Shannon 1992 (REF-014): Model establishes k-factor ✓ | No discrepancy on model; verify k value |
| Cymba conchae ABVN density | ~100% | "Peuker & Filler (2002)" | Peuker 2002 (REF-001): 100% ABVN in cymba conchae ✓ | No discrepancy |
| Tragus ABVN density | ~45% | Same | Peuker 2002: Mixed innervation ✓ (exact % varies by interpretation) | Minor — "~45%" is a common simplification |
| HRV ESC/NASPE standards | RMSSD, SDNN, LF/HF | "Task Force (1996)" | REF-009: Defines all metrics ✓ | No discrepancy |
| HRV significance threshold | RMSSD increase >20% | REQUIREMENTS (BLE-03) | Training data consensus, not from a single paper | **Not from REF-009; appears to be from aggregated literature** |

---

## Recommended README Citation Block

The following formatted block is suitable for direct inclusion in the project's `README.md`, using numbered citations with DOI/PubMed links in GitHub-flavored Markdown.

```markdown
## Key References

The clinical parameters and safety limits in this project are grounded in peer-reviewed literature:

### Vagal Anatomy
1. Peuker ET, Filler TJ. "The nerve supply of the human auricle." *Clin Anat.* 2002;15(1):35-37. [DOI: 10.1002/ca.1089](https://doi.org/10.1002/ca.1089)

### taVNS Clinical Trials
2. Stavrakis S, Humphrey MB, Scherlag BJ, et al. "Low-level transcutaneous electrical vagus nerve stimulation suppresses atrial fibrillation." *J Am Coll Cardiol.* 2015;65(9):867-875. [DOI: 10.1016/j.jacc.2014.12.026](https://doi.org/10.1016/j.jacc.2014.12.026)
3. Stavrakis S, Stoner JA, Humphrey MB, et al. "TREAT AF: A Randomized Clinical Trial." *JACC Clin Electrophysiol.* 2020;6(1):1-11. [DOI: 10.1016/j.jacep.2019.11.008](https://doi.org/10.1016/j.jacep.2019.11.008)
4. Badran BW, et al. "Short trains of taVNS have parameter-specific effects on heart rate." *Brain Stimul.* 2018;11(4):699-708. [DOI: 10.1016/j.brs.2018.04.004](https://doi.org/10.1016/j.brs.2018.04.004)
5. Yakunina N, Kim SS, Nam EC. "Optimization of taVNS Using Functional MRI." *Neuromodulation.* 2017;20(3):290-300. [DOI: 10.1111/ner.12541](https://doi.org/10.1111/ner.12541)

### Vagal Neuroscience
6. Tracey KJ. "The inflammatory reflex." *Nature.* 2002;420:853-859. [DOI: 10.1038/nature01321](https://doi.org/10.1038/nature01321)
7. Tracey KJ. "Physiology and immunology of the cholinergic antiinflammatory pathway." *J Clin Invest.* 2007;117(2):289-296. [DOI: 10.1172/JCI30555](https://doi.org/10.1172/JCI30555)

### Stimulation Safety
8. McCreery DB, Agnew WF, Yuen TG, Bullara L. "Charge density and charge per phase as cofactors in neural injury." *IEEE Trans Biomed Eng.* 1990;37(10):996-1001. [DOI: 10.1109/10.102812](https://doi.org/10.1109/10.102812)
9. Shannon RV. "A model of safe levels for electrical stimulation." *IEEE Trans Biomed Eng.* 1992;39(4):424-426. [DOI: 10.1109/10.126616](https://doi.org/10.1109/10.126616)
10. Merrill DR, Bikson M, Jefferys JGR. "Electrical stimulation of excitable tissue: Design of efficacious and safe protocols." *J Neurosci Methods.* 2005;141(2):171-198. [DOI: 10.1016/j.jneumeth.2004.10.020](https://doi.org/10.1016/j.jneumeth.2004.10.020)

### HRV Standards
11. Task Force of the ESC/NASPE. "Heart rate variability: Standards of measurement." *Circulation.* 1996;93(5):1043-1065. [DOI: 10.1161/01.CIR.93.5.1043](https://doi.org/10.1161/01.CIR.93.5.1043)
12. Laborde S, Mosley E, Thayer JF. "Heart Rate Variability and Cardiac Vagal Tone in Psychophysiological Research." *Front Psychol.* 2017;8:213. [DOI: 10.3389/fpsyg.2017.00213](https://doi.org/10.3389/fpsyg.2017.00213)
13. Shaffer F, Ginsberg JP. "An Overview of Heart Rate Variability Metrics and Norms." *Front Public Health.* 2017;5:258. [DOI: 10.3389/fpubh.2017.00258](https://doi.org/10.3389/fpubh.2017.00258)

### Safety Standards
- IEC 60601-1:2005+AMD1:2012 — General requirements for medical electrical equipment
- IEC 60601-2-10 — Particular requirements for nerve and muscle stimulators
- ANSI/AAMI NS4:2024 — Transcutaneous electrical nerve stimulators

> ⚠️ **This device is NOT a medical device.** It has not been evaluated or cleared by any regulatory authority. The references above document the clinical and safety evidence informing the design; they do not imply certification or medical device status. For personal research use only.
```

---

## Cross-Reference Index: REF-ID → Source File Locations

For traceability, showing where each REF-ID maps to claims in the source files.

| REF-ID | Primary Claim Supported | Source File(s) | Specific Section |
|--------|------------------------|----------------|------------------|
| REF-001 | Cymba conchae has highest ABVN density (~100%) | FEATURES (Golden Params), PITFALLS (C-01) | Electrode site selection |
| REF-002 | Cholinergic anti-inflammatory reflex via vagus | REQUIREMENTS (FW-08) | Anti-inflammatory preset rationale |
| REF-003 | α7 nAChR pathway for inflammatory modulation | REQUIREMENTS (FW-08) | Anti-inflammatory preset rationale |
| REF-004 | 20 Hz, 200µs tragus stimulation suppresses AF | FEATURES (Golden Params), PROJECT | "RESET-AF" parameters |
| REF-005 | TREAT-AF confirmed in larger RCT | FEATURES (Sources table) | Parameter validation |
| REF-006 | Cymba conchae > tragus on fMRI | PITFALLS (C-01, Sources) | Electrode placement rationale |
| REF-007 | taVNS parameters have dose-dependent HR effects | PITFALLS (Sources) | Parameter optimization |
| REF-008 | Insomnia use case for taVNS | FEATURES, PROJECT | Bedtime use case reports |
| REF-009 | RMSSD, SDNN, LF/HF definitions and standards | FEATURES (D-2), PITFALLS (C-03) | HRV computation methodology |
| REF-010 | Practical HRV measurement guidelines | FEATURES (D-2) | Experimental protocol design |
| REF-011 | HRV metrics norms and interpretation | PITFALLS (C-03, Sources) | HRV measurement confounds |
| REF-012 | Polar H10 RR interval accuracy validation | PROJECT (implicit) | "Gold standard consumer HRV" claim |
| REF-013 | 30 µC/cm² charge density safety limit | FEATURES (TS-1, TS-2), PITFALLS (S-01), PROJECT | Charge density calculations |
| REF-014 | k-factor model for stimulation tissue damage | FEATURES (TS-1), PITFALLS (S-01), SUMMARY | Safety limit model |
| REF-015 | Electrical stimulation protocol design principles | PITFALLS (Sources) | Charge balance requirements |
| REF-016 | Patient isolation requirements, leakage limits | FEATURES (TS-5), ARCHITECTURE (§2.2) | Isolation barrier specification |
| REF-017 | Nerve stimulator specific requirements | FEATURES (TS-1), PITFALLS (Sources) | Charge balance standards |
| REF-018 | TENS device output standards | SUMMARY (Sources) | TENS hack compliance |
| REF-019 | Medical device QMS (deferred) | REQUIREMENTS, FEATURES (AF-10) | v2+ compliance path |
| REF-020 | Howland current pump design and failure modes | PITFALLS (H-01, Sources) | Why NOT to use Howland |
| REF-021 | Howland pump topology analysis | PITFALLS (Sources), ARCHITECTURE | Current source alternatives |
| REF-022 | V-to-I converter design | PITFALLS (Sources) | V-to-I implementation |
| REF-023 | MCP4922 LDAC timing, glitch energy | ARCHITECTURE (§2.1), PITFALLS (H-02) | DAC interface design |
| REF-024 | DRV8871 specs, dead-time, max voltage | ARCHITECTURE (§2.4) | H-bridge selection |
| REF-025 | Si8621 isolation specs, CMTI | ARCHITECTURE (§2.2) | Isolation barrier design |

---

## Summary Statistics

| Category | Count | HIGH Confidence | MEDIUM | LOW |
|----------|-------|-----------------|--------|-----|
| A. Foundational Neuroscience | 3 | 3 | 0 | 0 |
| B. taVNS Clinical Trials | 4 | 2 | 2 | 0 |
| D. Target Conditions | 2 new + refs from A/B | 0 | 0 | 2 |
| E. HRV Methodology | 4 | 3 | 1 | 0 |
| F. Safety Standards | 7 | 7 | 0 | 0 |
| G. Hardware/Technical | 9 | 7 | 2 | 0 |
| H. Software/Tools | 5 | 5 | 0 | 0 |
| **TOTAL unique refs** | **33** | **27** | **4** | **2** |
| **Unresolved** | **9** | — | — | — |

---

*Generated: 2025-07-14 | Reference confidence: HIGH for DOI format, MEDIUM for exact DOI values (from training data, flagged with ⚠️ VERIFY). All DOIs should be tested in a browser before publication.*
