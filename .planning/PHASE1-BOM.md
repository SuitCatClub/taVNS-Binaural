# Phase 1 — TENS Hack Bill of Materials (BOM)
**Safety-audited. Do not skip any item marked 🔴 or 🟡.**

---

## 🔴 MUST HAVE (can't start without)

### 1. TENS Unit — battery-powered, 2-channel

| | |
|---|---|
| **Amazon.es search** | `"electroestimulador TENS 2 canales pilas"` |
| **AliExpress search** | `"TENS unit 2 channel battery operated adjustable frequency"` |
| **Recommended model** | TENS 7000 2nd Edition |
| **Alternative** | Any generic 2-channel TENS with adjustable freq + PW |
| **Price range** | €15–35 |

> ⚠️ **SAFETY — battery only:** Must run on batteries during stimulation. Never connect charger or USB while electrodes are on your body. A USB/mains connection creates a ground loop through your body. Battery-only eliminates this risk completely.

> ⚠️ **Avoid:** Units with only "mode presets" and no raw frequency/pulse-width control. Need frequency setting at 20–25Hz.

---

### 2. DC-Blocking Capacitors — 10µF film (buy 4×, 2 per channel + 2 spares)

| | |
|---|---|
| **AliExpress search** | `"CBB21 capacitor 10uF 100V polypropylene"` |
| **Amazon.es search** | `"condensador film 10uF 100V polipropileno"` |
| **Mouser/DigiKey search** | `"film capacitor 10uF 100V"` — look for MKS or CBB21 type, through-hole |
| **Price** | ~€0.50–1.00 each |

> ✅ **Best pick: MKS / polypropylene (CBB21)** over MKT / polyester (CBB22). Lower loss, more stable, self-healing. At equal price, always pick polypropylene.

> ⚠️ **CRITICAL — film type only:** Must be **film** type — rectangular yellow/orange body. **NOT** cylindrical aluminum electrolytics (those are polarized and will fail dangerously). Look for "CBB21", "MKS", "polypropylene" in the description.

---

### 3. Bleed Resistors — 1MΩ, 1/4W (buy 4×)

Goes **in parallel** with each DC-blocking capacitor. Bleeds any accumulated charge between pulses. Without this, the capacitor charges up over the session and DC leaks through — causing chemical burns under the electrode over a 30-minute session.

| | |
|---|---|
| **AliExpress search** | `"resistor 1M ohm 1/4W metal film"` |
| **Amazon.es search** | `"resistencias 1 megaohm 1/4W metal film"` |
| **Price** | < €1 — included in any resistor assortment kit |

> ✅ **Best pick: Metal film (±1%)** over carbon film (±5%). Lower noise, tighter tolerance, more stable. Same price in assortment kits.

---

### 4. PTC Resettable Fuse — 10–15mA hold / 30mA trip (buy 2×)

Goes **in series** with each output. Passive, firmware-independent, hardware-only current limiter. If electronics fail or intensity is set too high, the fuse trips before dangerous current reaches you. Resets automatically when cooled.

| | |
|---|---|
| **AliExpress search** | `"Bourns MF-MSMF010 resettable fuse"` or `"PPTC resettable fuse 15mA"` |
| **Mouser / DigiKey** | Bourns `MF-MSMF010` (10mA hold, 20mA trip) or Littelfuse `1812L020` |
| **Price** | ~€0.30–0.50 each |

> ✅ **Best pick: Bourns MF-MSMF series** over generic Chinese PPTC. Well-documented trip curves — you know exactly when it triggers. For a safety-critical component between you and the circuit, brand matters.

---

### 5. Ear Electrodes — Cymba Conchae

The cymba conchae is the **upper hollow of the ear bowl** (above the antitragus) — NOT the earlobe, NOT the ear canal.

**Option A — Easiest:** Self-adhesive TENS pads (standard 2×2cm), cut into ~1cm circles, fixed with medical tape.

| | |
|---|---|
| **Amazon.es search** | `"electrodos TENS autoadhesivos"` (any brand, 40-pack) |
| **Price** | €5–10 |

**Option B — Better fit (recommended):** TENS ear clip electrodes (clip to ear, repositionable to cymba conchae).

| | |
|---|---|
| **AliExpress search** | `"ear electrode TENS clip auricular"` |
| **Amazon.es search** | `"electrodo auricular TENS oreja"` |
| **Price** | €5–12 |

> ✅ **Best pick: Ear clips (Option B).** Repositionable, consistent pressure, reusable across sessions. Cut pads shift on the cymba conchae and adhesive degrades with gel.

**Option C — Best:** Dedicated taVNS / auricular vagus nerve electrodes.

| | |
|---|---|
| **AliExpress search** | `"taVNS electrode auricular vagus nerve"` or `"tragus ear electrode"` |
| **Price** | €10–25 |

> ⚠️ **Minimum electrode area: 0.5 cm²** (1cm diameter minimum). Below this, charge density exceeds safe limits. Do NOT cut pads smaller than 1cm diameter.

---

### 6. Electrode Gel (conductive)

| | |
|---|---|
| **Where** | Farmacia or Amazon.es |
| **Amazon.es search** | `"Spectra 360 gel conductor"` or `"gel conductor electrodos TENS ECG"` |
| **Price** | €5–8 |
| **Why** | Reduces skin-electrode impedance to <5kΩ — required by Phase 1 success criteria. |

> ✅ **Best pick: Spectra 360 (Parker Labs)** if available. Clinical standard — salt-free formulation designed for long sessions. Generic TENS gels sometimes dry out mid-session.

---

### 7. Polar H10 — HRV verification

Required to validate Phase 1 actually works (success criterion 4: measurable RMSSD change pre/post session).

| | |
|---|---|
| **Amazon.es search** | `"Polar H10"` |
| **Price** | ~€60–80 |
| **Free app** | Elite HRV (iOS/Android), HRV4Training, or Polar Beat |
| **Note** | If you already own a chest-strap HR monitor, check if it exports RR intervals (some Garmin/Wahoo straps do) |

> ⚠️ **Safety use:** If heart rate drops >20 BPM from baseline during stimulation, stop immediately.

---

## 🟡 REQUIRED FOR VERIFICATION (success criteria require these)

### 8. Oscilloscope — REQUIRED before first skin contact, long-term lab tool

This is a one-time investment. It will be used across all 9 phases of this project **and every confirmed future project in the pipeline.** Buy it right, buy it once.

#### Why specs matter — full project pipeline

##### taVNS device (this project)

| Requirement | Source | Spec needed |
|-------------|--------|-------------|
| Verify 200µs biphasic pulse shape | Phase 1 success criteria | ≥10MHz bandwidth |
| Detect Howland/V-to-I oscillations at 500kHz+ | PITFALLS H-01: *"invisible on a slow oscilloscope"* | **≥100MHz bandwidth** |
| Verify H-bridge dead-time on all 4 gate signals | PITFALLS S-06: *"capture all four gate drive signals on a 4-channel oscilloscope"* | **4 channels** |
| Detect baseline DC drift over 60s at slow timebase | Phase 1 success criteria / PITFALLS S-02 | Deep memory (≥1M pts) |
| Debug MCP4922 SPI (up to 20MHz) | Phase 2 | Protocol decode (SPI/I2C) |
| Detect parasitic oscillations via spectrum | PITFALLS H-01 | FFT |

##### Future projects (confirmed pipeline)

| Project | Scope requirement | Why |
|---------|------------------|-----|
| **Hoverboard BLDC motor control** | 4 channels + 100MHz | Dead-time verification on all 4 gate signals (same as Phase 5/8 here); PWM at 8–20kHz; shoot-through detection |
| **Solar MPPT controller** | 100MHz + FFT | Switching regulators run at 50–200kHz; parasitic oscillations; duty cycle measurement |
| **Home automation (ESP32, MQTT, sensors)** | Protocol decode | I2C / SPI / UART bus debugging across multiple nodes; timing analysis |
| **Plant care system** | Protocol decode | Sensor buses (I2C/SPI), servo PWM, camera interfaces |

**The 4-channel 100MHz spec is independently required by 3 separate confirmed projects — not just this one.** A 2-channel or sub-100MHz scope is a false saving.

**Summary of minimum viable spec for the full pipeline:** 100MHz bandwidth, 4 channels, deep memory (≥1M pts), FFT, protocol decode (SPI/I2C/UART).

---

#### ❌ Do NOT buy — budget scopes under €50

| Model | Why not |
|-------|---------|
| DSO138 / DSO150 kit | 1MHz bandwidth — completely blind to Howland oscillations, H-bridge transitions, SPI signals. Not suitable even for Phase 1. |
| Any single-channel scope | Phase 5/8 H-bridge requires 4 simultaneous channels. |

---

#### Tier 1 — Tight budget, Phase 1–4 only (~€75–90)

**FNIRSI 1014D** — 2ch, 100MHz, 500Msps, FFT, SPI/I2C/UART decode, built-in AWG signal generator

| Spec | Value |
|------|-------|
| Bandwidth | 100MHz ✅ |
| Channels | 2 ⚠️ (Phase 5/8 needs 4ch — plan to upgrade) |
| Sample rate | 500Msps |
| Memory depth | 8K points ⚠️ (shallow — limits zoom resolution) |
| Protocol decode | SPI, I2C, UART ✅ |
| FFT | ✅ |
| Built-in AWG | ✅ bonus — useful for bench testing |
| Price | ~€75–90 AliExpress |

> **Verdict:** Gets you through Phase 1–4 cleanly. Will fail at Phase 5/8 H-bridge dead-time verification (4ch required) — and will also be insufficient for the hoverboard motor project for the same reason. If budget is genuinely the constraint right now, buy this as a temporary tool, but treat the upgrade to 4-channel as a firm commitment before Phase 5, not optional.

| | |
|---|---|
| **AliExpress search** | `"FNIRSI 1014D oscilloscope"` |

---

#### Tier 2 — Best value for the full project (~€150–220)

**Rigol DS1054Z** — 4ch, 50MHz (upgradeable to 100MHz), 12M memory, 1Gsps

| Spec | Value |
|------|-------|
| Bandwidth | 50MHz stock — **upgradeable to 100MHz** via free license key (widely documented, Rigol doesn't enforce) ✅ |
| Channels | **4** ✅ covers all 9 phases |
| Sample rate | 1Gsps |
| Memory depth | 12M points ✅ deep — capture long events and zoom without losing context |
| Protocol decode | SPI, I2C, RS232 (some decodable without paid license) |
| FFT | ✅ |
| Community | Enormous — 10 years of tutorials, forum posts, YouTube guides |
| Price | ~€280 new / **€150–200 used** (eBay, Wallapop) |

> **Verdict:** The right answer given your confirmed project pipeline. 4 channels covers every phase of this project, the hoverboard motor control project, and the solar MPPT controller. Deep memory is genuinely useful. Buy used if available — these are built to last and show no wear. The 100MHz unlock is universally done by hobbyists.

| | |
|---|---|
| **Search (new)** | `"Rigol DS1054Z"` — Mouser, Amazon, Farnell |
| **Search (used)** | `"Rigol DS1054Z"` on eBay, Wallapop |

---

#### Tier 3 — Buy once, use forever (~€290–340)

**Siglent SDS1104X-E** — 4ch, 100MHz native, 1Gsps, 14M memory, active firmware development

| Spec | Value |
|------|-------|
| Bandwidth | **100MHz native** (no unlock needed) ✅ |
| Channels | **4** ✅ |
| Sample rate | 1Gsps |
| Memory depth | 14M points ✅ |
| Protocol decode | SPI, I2C, UART, CAN ✅ |
| FFT | ✅ |
| PC software | SDS software suite, active updates |
| Build quality | Professional — noticeably better analog front-end than Rigol DS1054Z |
| Price | ~€290–340 |

> **Verdict:** If you are going to run this project through all 9 phases AND build the hoverboard, solar, and automation projects after it, this is the correct tool. Better analog front-end than the Rigol, native 100MHz, active firmware updates from Siglent, and a scope that will still be excellent in 15 years. The ~€100 premium over a used Rigol is justified across the full pipeline.

| | |
|---|---|
| **Search** | `"Siglent SDS1104X-E"` — Amazon.es, Farnell, RS Components |

---

#### Summary — recommendation updated for full project pipeline

Given the confirmed future projects (hoverboard BLDC, solar MPPT, home automation), the Tier 1 FNIRSI is a **temporary bridge only** — it will hit its limit in this project at Phase 5, and again in every motor/power electronics project. Tier 1 is only justified if cash is the hard constraint right now.

| Budget | Buy | Gets you through |
|--------|-----|-----------------|
| <€100 | FNIRSI 1014D | Phase 1–4 only. **Must upgrade before Phase 5 and before hoverboard/solar projects. Treat as temporary.** |
| €150–200 | Rigol DS1054Z (used) ← **recommended** | All 9 phases + hoverboard + solar + automation. Best value for the full pipeline. |
| €300+ | Siglent SDS1104X-E ← **recommended if buying new** | All 9 phases + every future project + 15 years. Better analog performance, native 100MHz, active support. |

---

### 9. Multimeter with mV DC range — REQUIRED, long-term lab tool

Needed in Phase 1 to verify DC offset <1mV across dummy load. Used in every phase after.

> ℹ️ **User note:** You mentioned having an existing multimeter — share the model when you find it and it will be assessed against the requirements below. It may well be sufficient.

#### What this project needs

- **22,000 count (4.5 digit) minimum** — a standard 2000-count (3.5 digit) meter reads 1mV only at its lowest voltage range. At 2V range it reads 1mV. At 20V range it reads 10mV — too coarse. A 4.5-digit / 22,000-count meter gives 0.1mV resolution on the 200mV range ✅
- **True RMS** — for accurately measuring the RMS current through the sense resistor from a non-sinusoidal biphasic waveform. Average-responding meters give wrong readings on pulsed waveforms
- **Auto-ranging** — quality-of-life, saves time during bench work

---

#### Option A — Best value (~€30–40)

**UNI-T UT61E+** — 22,000 count, 4.5 digit, True RMS, auto-ranging, USB data logging

| Spec | Value |
|------|-------|
| Count | 22,000 (4.5 digit) ✅ |
| DC voltage resolution | 0.1mV on 220mV range ✅ |
| True RMS | ✅ |
| Auto-ranging | ✅ |
| USB logging | ✅ (useful for logging DC offset over 10-minute test) |
| Price | ~€30–40 AliExpress / Amazon |

> **Verdict:** The go-to recommendation in the DIY electronics community. Punches well above its price. Handles everything this project needs. If you don't own a multimeter yet, this is what to buy.

| | |
|---|---|
| **AliExpress search** | `"UNI-T UT61E+ multimeter"` |
| **Amazon.es search** | `"UNI-T UT61E multimetro"` |

---

#### Option B — Buy once, use forever (~€100–130)

**Fluke 115** — True RMS, 6000 count, CAT III safety rating, lifetime build quality

| Spec | Value |
|------|-------|
| Count | 6,000 (3.5 digit) — lower count than UT61E+ but 1mV resolution at 400mV range ✅ |
| True RMS | ✅ |
| Build quality | Legendary — still working after 20 years in field conditions |
| Safety rating | CAT III 600V — relevant if you ever work near mains voltage |
| Warranty | Fluke's reputation is the warranty |
| Price | ~€100–130 |

> **Verdict:** If you want a multimeter that outlasts you: Fluke 115. The UNI-T does everything needed for this project — the Fluke adds ruggedness, CAT III safety, and the peace of mind of a professional tool. Worth it if electronics is a long-term hobby.

---

### 10. Resistors — dummy load + current sense

| Part | Value | Qty | Purpose |
|------|-------|-----|---------|
| Dummy load | **1kΩ, 1W, ±1%** | 2× | Connect across output to verify waveform before skin |
| Current sense | **10Ω, 1W, ±1%** | 2× | In-series measurement: V=IR → 1mA=10mV |

> ✅ **Best pick: Metal film ±1%** over carbon film ±5%. When measuring mV of DC offset across the dummy load, a ±5% resistor adds uncertainty you don't need.

Any metal film resistor assortment kit on AliExpress (~€3) includes both values.

---

### 11. Heat Shrink Tubing — REQUIRED

All bare connections anywhere near the body must be insulated. Not optional.

| | |
|---|---|
| **AliExpress search** | `"dual wall heat shrink tubing adhesive lined assortment"` |
| **Amazon.es search** | `"termorretráctil doble pared adhesivo surtido"` |
| **Price** | ~€3–5 |

> ✅ **Best pick: Dual-wall with adhesive lining** over standard single-wall. Melts and seals — no moisture ingress. Matters near skin and conductive gel.

---

## 🟢 NICE TO HAVE

| Item | Search | ~Price |
|------|--------|--------|
| Medical tape (Micropore) | `"micropore tape"` — farmacia | €2 |
| Alligator clip cables | `"cables cocodrilo laboratorio"` | €3–5 |
| Hookup wire 22 AWG | `"cable puente arduino"` | €3 |
| Small breadboard | `"breadboard mini"` | €3 |

---

## 💰 Total Cost Estimate

| Category | Cost |
|----------|------|
| TENS unit | €15–35 |
| DC-blocking caps 10µF film (×4) | €3–5 |
| Bleed resistors 1MΩ (×4) | ~€0 (in resistor kit) |
| PTC fuses 10–15mA (×2) | €1–3 |
| Ear electrodes | €5–20 |
| Electrode gel | €5–8 |
| Polar H10 | €60–80 |
| Resistors assortment kit | €3–5 |
| Heat shrink | €3 |
| Misc (tape, wire) | €5–8 |
| **Subtotal (consumables only)** | **~€100–167** |
| | |
| **+ Oscilloscope — FNIRSI 1014D** | +€75–90 → covers Phase 1–4 only |
| **+ Oscilloscope — Rigol DS1054Z (used)** | +€150–200 → covers all 9 phases ← recommended |
| **+ Oscilloscope — Siglent SDS1104X-E** | +€290–340 → all 9 phases + long-term |
| **+ Multimeter — UNI-T UT61E+** | +€30–40 ← if buying new |
| **+ Multimeter — Fluke 115** | +€100–130 ← if you want it to last forever |
| **+ Multimeter — existing** | €0 ← pending model assessment |

---

## 🔴 PRE-USE PROTOCOL — mandatory before first skin contact

1. Assemble full test circuit: TENS → PTC fuse → DC cap + bleed resistor in parallel → 1kΩ dummy load
2. Power via **batteries only** — no USB, no charger connected
3. Set intensity to minimum, connect oscilloscope to dummy load
4. Verify waveform is biphasic — positive phase area matches negative within ±2%
5. Measure DC offset with multimeter (mV DC range) across dummy load — must be **<1mV** after 10 minutes at max intensity
6. Test PTC fuse trips before ~30mA (briefly short output through low resistance — fuse should go warm/trip)
7. Only after all checks pass → apply to skin, **one ear only** (no simultaneous binaural until Phase 5 stagger hardware is built)
8. Start at **minimum intensity**, ramp up slowly until just perceptible
9. **Never use standing up** — vagal stimulation can cause vasovagal syncope (fainting)
10. Session max: 30 minutes

---

## 🚫 Absolute Contraindications — do not use if:

- Cardiac pacemaker or ICD (implantable cardioverter-defibrillator)
- Active epilepsy
- Pregnancy
- Prior vagotomy (surgical cutting of vagus nerve)

---

## 📋 Circuit Diagram — Phase 1 Output Stage (per channel)

```
TENS output (+) ──[PTC fuse 15mA]──[10µF film cap]──┬── Electrode (cymba conchae)
                                                     │
                                                  [1MΩ bleed]
                                                     │
TENS output (−) ─────────────────────────────────────┴── Counter electrode (back of ear)
```

**Test points:**
- For waveform check: connect oscilloscope across dummy load (replace electrodes with 1kΩ resistor)
- For DC offset check: connect multimeter DC mV range across the 1kΩ dummy load
- For current check: put 10Ω in series, measure mV across it → divide by 10 = mA

---

*Generated: 2026-04-15 | Phase 1 of 9 | OpenBinaural-taVNS | v1.0*
