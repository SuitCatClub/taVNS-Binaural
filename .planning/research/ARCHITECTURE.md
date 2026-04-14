# Architecture Patterns — OpenBinaural-taVNS

**Domain:** DIY precision binaural transcutaneous auricular vagus nerve stimulation
**Researched:** 2025-07-13
**Overall confidence:** HIGH for analog signal chain and safety architecture (well-established bioelectronics design principles, mature component ecosystem). MEDIUM for ESP32-S3 BLE dual-role (functional but edge-case stability depends on NimBLE version and connection parameters).

> **Design philosophy:** This is a **safety-first** architecture. Every signal path, power rail, and firmware state is designed so that any single failure mode (hardware fault, firmware crash, user error) results in zero current to the patient. Hardware enforces safety. Firmware optimizes the experience. Software can fail — physics cannot.

---

## 1. System-Level Block Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           DIGITAL DOMAIN (Non-Isolated)                         │
│                                                                                 │
│  ┌──────────┐    ┌──────────┐    ┌────────────┐    ┌────────────────────────┐  │
│  │  USB-C    │    │ TP4056   │    │ LiPo       │    │ Boost 3.7V→5V         │  │
│  │ (charge   │───▸│ Charger  │───▸│ 3.7V       │───▸│ (TPS61023/MT3608)     │  │
│  │  only)    │    │ +protect │    │ 2000mAh    │    │ 5V @ 1A digital rail  │  │
│  └──────────┘    └──────────┘    └────────────┘    └─────────┬──────────────┘  │
│                                                               │                 │
│  ┌────────────────────────────────────────────────────────────┤                 │
│  │                                                            │                 │
│  │  ┌─────────────┐    ┌────────┐    ┌────────────┐          │                 │
│  │  │ ESP32-S3    │    │ VBUS   │    │ Status LED │          │                 │
│  │  │             │    │ Detect │    │ (RGB)      │          │                 │
│  │  │ Core 0:     │    │ GPIO   │    └────────────┘          │                 │
│  │  │  NimBLE     │    └────────┘                            │                 │
│  │  │  Session Mgr│                                          │                 │
│  │  │  Flash Log  │                                          │                 │
│  │  │             │    SPI bus (SCK, MOSI, CS_A, CS_B, LDAC) │                 │
│  │  │ Core 1:     │────────────────────────────────┐         │                 │
│  │  │  Stim ISR   │    Heartbeat PWM (100Hz)───────┤         │                 │
│  │  │  Impedance  │    E-Stop GPIO ◄───────────┐   │         │                 │
│  │  │  Watchdog   │                            │   │         │                 │
│  │  └─────────────┘                            │   │         │                 │
│  │                                             │   │         │                 │
│  │  ┌──────────────────┐    ┌─────────────┐    │   │         │                 │
│  │  │ Hardware E-Stop  │    │ VBUS lockout│    │   │         │                 │
│  │  │ Button (NO, pull │───▸│ AND gate:   │    │   │         │                 │
│  │  │ up to 3.3V)      │    │ !VBUS &     │    │   │         │                 │
│  │  └──────────────────┘    │ !E-STOP     │    │   │         │                 │
│  │                          └──────┬──────┘    │   │         │                 │
│  │                                 │           │   │         │                 │
│  └─────────────────────────────────┼───────────┘   │         │                 │
│                                    │               │         │                 │
│ ═══════════════════════════════════╪═══════════════╪═════════╪═════════════════ │
│         ISOLATION BARRIER          │               │         │                 │
│     (1500V reinforced)             │               │         │                 │
│ ═══════════════════════════════════╪═══════════════╪═════════╪═════════════════ │
│                                    │               │         │                 │
│                           PATIENT DOMAIN (Isolated)│         │                 │
│                                    │               │         │                 │
│  ┌─────────────────────────────────┼───────────────┼─────────┼──────────────┐  │
│  │                                 │               │         │              │  │
│  │  ┌──────────────┐  ┌───────────┴───────────┐   │  ┌──────┴───────────┐ │  │
│  │  │ B0515S-1WR3  │  │ Si8621 Digital        │   │  │ Heartbeat        │ │  │
│  │  │ Isolated     │  │ Isolator              │   │  │ Detector         │ │  │
│  │  │ DC-DC        │  │ (4-ch: SCK,MOSI,     │   │  │ (RC + comparator)│ │  │
│  │  │ 5V→±15V     │  │  CS, LDAC)            │   │  │ Gates analog VDD │ │  │
│  │  │ 1W, 1500VDC │  └───────────┬───────────┘   │  └──────┬───────────┘ │  │
│  │  └──────┬───────┘              │               │         │             │  │
│  │         │                      │               │         │             │  │
│  │  ┌──────┴───────┐   ┌─────────┴────────┐      │  ┌──────┴───────────┐ │  │
│  │  │ Post-reg LDO │   │ MCP4922          │      │  │ Analog VDD       │ │  │
│  │  │ LM317→+12V   │   │ Dual 12-bit DAC  │      │  │ Power Switch     │ │  │
│  │  │ LM337→-12V   │   │ (SPI, LDAC sync) │      │  │ (P-ch MOSFET)    │ │  │
│  │  │ Clean analog │   └────┬────────┬────┘      │  └──────────────────┘ │  │
│  │  │ rails        │        │Ch A    │Ch B       │                       │  │
│  │  └──────────────┘        │        │           │                       │  │
│  │                          ▼        ▼           │                       │  │
│  │              ┌────────────────────────────┐    │                       │  │
│  │              │  CHANNEL A    CHANNEL B    │    │                       │  │
│  │              │  (identical, independent)  │    │                       │  │
│  │              │                            │    │                       │  │
│  │              │  RC filter (1kΩ+100nF)     │    │                       │  │
│  │              │       │                    │    │                       │  │
│  │              │  Op-amp V-to-I             │    │                       │  │
│  │              │  (OPA2277 + Rsense 100Ω)   │    │                       │  │
│  │              │       │                    │    │                       │  │
│  │              │  H-bridge (DRV8837)        │    │                       │  │
│  │              │  (built-in dead-time)      │    │                       │  │
│  │              │       │                    │    │                       │  │
│  │              │  Overcurrent comparator    │    │                       │  │
│  │              │  (LM393 + latch)           │    │                       │  │
│  │              │       │                    │    │                       │  │
│  │              │  DC-blocking cap           │    │                       │  │
│  │              │  (10µF film + 1MΩ bleed)   │    │                       │  │
│  │              │       │                    │    │                       │  │
│  │              │  ┌────┴────┐               │    │                       │  │
│  │              │  │Electrode│               │    │                       │  │
│  │              │  │ pair    │               │    │                       │  │
│  │              │  └─────────┘               │    │                       │  │
│  │              └────────────────────────────┘    │                       │  │
│  └────────────────────────────────────────────────┘                       │  │
│                                                                           │  │
└───────────────────────────────────────────────────────────────────────────┘  │
```

---

## 2. Hardware Architecture — Signal Chain (Per Channel)

### 2.1 Complete Signal Path

```
ESP32-S3 (Core 1 ISR)
    │
    │  SPI @ 20MHz (SCK, MOSI, CS, LDAC)
    ▼
┌──────────────────┐
│ Si8621 Digital   │ ◄── ISOLATION BARRIER (1500V reinforced)
│ Isolator (4-ch)  │
└────────┬─────────┘
         │  Isolated SPI
         ▼
┌──────────────────┐
│ MCP4922 DAC      │  12-bit, dual-channel, Vref = 2.048V (internal or MCP4901)
│ Ch A: 0–2.048V   │  Resolution: 0.5mV/LSB → 5µA/LSB at Rsense=100Ω
│ Ch B: 0–2.048V   │  Settling: 4.5µs (well within 250µs pulse)
└────────┬─────────┘
         │  Analog voltage (0–2.048V per channel)
         ▼
┌──────────────────┐
│ RC Low-Pass      │  1kΩ series + 100nF to GND
│ Filter           │  fc = 1.6kHz (kills DAC glitch energy, passes 25Hz stim)
└────────┬─────────┘
         │  Filtered control voltage
         ▼
┌──────────────────────────────────────────────────┐
│ V-to-I Converter (Op-amp + Rsense feedback)      │
│                                                   │
│   Vdac ──►[+]─┐                                  │
│                │ OPA2277                           │
│         ┌─[−]─┘                                  │
│         │   │                                     │
│         │   ├──► Output to H-bridge               │
│         │   │                                     │
│         └───┤                                     │
│             │                                     │
│          Rsense (100Ω, 0.1%, 250mW)              │
│             │                                     │
│            GND_ISO                                │
│                                                   │
│  I_out = Vdac / Rsense                           │
│  At Vdac=0.5V: I_out = 5mA (full scale)         │
│  At Vdac=0.1mV: I_out = 1µA (below noise floor) │
└──────────────────────┬───────────────────────────┘
                       │  Constant current (unidirectional)
                       ▼
┌──────────────────────────────────────────────────┐
│ H-Bridge Driver: DRV8837 (TI)                   │
│                                                   │
│  IN1  IN2  │ Function                            │
│  ───────────┼──────────────                       │
│   0    0   │ Coast (Hi-Z) — default safe state   │
│   0    1   │ Forward: current L→R through load   │
│   1    0   │ Reverse: current R→L through load   │
│   1    1   │ Brake (both low-side ON) — avoid     │
│                                                   │
│  Built-in shoot-through protection                │
│  Built-in dead-time (~300ns)                     │
│  Max 1.8A (massive headroom for 5mA)             │
│  Logic-level inputs (3.3V compatible)            │
│  nSLEEP pin: LOW = disabled (safe default)       │
└──────────────────────┬───────────────────────────┘
                       │  Biphasic current output
                       ▼
┌──────────────────────────────────────────────────┐
│ Hardware Overcurrent Protection                   │
│                                                   │
│  Sense resistor: 10Ω (in series with output)     │
│  At 5mA: V_sense = 50mV                          │
│  At 6mA: V_sense = 60mV (trip threshold)         │
│                                                   │
│  LM393 Comparator:                               │
│    V+ = V_sense (across 10Ω)                     │
│    V− = 60mV reference (voltage divider)         │
│    Output: open-collector, pulls FAULT_LATCH low │
│                                                   │
│  SR Latch (74LVC1G79 or discrete):               │
│    SET = comparator output (overcurrent)         │
│    RESET = manual pushbutton only                │
│    Q = drives DRV8837 nSLEEP to LOW (kill output)│
│    Q also drives GPIO to ESP32 (fault notification│
└──────────────────────┬───────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────┐
│ DC-Blocking Capacitor                            │
│                                                   │
│  10µF polypropylene film (Vishay MKP or WIMA)    │
│  NOT ceramic (voltage coefficient, piezoelectric)│
│  NOT electrolytic (polarity, ESR, leakage)       │
│                                                   │
│  1MΩ bleed resistor in parallel                  │
│  (discharges accumulated offset during rest)     │
│                                                   │
│  Voltage rating: ≥50V (handles compliance rail)  │
└──────────────────────┬───────────────────────────┘
                       │
                       ▼
                 ┌───────────┐
                 │ Electrode │  Cymba conchae (active)
                 │   Pair    │  Earlobe (reference)
                 └───────────┘
```

### 2.2 Isolation Barrier — What, Where, Why

**The isolation barrier sits between the ESP32-S3 digital domain and ALL patient-facing analog circuitry.** Nothing on the patient side shares a ground reference with USB, the charger, or the digital MCU.

#### What crosses the barrier:

| Signal | Direction | Crossing Method | Part | Why |
|--------|-----------|----------------|------|-----|
| **SPI Clock (SCK)** | Digital → Patient | Si8621BB digital isolator | Channel 1 | DAC clocking |
| **SPI Data (MOSI)** | Digital → Patient | Si8621BB digital isolator | Channel 2 | DAC data |
| **Chip Select (CS)** | Digital → Patient | Si8621BB digital isolator | Channel 3 | DAC select |
| **LDAC** | Digital → Patient | Si8621BB digital isolator | Channel 4 | Atomic DAC update |
| **H-bridge IN1/IN2** | Digital → Patient | Si8622BB (2nd isolator, 2ch per bridge) | 4 additional channels | Polarity control |
| **Heartbeat (100Hz PWM)** | Digital → Patient | One channel of Si8621 (or separate optocoupler) | — | Watchdog enable |
| **Fault flag** | Patient → Digital | Si8621 reverse channel or optocoupler | — | Overcurrent notification |
| **Power** | Digital → Patient | B0515S-1WR3 isolated DC-DC | — | ±15V analog supply |

**Total isolation channels needed:** ~9 digital signals. Use 3× Si8621BB (dual-channel bidirectional) or 2× Si8642BB (4-channel). The exact isolator IC choice depends on channel count vs. package count tradeoff in PCB layout.

#### What does NOT cross the barrier:

| Signal | Why It Stays Put |
|--------|-----------------|
| **Analog DAC output** | DAC is on the patient side — no analog signal crosses |
| **Electrode current** | Entirely within patient domain |
| **Impedance sense voltage** | ADC for impedance is on the patient side (dedicated low-cost ADC like MCP3201, or use MCP4922's unused features + external ADC) |
| **Ground reference** | Patient ground (GND_ISO) and digital ground (GND_DIG) connect ONLY through the isolation barrier's parasitic capacitance (~10pF). They are electrically independent. |

#### Isolation specification:

| Parameter | Requirement | Justification |
|-----------|-------------|---------------|
| **Working voltage** | 30V (±15V rails) | Maximum analog rail differential |
| **Isolation voltage** | ≥1500 VDC (reinforced) | IEC 60601-1 Type BF patient isolation for battery-powered devices. Protects against mains fault coupling through USB charger. |
| **Leakage current** | <10 µA at 240 VAC | IEC 60601-1 patient auxiliary current limit |
| **CMTI** | >25 kV/µs | Si8621 provides 150 kV/µs — more than sufficient |

**Confidence: HIGH** — This isolation topology (digital signals via digital isolator + power via isolated DC-DC) is the standard architecture used in commercial neurostimulation devices (Medtronic, Boston Scientific implant programmers) and laboratory stimulators (Digitimer DS5, AM Systems 2200). The approach is proven over decades.

---

### 2.3 Constant-Current Source — V-to-I with Op-Amp Feedback

**Decision: Use simple V-to-I with op-amp + sense resistor, NOT Howland current pump.**

#### Why NOT Howland:

| Issue | Severity | Detail |
|-------|----------|--------|
| **Resistor matching** | HIGH | Howland output impedance depends on R-match to 0.01%. Even 0.1% resistors give only ~60dB output impedance. At 5mA into 5kΩ load, 0.1% mismatch causes ~5µA error — tolerable, but a tighter budget than needed. |
| **Oscillation risk** | HIGH | Electrode capacitance (10–100nF) + Howland feedback network creates a pole that degrades phase margin. Oscillation at 100kHz–1MHz is common and invisible on a normal scope. This is the #1 cause of Howland failures in neurostim designs (see PITFALLS.md H-01). |
| **Unnecessary complexity** | MEDIUM | The Howland pump's main advantage is a floating current source (both load terminals are independent of ground). But the H-bridge already provides load floating — the load doesn't need to be ground-referenced. |
| **Biphasic overhead** | MEDIUM | A single Howland pump can source and sink, but requires bipolar DAC drive (positive and negative voltage). With a unidirectional V-to-I + H-bridge, the DAC only outputs positive voltage and the H-bridge handles polarity. Simpler firmware, simpler analog. |

#### Why V-to-I + Rsense:

```
         Vdac (0–0.5V from MCP4922 via RC filter)
           │
           ▼
    ┌──────+──────┐
    │      │      │
    │   [+]│      │
    │      │ OPA2277 (or OPA2340 for rail-to-rail)
    │   [−]│      │
    │      │      │
    │    ┌─┴──────┤
    │    │  │     │
    │    │  │     └──────► I_out to H-bridge
    │    │  │
    │    │  Rsense = 100Ω (0.1% thin-film)
    │    │  │
    │    │  GND_ISO
    │    │
    │    └── Feedback: V(−) tracks V(+)
    │         Therefore: V_Rsense = Vdac
    │         Therefore: I_out = Vdac / Rsense = Vdac / 100
    │
    │  Full scale: Vdac = 0.5V → I_out = 5mA
    │  Resolution: 0.5V / 4096 = 122µV → 1.22µA per LSB
    │  Clinically useful: 0.1mA = 82 LSBs → excellent resolution
```

**Op-amp selection: OPA2277** (dual, one per channel)
- Input offset: ±20µV max → 0.2µA DC offset (negligible, DC-blocking cap catches the rest)
- GBW: 1MHz (bandwidth at gain=1 is 1MHz, sufficient for 250µs pulses with 10µs edges)
- Rail-to-rail output: No (but doesn't need to be — output swing is 0–0.5V, well within ±12V rails)
- Supply: ±2.25V to ±18V (works on ±12V post-regulated rails)
- Slew rate: 0.8V/µs → 0.5V step settles in <1µs
- Unconditionally stable at unity gain in this topology

**Alternative considered:** OPA388 (chopper-stabilized, 0.25µV offset) — better offset but costs 3× more and chopper noise can appear in the output. OPA2277 offset of ±20µV is already well within tolerance for this application. Use OPA388 only if DC offset testing shows the OPA2277 introduces measurable charge imbalance.

#### Compliance voltage analysis:

The critical question: how much load impedance can the current source drive?

```
Compliance Voltage = V_supply_rail − V_op_amp_saturation − V_Rsense − V_Hbridge_Rdson − V_DC_block_cap

Where:
  V_supply_rail = ±12V (post-regulated from ±15V isolated DC-DC)
  V_op_amp_saturation = 2V (OPA2277 output swings to within 2V of rail)
  V_Rsense = I × 100Ω = 5mA × 100Ω = 0.5V
  V_Hbridge_Rdson = 5mA × 0.5Ω (DRV8837 Rdson) = 2.5mV (negligible)
  V_DC_block_cap = ~0V at steady state (charges/discharges each half-cycle)

Available compliance = 12V − 2V − 0.5V − 0.003V ≈ 9.5V per polarity

Maximum load impedance at full current:
  Z_max = 9.5V / 5mA = 1.9kΩ

Wait — this is too tight. Let me reconsider the supply rails.
```

**Revised with ±15V raw (no post-regulation) and rail-to-rail op-amp:**

Using **OPA2340** (rail-to-rail output, output swings to within 50mV of rail):
```
Available compliance = 15V − 0.05V − 0.5V − 0.003V ≈ 14.4V per polarity
Z_max at 5mA = 14.4V / 5mA = 2.88kΩ
Z_max at 3mA = 14.4V / 3mA = 4.8kΩ  ← clinical sweet spot
Z_max at 1mA = 14.4V / 1mA = 14.4kΩ ← low intensity, high compliance
```

**But OPA2340 runs on ±2.7V max, not ±15V.** Need a different rail-to-rail op-amp.

**Revised op-amp selection: OPA2209** or **AD8676** for high supply voltage:
- AD8676: ±18V supply, 2.5V from rail output swing, low noise
  - Compliance = 15V − 2.5V − 0.5V = 12V → Z_max = 2.4kΩ at 5mA

**Or better: Reduce Rsense to lower its voltage drop:**

Using **Rsense = 20Ω** (0.1% thin-film):
```
V_Rsense at 5mA = 5mA × 20Ω = 100mV
Full-scale Vdac needed = 100mV (achievable with Vref=0.5V or voltage divider on MCP4922 output)
Compliance = 15V − 2V − 0.1V = 12.9V
Z_max at 5mA = 12.9V / 5mA = 2.58kΩ
Z_max at 3mA = 12.9V / 3mA = 4.3kΩ
```

**FINAL DESIGN DECISION: Use Rsense = 100Ω with ±15V rails and OPA2277:**

```
Compliance = 15V − 2V − 0.5V = 12.5V
Z_max at 5mA = 12.5V / 5mA = 2.5kΩ
Z_max at 3mA = 12.5V / 3mA = 4.17kΩ
Z_max at 2mA = 12.5V / 2mA = 6.25kΩ
```

The 100Ω sense resistor provides better SNR for the overcurrent comparator (50mV at 5mA vs. 10mV with 20Ω) at the cost of 0.5V compliance. This tradeoff is correct because:

1. **Well-prepared cymba conchae electrodes: 500Ω–3kΩ** — within compliance at all clinical currents
2. **Dry/poor contact: >5kΩ** — out of compliance at 5mA but within compliance at 2mA (which is the sub-threshold clinical range anyway)
3. **The device MUST refuse to stimulate when compliance is exceeded** — this is a feature, not a limitation. Driving into an out-of-compliance load means the electrode contact is bad.

#### The 50V compliance problem (5mA × 10kΩ) — explicit resolution:

**This is not a real operating scenario.** 10kΩ electrode impedance means:
- Electrode is detached or nearly so
- No conductive gel
- Wrong placement (not on skin)

**No battery-powered portable device achieves 50V compliance.** Commercial taVNS devices (NEMOS, Nuerisym, Parasym) all have compliance in the 10–25V range. The correct engineering response is:

1. **Impedance monitoring detects the condition** before stimulation begins
2. **Compliance detection** during stimulation (op-amp output approaching rail) triggers current ramp-down
3. **User notification**: "Electrode impedance too high — check contact and gel"
4. **Hard limit**: If Z > 5kΩ, refuse to start. If Z > 3kΩ, warn user. If Z rises above compliance during session, auto-pause with ramp-down.

**Confidence: HIGH** — This compliance range matches all portable battery-powered neurostimulation devices on the market.

---

### 2.4 Biphasic Generation — H-Bridge Timing

**Use DRV8837 integrated H-bridge driver** (TI, SOT-23-6, ~$1.50)

#### Why DRV8837 over discrete MOSFETs:

| Factor | DRV8837 | Discrete H-bridge |
|--------|---------|-------------------|
| Shoot-through protection | Built-in, hardware-guaranteed | Must implement with dead-time circuit |
| Dead-time | ~300ns internal, automatic | Manual RC delay or MCPWM configuration |
| Component count | 1 IC + 2 decoupling caps | 4 MOSFETs + 4 gate resistors + 2 gate drivers + dead-time circuit |
| PCB area | 3mm × 3mm SOT-23-6 | ~100mm² minimum |
| Failure modes | Well-characterized, single IC | Multiple discrete failure modes |
| Cost | ~$1.50 | ~$3–5 in discrete components |
| Max voltage | 11V (sufficient for our ±12V unipolar drive) | Depends on MOSFET selection |

**CRITICAL NOTE on DRV8837 voltage:** The DRV8837 is rated for 0–11V VM (motor supply). Since our V-to-I converter output is 0–0.5V (into the Rsense), and the H-bridge is switching the *current source output* not the *supply rail*, the H-bridge sees a voltage of:

```
V_Hbridge = I_load × Z_load = 5mA × 3kΩ = 15V  ← EXCEEDS DRV8837 11V LIMIT
```

**This is a problem.** The H-bridge sits between the current source output and the load. It sees the compliance voltage across the load, not just the sense voltage.

**Corrected topology:** The H-bridge must be rated for the full compliance voltage. Options:

1. **DRV8871** — Same family, 45V max, H-bridge driver with current sense. ~$2.50. **Use this.**
2. **L9110S** — Dual H-bridge, 12V max. Not sufficient.
3. **Discrete MOSFETs** — Use BSS138 (60V N-ch) + Si2301 (20V P-ch). Adds complexity. Use only if DRV8871 is unavailable.

**REVISED: Use DRV8871 (45V, 3.6A max).** SOP-8 package, internal dead-time, built-in shoot-through protection. At 5mA load, power dissipation is negligible. The 45V rating provides ample margin for the ±15V compliance scenario.

#### Biphasic Pulse Timing Sequence:

```
Time ──────────────────────────────────────────────────────────────►

     ┌──────────────────── One pulse period: 40ms at 25Hz ──────────────────────┐
     │                                                                           │
     │  Phase+    Gap   Phase−    Gap                  REST                      │
     │  250µs    50µs   250µs    50µs              39,400µs                     │
     │ ◄──────► ◄────► ◄──────► ◄────► ◄────────────────────────────────────►   │
     │                                                                           │
IN1: │ ▔▔▔▔▔▔▔  ▁▁▁▁  ▁▁▁▁▁▁▁  ▁▁▁▁  ▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁   │
IN2: │ ▁▁▁▁▁▁▁  ▁▁▁▁  ▔▔▔▔▔▔▔  ▁▁▁▁  ▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁   │
     │                                                                           │
DAC: │ ▔▔▔▔▔▔▔  ▁▁▁▁  ▔▔▔▔▔▔▔  ▁▁▁▁  ▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁   │
     │  Vdac=N   0V    Vdac=N    0V     Vdac=0V                                │
     │                                                                           │
I_out│  +5mA    0mA   −5mA     0mA    0mA                                      │
(at  │ ▔▔▔▔▔▔▔  ▁▁▁▁                   ▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁   │
load)│                 ▁▁▁▁▁▁▁  ▁▁▁▁                                           │
     │                                                                           │
     └───────────────────────────────────────────────────────────────────────────┘

Sequence (executed in hardware timer ISR, ~3µs total):
  1. Write DAC code N via SPI, pulse LDAC (atomic update)
  2. Set IN1=1, IN2=0 (forward current)        → PHASE_POS begins
  3. After 250µs: Set IN1=0, IN2=0 (coast)     → GAP (DRV8871 internal dead-time handles transition)
  4. After 50µs: Write DAC code N via SPI (same value — ensures current is identical)
  5. Set IN1=0, IN2=1 (reverse current)         → PHASE_NEG begins
  6. After 250µs: Set IN1=0, IN2=0 (coast)     → GAP
  7. Write DAC code 0 via SPI                   → REST begins
  8. After 39,400µs: Go to step 1               → Next pulse

Dead-time: The DRV8871's internal dead-time (~1µs) handles the IN1/IN2 transition.
The firmware-inserted 50µs GAP provides additional margin AND allows DAC settling.
Shoot-through is physically prevented by the DRV8871's internal logic.
```

#### Charge Balance Verification:

```
Q_positive = I × t = 5mA × 250µs = 1.25 µC
Q_negative = I × t = 5mA × 250µs = 1.25 µC
Net charge per cycle = 0 µC (by design: identical DAC code, identical timing)

Worst-case imbalance from timer jitter:
  ESP32-S3 hardware timer resolution = 12.5ns (80MHz APB clock)
  Worst-case jitter: ±1 tick = ±12.5ns
  Charge imbalance per pulse: 5mA × 12.5ns = 62.5 pC
  Over 30-min session (25Hz × 1800s × 0.5 duty = 22,500 pulses):
    Total imbalance = 62.5 pC × 22,500 = 1.406 µC (random walk, not cumulative)
    RMS imbalance = 62.5 pC × √22,500 = 9.375 nC
    
  Both values are negligible compared to the DC-blocking cap's charge handling.
  The 10µF film cap limits DC offset to: 9.375 nC / 10µF = 0.94 µV. Safe.
```

### 2.5 Two Independent Channels — Architecture

**Decision: Fully independent analog stages, shared DAC.**

| Component | Shared? | Rationale |
|-----------|---------|-----------|
| MCP4922 DAC | Shared (dual-channel IC) | One IC, two independent 12-bit outputs. LDAC synchronizes updates. Cost and board space advantage. |
| SPI bus to DAC | Shared | Single SPI bus, channel A/B selected by MCP4922's internal channel address bit |
| Op-amp V-to-I | Independent | Each channel has its own op-amp feedback loop (OPA2277 is dual — one IC, two independent amps) |
| H-bridge | Independent | Each channel has its own DRV8871. Independent polarity control. |
| Overcurrent comparator | Independent | Each channel has its own LM393 + sense resistor + latch. A fault on one channel does NOT affect the other (though firmware may choose to stop both). |
| DC-blocking cap | Independent | Each channel has its own 10µF film cap + 1MΩ bleed. |
| Power supply (±15V) | Shared | Both channels draw from the same isolated ±15V rail. At 5mA per channel, total load ~10mA — well within B0515S-1W capacity (33mA per rail). |
| Isolation barrier | Shared | Single Si8621/Si8622 bank handles both channels' control signals. |

**Why not a switched single output:** Switching a single current source between two electrode pairs introduces transient spikes at each switch event, requires a complex analog multiplexer rated for compliance voltage, and cannot achieve the required 20ms binaural stagger without dead zones where neither channel is stimulating. Fully independent stages are more components but dramatically simpler firmware and inherently safer.

---

## 3. Hardware Architecture — Power Subsystem

### 3.1 Power Tree

```
┌──────────────────────────────────────────────────────────┐
│                    POWER ARCHITECTURE                     │
│                                                           │
│  USB-C 5V ──► TP4056 ──► LiPo 3.7V (2000mAh)           │
│                           │                               │
│                           ▼                               │
│              ┌─────────────────────────┐                 │
│              │ Boost: TPS61023         │                 │
│              │ 3.7V → 5.0V @ 500mA    │                 │
│              │ (95% efficient)         │                 │
│              └─────────┬───────────────┘                 │
│                        │                                  │
│            ┌───────────┼───────────────────┐             │
│            │           │                   │             │
│            ▼           ▼                   ▼             │
│     ┌────────────┐ ┌───────────┐  ┌──────────────────┐  │
│     │ ESP32-S3   │ │ Digital   │  │ B0515S-1WR3      │  │
│     │ 3.3V LDO  │ │ ICs (5V)  │  │ Isolated DC-DC   │  │
│     │ (on-board) │ │ Isolators │  │ 5V→±15V, 1W      │  │
│     │ ~150mA avg │ │ ~20mA     │  │ 1500VDC isol.    │  │
│     │ 350mA peak │ │           │  │ 33mA per rail    │  │
│     └────────────┘ └───────────┘  └────────┬─────────┘  │
│                                             │            │
│              ══════ ISOLATION BARRIER ═══════             │
│                                             │            │
│                                    ┌────────┴─────────┐  │
│                                    │ Patient-side     │  │
│                                    │ power            │  │
│                                    │                  │  │
│                                    │ +15V rail:       │  │
│                                    │  100µF + 100nF   │  │
│                                    │  + 1nF decoupling│  │
│                                    │                  │  │
│                                    │ −15V rail:       │  │
│                                    │  100µF + 100nF   │  │
│                                    │  + 1nF decoupling│  │
│                                    │                  │  │
│                                    │ GND_ISO          │  │
│                                    │ (independent of  │  │
│                                    │  GND_DIGITAL)    │  │
│                                    └──────────────────┘  │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

### 3.2 Compliance Voltage — Design Tradeoff

| Scenario | Current | Impedance | Voltage Needed | Our ±15V Provides | Verdict |
|----------|---------|-----------|----------------|-------------------|---------|
| **Nominal clinical** | 2 mA | 1 kΩ | 2V | 12.5V available | ✅ Easy |
| **Clinical max** | 3 mA | 2 kΩ | 6V | 12.5V available | ✅ Good margin |
| **High impedance** | 5 mA | 2.5 kΩ | 12.5V | 12.5V available | ⚠️ At limit |
| **Degrading contact** | 3 mA | 4 kΩ | 12V | 12.5V available | ⚠️ Near limit |
| **Poor contact** | 5 mA | 5 kΩ | 25V | 12.5V available | ❌ Out of compliance — detect and pause |
| **Detached** | 5 mA | 10+ kΩ | 50+V | 12.5V available | ❌ Open circuit — detect and stop |

**Design target: 12.5V compliance at ±15V rails.** This handles:
- 100% of well-prepared electrode scenarios (Z < 3kΩ with gel)
- 95% of clinical operating conditions (2mA into 1–5kΩ)
- The remaining 5% is detected by compliance monitoring and handled by firmware (impedance warning, auto-pause)

**Post-regulation decision: Skip LDOs for the ±15V rails.** Reasoning:
- LDOs (LM317/LM337) would drop ~2V each, reducing rails to ±13V and compliance to 10.5V
- The B0515S-1W has ~100mV ripple at 100kHz — significant for audio but acceptable for 25Hz stimulation pulses
- Instead: use aggressive LC filtering (10µH + 10µF ceramic + 100µF electrolytic) on each rail to attenuate switching ripple by >40dB
- If bench testing reveals ripple affecting impedance measurements, add post-regulation as a Rev 2 improvement

### 3.3 B0515S-1WR3 — Adequacy Analysis

| Parameter | Spec | Our Requirement | Margin |
|-----------|------|-----------------|--------|
| Input voltage | 4.5–5.5V | 5.0V (regulated boost) | ✅ |
| Output voltage | ±15V (nominal) | ±15V | ✅ |
| Output power | 1W total | 2 channels × 5mA × 15V + quiescent = ~200mW | ✅ 5× margin |
| Output current | ±33mA (each rail) | ~10mA stimulation + 5mA op-amp quiescent | ✅ 2× margin |
| Isolation | 1500 VDC | 1500 VDC required | ✅ Meets requirement |
| Efficiency | ~80% | — | Input draw: 250mW / 0.8 = 312mW = 62mA from 5V |
| Ripple | 50–100 mV | Need <10mV after filtering | Filter required |
| Package | SIP-6 | Through-hole, easy to solder | ✅ |
| Cost | ~$5–8 | Within BOM target | ✅ |

**Alternatives considered:**
- **Murata NMH0515SC** (~$15) — Higher isolation (3kVDC), lower ripple, better regulation. Overkill for battery-powered v1 but worth considering for Rev 2 if noise is problematic.
- **Traco TMA 0515D** (~$10) — 1W, ±15V, similar specs. Pin-compatible alternative.
- **Custom transformer + boost** — Maximum flexibility but massively increases design complexity. Not justified for v1.

**Confidence: HIGH** — B0515S-1WR3 is a standard choice in isolated instrument designs.

### 3.4 Power Budget — Battery Life Estimate

| Consumer | Current from 5V rail | Notes |
|----------|---------------------|-------|
| ESP32-S3 (BLE active) | ~80mA average | Spikes to 350mA during TX |
| B0515S-1WR3 (analog) | ~62mA | 250mW at 80% efficiency |
| Digital isolators (×3) | ~15mA | Si8621 quiescent × 3 ICs |
| Boost converter overhead | ~10mA | TPS61023 quiescent + losses |
| Status LED | ~5mA | Low-current RGB |
| **Total from 5V** | **~172mA avg** | ~200mA peak |

LiPo 2000mAh at 3.7V = 7.4Wh. System draws ~172mA × 5V = 860mW. 
**Battery life: 7.4Wh / 0.86W ≈ 8.6 hours.** 
At 30-min sessions: **~17 sessions per charge.** More than adequate.

---

## 4. Hardware Architecture — Impedance Sensing

### 4.1 Method: AC Injection During Inter-Pulse Rest

```
Measurement Window (during the 39.4ms REST period of each pulse cycle):
                                                      
  ...PHASE_NEG ─┤ GAP ├─ REST (39.4ms) ─────────────────────── PHASE_POS...
                        │                                       │
                        │  ← Impedance measurement window →     │
                        │    ~35ms available                     │
                        │                                       │
  Steps:                │                                       │
  1. Set DAC to test    │                                       │
     current code       │  ┌──────┐                             │
     (100µA = DAC       │  │ Vtest│ Measure this voltage        │
      code ~2)          │  │      │ with ADC                    │
  2. Set H-bridge to    │  └──────┘                             │
     FORWARD (IN1=1)    │                                       │
  3. Wait 1ms settle    │                                       │
  4. Read ADC (voltage  │                                       │
     across electrodes) │                                       │
  5. Z = Vadc / Itest   │                                       │
  6. Set DAC to 0,      │                                       │
     H-bridge to coast  │                                       │
```

### 4.2 Implementation Without Dedicated IC

**No AD5933 or similar impedance analyzer IC needed.** The existing hardware (DAC + current source + ADC) performs the measurement:

```
Hardware used:
  - MCP4922 DAC: Outputs a small voltage → V-to-I converter produces 100µA test current
  - DRV8871 H-bridge: Set to FORWARD to establish current path
  - Sense point: Voltage divider on op-amp output (before H-bridge) fed to ADC
  - ADC: MCP3201 (12-bit, SPI, on patient side) or ESP32-S3 internal ADC via isolation
  
Calculation:
  Itest = 100µA (known, set by DAC)
  Vmeasured = ADC reading × ADC_reference / 4096
  Z_electrode = Vmeasured / Itest

  At Z = 1kΩ: V = 100µA × 1kΩ = 100mV → easily measurable
  At Z = 10kΩ: V = 100µA × 10kΩ = 1V → well within ADC range
  At Z = 100Ω: V = 100µA × 100Ω = 10mV → near ADC noise floor, but this indicates a short/metallic contact (fault condition anyway)
```

**ADC Choice: MCP3201** (12-bit, SPI, single-channel)
- Sits on the patient side of the isolation barrier
- Shares the isolated SPI bus (different CS line)
- Samples the voltage across the electrode load
- 100 ksps max → one sample per measurement (averaged over 4–8 samples for noise rejection)
- Cost: ~$2

**Alternative: Use the ESP32-S3's internal ADC.** This requires routing the analog measurement signal across the isolation barrier (analog optocoupler or isolation amplifier like AMC1200). More complex and less accurate than placing a dedicated SPI ADC on the patient side. **Recommendation: MCP3201 on patient side.**

### 4.3 Measurement Integration with Stimulation Cycle

```
State machine integration:

  DUTY_ON period (30s):
    Each pulse cycle (40ms):
      [PHASE_POS(250µs)] → [GAP(50µs)] → [PHASE_NEG(250µs)] → [GAP(50µs)] → [REST(39.4ms)]
                                                                                    │
                                                                         Impedance measurement
                                                                         (1ms test pulse + 1ms ADC)
                                                                         
  DUTY_OFF period (30s):
    Full impedance characterization:
      - Sweep test current from 50µA to 500µA
      - Measure voltage at each point
      - Compute Z vs. I curve (nonlinear skin impedance)
      - Average and store

  Pre-session:
    - Full impedance check before first pulse
    - Must read 100Ω < Z < 10kΩ to proceed
    - Report Z value via BLE

  Thresholds:
    Z < 100Ω   → FAULT: metallic short or electrode error
    100–500Ω   → GOOD: excellent contact
    500–3kΩ    → GOOD: normal contact with gel
    3–5kΩ      → WARN: marginal contact, suggest reapplying gel
    5–10kΩ     → WARN: poor contact, reduce current to compliance limit
    >10kΩ      → FAULT: electrode detached, refuse to stimulate
```

---

## 5. Firmware Architecture

### 5.1 FreeRTOS Task Map — Dual-Core Assignment

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        ESP32-S3 DUAL-CORE LAYOUT                        │
│                                                                         │
│  ┌─────────────────────────────┐  ┌──────────────────────────────────┐ │
│  │        CORE 0               │  │          CORE 1                  │ │
│  │   (Protocol Core)           │  │     (Stimulation Core)           │ │
│  │                             │  │                                  │ │
│  │  ┌────────────────────┐     │  │  ┌──────────────────────────┐   │ │
│  │  │ NimBLE Host Task   │     │  │  │ HW Timer ISR             │   │ │
│  │  │ Priority: 21       │     │  │  │ (highest, non-preemptible)│   │ │
│  │  │ Stack: 8192 bytes  │     │  │  │ IRAM_ATTR                │   │ │
│  │  │                    │     │  │  │                          │   │ │
│  │  │ • GATT server      │     │  │  │ Executes every 250µs or  │   │ │
│  │  │   (phone control)  │     │  │  │ 50µs or 39.4ms depending │   │ │
│  │  │ • GATT client      │     │  │  │ on waveform state        │   │ │
│  │  │   (Polar H10 HRS)  │     │  │  │                          │   │ │
│  │  │ • Scan/connect     │     │  │  │ Does ONLY:               │   │ │
│  │  │   manager          │     │  │  │  • SPI write to MCP4922  │   │ │
│  │  └────────────────────┘     │  │  │  • GPIO toggle H-bridge  │   │ │
│  │                             │  │  │  • Pulse LDAC            │   │ │
│  │  ┌────────────────────┐     │  │  │  • Update state machine  │   │ │
│  │  │ Session Manager    │     │  │  │  • Kick heartbeat timer  │   │ │
│  │  │ Priority: 5        │     │  │  │                          │   │ │
│  │  │ Stack: 4096 bytes  │     │  │  │ Execution time: <5µs     │   │ │
│  │  │                    │     │  │  └──────────────────────────┘   │ │
│  │  │ • Start/stop logic │     │  │                                  │ │
│  │  │ • Duty cycle timer │     │  │  ┌──────────────────────────┐   │ │
│  │  │ • Ramp calculator  │     │  │  │ Impedance Task           │   │ │
│  │  │ • Param validation │     │  │  │ Priority: 10             │   │ │
│  │  │ • Safety state     │     │  │  │ Stack: 2048 bytes        │   │ │
│  │  │   machine          │     │  │  │                          │   │ │
│  │  └────────────────────┘     │  │  │ Runs during:             │   │ │
│  │                             │  │  │  • DUTY_OFF periods      │   │ │
│  │  ┌────────────────────┐     │  │  │  • Pre-session check     │   │ │
│  │  │ Flash Logger Task  │     │  │  │                          │   │ │
│  │  │ Priority: 3        │     │  │  │ Coordinates with ISR     │   │ │
│  │  │ Stack: 4096 bytes  │     │  │  │ via shared flag:         │   │ │
│  │  │                    │     │  │  │  ISR sets PULSE_DONE=1   │   │ │
│  │  │ • RR intervals     │     │  │  │  Task reads, measures,   │   │ │
│  │  │   → LittleFS       │     │  │  │  clears flag             │   │ │
│  │  │ • Session events   │     │  │  └──────────────────────────┘   │ │
│  │  │ • Impedance log    │     │  │                                  │ │
│  │  │ • Fault events     │     │  │  ┌──────────────────────────┐   │ │
│  │  └────────────────────┘     │  │  │ Heartbeat Generator      │   │ │
│  │                             │  │  │ (Hardware LEDC timer)     │   │ │
│  │  ┌────────────────────┐     │  │  │ 100Hz PWM on GPIO        │   │ │
│  │  │ HRV Processor Task │     │  │  │ Auto-stops on MCU crash  │   │ │
│  │  │ Priority: 2        │     │  │  │ (peripheral independent  │   │ │
│  │  │ Stack: 4096 bytes  │     │  │  │  of CPU, but stops on    │   │ │
│  │  │                    │     │  │  │  reset)                   │   │ │
│  │  │ • Post-session     │     │  │  └──────────────────────────┘   │ │
│  │  │   RMSSD, pNN50     │     │  │                                  │ │
│  │  │ • LF/HF ratio      │     │  └──────────────────────────────────┘ │
│  │  │ • Report via BLE   │     │                                       │
│  │  └────────────────────┘     │                                       │
│  │                             │                                       │
│  └─────────────────────────────┘                                       │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Waveform State Machine (Per Channel)

```
                          ┌─────────────────────────────────────────────┐
                          │         WAVEFORM STATE MACHINE              │
                          │         (runs inside HW Timer ISR)          │
                          └─────────────────────────────────────────────┘

                    Power-on / Reset
                          │
                          ▼
                   ┌──────────────┐
           ┌──────│     IDLE      │◄──────────── Manual restart only
           │      │ DAC=0, H=off │              (button or BLE cmd
           │      └──────┬───────┘               after fault clear)
           │             │
           │      BLE "START" cmd
           │      + impedance OK
           │      + !USB_VBUS
           │      + !fault_latch
           │             │
           │             ▼
           │      ┌──────────────┐
           │      │   RAMP_UP    │  Duration: 30s (configurable)
           │      │              │  DAC code increments each pulse
           │      │ target_code  │  by (target / (freq × ramp_time))
           │      │ /= N_pulses  │  
           │      └──────┬───────┘
           │             │
           │      ramp complete (current_code == target_code)
           │             │
           │             ▼
           │      ┌──────────────────────────────────────────┐
           │      │          STIMULATING (duty-cycled)        │
           │      │                                           │
           │      │  DUTY_ON (30s):                           │
           │      │    ┌─────────────────────────────────┐    │
           │      │    │  Per-pulse micro-states:         │    │
           │      │    │                                  │    │
           │      │    │  PHASE_POS ──► GAP_1 ──► PHASE_NEG  │
           │      │    │  (250µs)      (50µs)    (250µs)  │    │
           │      │    │       │                     │     │    │
           │      │    │       └─── identical DAC ───┘     │    │
           │      │    │              code (charge          │    │
           │      │    │              balanced)             │    │
           │      │    │                  │                 │    │
           │      │    │                  ▼                 │    │
           │      │    │              GAP_2 (50µs)         │    │
           │      │    │                  │                 │    │
           │      │    │                  ▼                 │    │
           │      │    │              REST (39.4ms)         │    │
           │      │    │              [impedance meas       │    │
           │      │    │               window here]        │    │
           │      │    │                  │                 │    │
           │      │    │                  ▼                 │    │
           │      │    │            Loop to PHASE_POS      │    │
           │      │    └─────────────────────────────────┘    │
           │      │                                           │
           │      │  DUTY_OFF (30s):                          │
           │      │    DAC=0, H-bridge coast                  │
           │      │    Full impedance characterization        │
           │      │    Update BLE status                      │
           │      │                                           │
           │      └────────────────────┬─────────────────────┘
           │                           │
           │                    session_timer expired
           │                           │
           │                           ▼
           │                    ┌──────────────┐
           │                    │  RAMP_DOWN   │  Duration: 30s (symmetric with ramp-up)
           │                    │              │  DAC code decrements each pulse
           │                    └──────┬───────┘
           │                           │
           │                    current_code == 0
           │                           │
           │                           ▼
           │                       → IDLE
           │
           │
           │   ANY STATE (except IDLE)
           │      │
           │      │  fault detected:
           │      │    • HW overcurrent latch
           │      │    • Impedance out of range
           │      │    • Watchdog timeout
           │      │    • BLE "STOP" command
           │      │    • E-Stop button
           │      │    • USB VBUS detected
           │      │
           │      ▼
           │  ┌──────────────┐
           └──│  SAFE_STOP   │
              │              │
              │ Immediate:   │
              │  DAC → 0     │
              │  H-bridge    │
              │    → coast   │
              │  nSLEEP→LOW  │
              │              │
              │ Then:        │
              │  Log fault   │
              │  BLE notify  │
              │  Require     │
              │  manual      │
              │  restart     │
              │  (clear      │
              │   fault      │
              │   first)     │
              └──────────────┘
```

### 5.3 Binaural Stagger — Single Timer, Dual Phase Offset

```
TIMING ARCHITECTURE: One hardware timer drives both channels.

  Hardware Timer 0 (General Purpose Timer, 80MHz APB clock):
    Prescaler: 80 → 1µs resolution
    Alarm period: 1µs (ticks at 1MHz for sub-pulse-width state transitions)
    
  The ISR maintains a microsecond counter that wraps every 40,000µs (40ms = 1 period at 25Hz).
  
  Channel A fires at counter = 0
  Channel B fires at counter = stagger_offset (default: 20,000µs = 20ms)
  
  Both channels' waveform state machines run inside the SAME ISR.
  The ISR checks if it's time for Channel A's next micro-state transition,
  then checks Channel B's. Worst-case ISR execution: ~5µs (2 channels × SPI write + GPIO).

Timeline at 25Hz with 20ms stagger:

  t=0ms     t=0.25ms  t=0.55ms   t=20ms    t=20.25ms  t=20.55ms  t=40ms
  │         │         │          │         │          │          │
  ChA+      ChA gap   ChA−       ChB+      ChB gap    ChB−       ChA+ (next)
  │         │         │          │         │          │          │
  ▼         ▼         ▼          ▼         ▼          ▼          ▼
  ┌───┐    ┌┐        ┌───┐      ┌───┐    ┌┐        ┌───┐      ┌───┐
  │A+ │    ││        │A− │      │B+ │    ││        │B− │      │A+ │
  │250│    │50       │250│      │250│    │50       │250│      │250│
  │µs │    │µs       │µs │      │µs │    │µs       │µs │      │µs │
  └───┘    └┘        └───┘      └───┘    └┘        └───┘      └───┘
  
  ◄── 600µs active ──►          ◄── 600µs active ──►
  
  Channel separation: 19.4ms between end of A and start of B.
  No overlap. No contention for the shared SPI bus (each SPI transaction takes ~1µs at 20MHz).

WHY SINGLE TIMER (not independent timers per channel):
  1. Independent timers on different clock sources can drift. Over a 30-min session at
     1ppm clock difference: 30×60×1e-6 = 1.8ms drift. The stagger would shift by 1.8ms,
     or ~5% of the 40ms period. Not catastrophic, but unnecessary.
  2. Single timer means both channels share the EXACT same time base. Stagger is 
     deterministic to ±1 timer tick (1µs). Zero drift.
  3. Simpler ISR: one entry point, one counter, two state machines.
  4. If user sets stagger to 0ms, firmware clamps to 10ms minimum (see PITFALLS.md S-07).
```

### 5.4 BLE Dual-Role — NimBLE on ESP32-S3

#### Architecture:

```
┌──────────────────────────────────────────────┐
│              NimBLE Stack (Core 0)             │
│                                               │
│  ┌──────────────┐    ┌─────────────────────┐ │
│  │ GAP          │    │ GAP                  │ │
│  │ Peripheral   │    │ Central              │ │
│  │ Role         │    │ Role                 │ │
│  │              │    │                      │ │
│  │ Advertises:  │    │ Scans for:           │ │
│  │  "taVNS-01"  │    │  Polar H10           │ │
│  │              │    │  (by name or UUID)    │ │
│  └──────┬───────┘    └──────────┬───────────┘ │
│         │                       │              │
│  ┌──────┴───────┐    ┌──────────┴───────────┐ │
│  │ GATT Server  │    │ GATT Client          │ │
│  │              │    │                      │ │
│  │ Service:     │    │ Connects to:         │ │
│  │  Stim Control│    │  Heart Rate Service  │ │
│  │  UUID: custom│    │  UUID: 0x180D       │ │
│  │              │    │                      │ │
│  │ Chars:       │    │ Reads:               │ │
│  │  current_mA  │    │  HR Measurement      │ │
│  │  frequency   │    │  (0x2A37)            │ │
│  │  pulse_width │    │  → RR intervals      │ │
│  │  stagger_ms  │    │  → stored to         │ │
│  │  duty_on/off │    │    LittleFS          │ │
│  │  command     │    │                      │ │
│  │  status(N)   │    │ Conn interval: 1000ms│ │
│  │  impedance(N)│    │ (H10 sends RR ~1/s)  │ │
│  │              │    │                      │ │
│  │ Conn interval│    │                      │ │
│  │  : 100ms     │    │                      │ │
│  └──────────────┘    └──────────────────────┘ │
│                                               │
│  Unified NimBLE scheduler handles both roles  │
│  with time-division multiplexing of radio.    │
└──────────────────────────────────────────────┘
```

#### Dual-Role Feasibility Assessment:

**Verdict: FEASIBLE, with constraints.** Confidence: MEDIUM.

| Aspect | Assessment |
|--------|------------|
| **NimBLE support** | NimBLE-Arduino 1.4.x explicitly supports simultaneous central + peripheral. The ESP32-S3's BLE 5.0 controller handles multiple connections natively. |
| **Radio scheduling** | The BLE radio is single-antenna, time-multiplexed. NimBLE's scheduler interleaves peripheral advertising/connection events with central scanning/connection events. At phone connection interval 100ms + H10 connection interval 1000ms, the duty cycle is well under 50%. |
| **Known issues** | Bluedroid has documented instability in dual-role on ESP32 (ESP-IDF GitHub issues #7022, #8394). NimBLE is the recommended path. |
| **Scanning + connected** | Continuous BLE scanning while maintaining a peripheral connection can cause event collisions. **Mitigation: Scan only during explicit "connect H10" phase, not continuously.** Sequence: phone connects → user initiates session → scan for H10 (10s window) → connect or timeout → begin stimulation. |
| **Stack size** | NimBLE host task needs ≥8192 bytes stack for dual-role. Default 4096 is insufficient. Arduino-ESP32 allows configuration via `NimBLEDevice::init()` settings. |
| **Max connections** | ESP32-S3 supports up to 9 simultaneous BLE connections (NimBLE default: 3). We need exactly 2 (phone + H10). Well within limits. |

**Risk flags:**
- 🟡 Connection parameter negotiation may fail with certain phone BLE stacks (iOS is aggressive about preferred intervals). Test with both iOS and Android.
- 🟡 Polar H10 may require specific MTU negotiation. Test actual H10 hardware early.
- 🟢 NimBLE dual-role is used in production by commercial ESP32 products (e.g., Meshtastic nodes act as central + peripheral simultaneously).

**Fallback plan:** If dual-role proves unstable, decouple HRV and stimulation:
- Option A: Log HRV via phone (phone connects to H10 separately, phone app forwards RR to device via BLE)
- Option B: Post-hoc HRV (user wears H10, records with Polar app, exports CSV, Python script aligns timestamps)
- Option C: Use second ESP32-S3 dedicated to H10 data collection (hardware solution, last resort)

### 5.5 Session Data — LittleFS Storage

```
Flash partition layout (16MB ESP32-S3-WROOM-1):

  ┌─────────────────────────┐
  │ Bootloader (64KB)       │
  ├─────────────────────────┤
  │ Partition Table (8KB)   │
  ├─────────────────────────┤
  │ NVS (24KB)              │  ← WiFi config, BLE bonds, user preferences
  ├─────────────────────────┤
  │ App0 (OTA slot 1, 2MB)  │  ← Active firmware
  ├─────────────────────────┤
  │ App1 (OTA slot 2, 2MB)  │  ← OTA update target
  ├─────────────────────────┤
  │ LittleFS (11.9MB)       │  ← Session data, HRV logs
  │                          │
  │  /sessions/              │
  │    2025-07-13_2230.json  │  ← ~2KB per session
  │    2025-07-14_2215.json  │
  │  /hrv/                   │
  │    2025-07-13_2230_rr.bin│  ← Raw RR intervals (~4KB/30min)
  │  /config/                │
  │    presets.json           │
  │    calibration.json       │
  │                          │
  │  At ~6KB/session: 11.9MB │
  │  stores ~2000 sessions   │
  │  (~5 years daily use)    │
  └─────────────────────────┘

Session file format (JSON):
{
  "version": 1,
  "device_id": "taVNS-A1B2C3",
  "fw_version": "1.0.0",
  "timestamp_utc": 1752451800,
  "params": {
    "freq_hz": 25,
    "pw_us": 250,
    "current_mA_L": 2.0,
    "current_mA_R": 2.1,
    "stagger_ms": 20,
    "duty_on_s": 30,
    "duty_off_s": 30,
    "duration_min": 30,
    "ramp_s": 30
  },
  "impedance": {
    "pre_L_ohm": 1200,
    "pre_R_ohm": 1450,
    "post_L_ohm": 1100,
    "post_R_ohm": 1380
  },
  "events": [
    {"t_ms": 15000, "type": "ramp_complete"},
    {"t_ms": 900000, "type": "duty_off", "z_L": 1180, "z_R": 1400},
    {"t_ms": 1800000, "type": "session_complete"}
  ],
  "faults": [],
  "hrv_file": "/hrv/2025-07-13_2230_rr.bin"
}
```

---

## 6. Safety Architecture — Three Independent Layers

### Layer 1: Hardware (Physics-Based, Firmware-Independent)

These protections work even if the ESP32-S3 is completely dead, crashed, or running malicious firmware.

| Protection | Implementation | Fail-Safe Mechanism |
|------------|---------------|---------------------|
| **Overcurrent cutoff** | LM393 comparator on 10Ω sense resistor. Trips at 60mV (6mA). Drives SR latch that pulls DRV8871 nSLEEP LOW. | Latch requires manual pushbutton reset. Cannot be reset by firmware. Overrides all software commands. |
| **DC blocking** | 10µF polypropylene film cap in series with each electrode output. 1MΩ bleed resistor in parallel. | Physically cannot pass DC. If every other protection fails and the H-bridge sticks in one state, the cap charges to the supply voltage and current drops to nanoamps (through bleed resistor leakage). |
| **Heartbeat gate** | ESP32 Core 1 generates 100Hz PWM on a GPIO. On the patient side: RC integrator (47kΩ + 1µF, τ = 47ms) charges to ~2.5V when heartbeat present. Comparator (LM393 #2) with 1.5V threshold drives a P-channel MOSFET that gates the +15V supply to the entire analog output stage. If heartbeat stops (MCU crash/reset), RC decays below threshold in ~50ms, MOSFET opens, all analog power dies. | Requires continuous CPU activity to keep analog alive. MCU reset = analog power OFF within 50ms. No firmware action needed — the ABSENCE of the heartbeat is the shutdown signal. |
| **USB VBUS lockout** | Voltage divider on USB VBUS (5V → 1.65V). Fed to comparator. If VBUS present AND stimulation enable is asserted, comparator forces DRV8871 nSLEEP LOW. | Prevents stimulation during USB charging regardless of firmware state. Even if firmware ignores VBUS, hardware disconnects output. |
| **E-Stop button** | Hardware momentary-contact pushbutton (normally open, pulled high). Press connects GPIO to GND. This GPIO feeds both (a) the heartbeat gate comparator (forcing it LOW = analog power off) AND (b) a firmware interrupt on Core 1 for clean shutdown logging. | Button mechanically kills analog power path in <1ms. Firmware interrupt handles graceful state transition and logging. Purely hardware path works even if firmware is unresponsive. |

### Layer 2: Firmware (Software Watchdog + State Guards)

| Protection | Implementation | Response |
|------------|---------------|----------|
| **RTC Watchdog (RWDT)** | ESP32-S3 RTC watchdog, independent of FreeRTOS. Set to 500ms timeout. Stimulation ISR feeds it every pulse cycle (40ms). If ISR stalls, RWDT triggers system reset. System reset kills heartbeat PWM → Layer 1 heartbeat gate kills analog power. | Reset → heartbeat stops → analog dies in 50ms. Boot sequence enforces DAC=0 before re-enabling anything. |
| **Charge balance counter** | ISR maintains `int32_t charge_balance_nC` per channel. Adds (I × t_pos) and subtracts (I × t_neg) each pulse. If |charge_balance| > 100nC (0.1µC), assert SAFE_STOP. | Catches systematic timing errors that the DC-blocking cap would eventually handle, but catches them faster (within a few pulses vs. cap charging over seconds). |
| **Compliance monitor** | During each REST period, firmware reads the ADC to check op-amp output voltage. If within 2V of either rail for 3 consecutive pulses, the load impedance exceeds compliance → SAFE_STOP with impedance fault. | Prevents sustained stimulation into a high-impedance load where the current source is no longer controlling. |
| **Session timeout** | FreeRTOS software timer on Core 0. Maximum 60 minutes (hard limit). When expired, triggers RAMP_DOWN → IDLE. If RAMP_DOWN doesn't complete within 60s (firmware stuck), the hardware timer ISR also has a pulse counter that hits zero and forces DAC=0. | Prevents infinite stimulation if session manager hangs. |
| **Parameter validation** | Every parameter written via BLE is bounds-checked before acceptance. Charge per phase ≤ 25µC. Current ≤ 5mA. Frequency × pulse_width × 2 < 1.0 (no pulse overlap). Stagger ≥ 10ms. | Invalid parameters are rejected with BLE error notification. Never stored, never applied. |

### Layer 3: State Machine (Behavioral Safety)

```
                SYSTEM-LEVEL SAFETY STATE MACHINE
                
    ┌───────────┐     ┌───────────┐     ┌──────────────┐
    │   BOOT    │────▸│  SELF_TEST │────▸│   READY      │
    │           │     │           │     │              │
    │ DAC=0     │     │ Check:    │     │ Awaiting     │
    │ H=off     │     │ • DAC zero│     │ BLE "START"  │
    │ nSLEEP=L  │     │ • ADC cal │     │              │
    │           │     │ • Comp OK │     │ All outputs  │
    │           │     │ • Z sense │     │ verified off │
    └───────────┘     │ • BLE init│     └──────┬───────┘
                      │           │            │
                      │ Any fail: │     BLE START +
                      │  → FAULT  │     impedance OK +
                      └───────────┘     !VBUS + !fault
                                               │
                                               ▼
                                        ┌──────────────┐
                                        │  STIMULATING  │
                                        │              │
                                        │ Normal       │
                                        │ operation    │
                                        └──────┬───────┘
                                               │
                                        Any fault condition
                                               │
                                               ▼
                                        ┌──────────────┐
    ┌──────────────────────────────────▸│  SAFE_STOP   │
    │  Hardware fault (Layer 1)         │              │
    │  OR firmware fault (Layer 2)      │ Immediate:   │
    │  OR user E-Stop                   │  DAC → 0     │
    │  OR BLE STOP                      │  H → coast   │
    │  OR USB VBUS detected             │  Log event   │
    │                                   │  BLE notify  │
    │                                   │              │
    │                                   │ Transition:  │
    │                                   │  → FAULT     │
    │                                   └──────┬───────┘
    │                                          │
    │                                          ▼
    │                                   ┌──────────────┐
    │                                   │   FAULT      │
    │                                   │              │
    │                                   │ Latched.     │
    │                                   │ Requires:    │
    │                                   │  1. HW fault │
    │                                   │     cleared  │
    │                                   │     (button) │
    │                                   │  2. BLE      │
    │                                   │     "CLEAR"  │
    │                                   │     command  │
    │                                   │  3. Self-test│
    │                                   │     passes   │
    │                                   │              │
    │                                   │ → READY      │
    │                                   └──────────────┘
    │
    │  INVARIANT: SAFE_STOP is reachable from ANY state
    │  in ≤1 ISR cycle (40ms) via firmware,
    │  or ≤50ms via hardware heartbeat gate.
    │  
    │  INVARIANT: FAULT requires BOTH hardware AND
    │  software acknowledgment to clear. Neither alone
    │  is sufficient.
```

---

## 7. Component Boundaries and Data Flow

### 7.1 Module Decomposition

```
firmware/
├── src/
│   ├── main.cpp                    ← Setup, task creation, core pinning
│   │
│   ├── stim/                       ← STIMULATION ENGINE (Core 1)
│   │   ├── waveform_sm.h/.cpp      ← Per-channel waveform state machine
│   │   ├── stim_isr.h/.cpp         ← Hardware timer ISR, SPI DAC writes
│   │   ├── stim_params.h           ← Parameter struct + validation
│   │   └── charge_counter.h/.cpp   ← Running charge balance integrator
│   │
│   ├── safety/                     ← SAFETY SUBSYSTEM (Core 1 + hardware)
│   │   ├── overcurrent.h/.cpp      ← Read HW fault latch GPIO, log, assert SAFE_STOP
│   │   ├── compliance_monitor.h/.cpp ← ADC-based op-amp output voltage check
│   │   ├── impedance.h/.cpp        ← Impedance measurement task
│   │   ├── heartbeat.h/.cpp        ← LEDC PWM configuration for heartbeat
│   │   └── safety_sm.h/.cpp        ← System-level safety state machine
│   │
│   ├── ble/                        ← BLE SUBSYSTEM (Core 0)
│   │   ├── ble_manager.h/.cpp      ← NimBLE init, dual-role orchestration
│   │   ├── gatt_server.h/.cpp      ← GATT service for phone control
│   │   ├── gatt_client_h10.h/.cpp  ← GATT client for Polar H10 HRS
│   │   └── ble_params.h            ← GATT UUIDs, characteristic definitions
│   │
│   ├── session/                    ← SESSION MANAGEMENT (Core 0)
│   │   ├── session_manager.h/.cpp  ← Start/stop logic, duty cycle timer, ramp calc
│   │   ├── session_logger.h/.cpp   ← LittleFS writes (session JSON, RR binary)
│   │   └── hrv_processor.h/.cpp    ← RMSSD, pNN50, SDNN computation
│   │
│   ├── hal/                        ← HARDWARE ABSTRACTION
│   │   ├── dac_mcp4922.h/.cpp      ← SPI DAC driver (write, LDAC, zero)
│   │   ├── adc_mcp3201.h/.cpp      ← SPI ADC driver (impedance sense)
│   │   ├── hbridge.h/.cpp          ← DRV8871 control (forward, reverse, coast)
│   │   └── gpio_map.h              ← All pin assignments in one place
│   │
│   └── config/                     ← CONFIGURATION
│       ├── golden_params.h         ← Default stimulation parameters (single source of truth)
│       ├── safety_limits.h         ← Hard limits (compile-time, non-overridable)
│       └── build_config.h          ← Board revision, feature flags
│
├── test/                           ← Unit tests (PlatformIO native)
│   ├── test_waveform_sm/           ← State machine transition tests
│   ├── test_charge_counter/        ← Charge balance edge cases
│   ├── test_stim_params/           ← Parameter validation bounds
│   └── test_ramp/                  ← Ramp calculation (integer math edge cases)
│
└── platformio.ini                  ← Pinned platform, library, framework versions
```

### 7.2 Inter-Module Data Flow

```
                         ┌─────────────────────────┐
                         │      BLE (Core 0)        │
                         │                          │
    Phone ◄──BLE──►      │  gatt_server             │
                         │    │ write: params        │
                         │    │ notify: status,      │
                         │    │         impedance    │
                         │    ▼                      │
    Polar H10 ◄──BLE──► │  gatt_client_h10          │
                         │    │ notify: RR intervals │
                         │    ▼                      │
                         │  ble_manager              │
                         │    │ orchestrates scan/   │
                         │    │ connect lifecycle    │
                         └────┼──────────────────────┘
                              │
              ┌───────────────┼───────────────────────┐
              │               │                       │
              ▼               ▼                       ▼
    ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐
    │ session_mgr  │  │ session_log  │  │ hrv_processor    │
    │              │  │              │  │                  │
    │ Receives:    │  │ Writes:      │  │ Receives:        │
    │  BLE params  │  │  session.json│  │  RR intervals    │
    │  BLE cmds    │  │  rr.bin      │  │                  │
    │              │  │  to LittleFS │  │ Computes:        │
    │ Sends:       │  │              │  │  RMSSD, pNN50    │
    │  target_code │  │ Receives:    │  │  SDNN, LF/HF    │
    │  state cmds  │  │  events from │  │                  │
    │  to stim ISR │  │  all modules │  │ Sends:           │
    │  via shared  │  │              │  │  results via BLE │
    │  atomic vars │  └──────────────┘  └──────────────────┘
    └──────┬───────┘
           │
           │  Shared state (volatile atomic):
           │    target_dac_code_A  (uint16_t)
           │    target_dac_code_B  (uint16_t)
           │    stim_command       (enum: START, STOP, RAMP)
           │    stim_state         (enum: read by session mgr)
           │
    ═══════╪═══════════════════════════════════════════════
           │           CORE BOUNDARY (Core 0 ←→ Core 1)
    ═══════╪═══════════════════════════════════════════════
           │
           ▼
    ┌──────────────┐
    │ stim_isr     │  (Core 1, hardware timer, highest priority)
    │              │
    │ Reads:       │
    │  target_code │  (atomic, set by session_mgr on Core 0)
    │  stim_command│
    │              │
    │ Writes:      │
    │  DAC via SPI │  (hal/dac_mcp4922)
    │  H-bridge    │  (hal/hbridge — GPIO direct)
    │  LDAC pulse  │
    │              │
    │ Updates:     │
    │  charge_counter (per-channel running balance)
    │  stim_state   (for session_mgr to read)
    │  pulse_done_flag (for impedance task)
    │              │
    │ Feeds:       │
    │  RTC watchdog (RWDT) — pet every pulse cycle
    │  Heartbeat PWM — runs as independent LEDC timer
    └──────┬───────┘
           │
           │  pulse_done_flag (set by ISR during REST)
           │
           ▼
    ┌──────────────┐
    │ impedance    │  (Core 1, FreeRTOS task, priority 10)
    │              │
    │ Waits for:   │
    │  pulse_done  │
    │  OR duty_off │
    │              │
    │ Performs:     │
    │  DAC write   │  (test current)
    │  H-bridge    │  (forward)
    │  ADC read    │  (MCP3201)
    │  Calculate Z │
    │              │
    │ Reports:     │
    │  impedance_L │  → BLE notify
    │  impedance_R │  → session log
    │  fault if    │  → safety_sm
    │  out of range│
    └──────────────┘
```

---

## 8. PCB Architecture — Physical Layout Zones

```
┌──────────────────────────────────────────────────────────────────────┐
│                        PCB TOP VIEW (4-layer)                        │
│                                                                      │
│  ZONE 1: DIGITAL                    │  ZONE 2: PATIENT ANALOG       │
│  (GND_DIGITAL)                      │  (GND_ISO)                    │
│                                     │                                │
│  ┌───────────┐  ┌─────────────┐    ║  ┌──────────────┐              │
│  │ USB-C     │  │ ESP32-S3    │    ║  │ MCP4922 DAC  │              │
│  │ connector │  │ WROOM-1     │    ║  │              │              │
│  └───────────┘  │             │    ║  └──────────────┘              │
│                 │             │    ║                                 │
│  ┌───────────┐  │  BLE        │    ║  ┌──────────────┐              │
│  │ TP4056    │  │  antenna    │    ║  │ OPA2277      │              │
│  │ charger   │  │  keepout    │    ║  │ (V-to-I ×2)  │              │
│  └───────────┘  │             │    ║  └──────────────┘              │
│                 └─────────────┘    ║                                 │
│  ┌───────────┐                     ║  ┌──────────────┐  ┌─────────┐ │
│  │ Boost     │  ┌──────────────┐   ║  │ DRV8871 ×2   │  │ LM393   │ │
│  │ TPS61023  │  │ Si8621 ×2-3  │   ║  │ (H-bridge)   │  │ ×2      │ │
│  └───────────┘  │ Isolators    │   ║  └──────────────┘  │(overI)  │ │
│                 │ (straddle    │   ║                     └─────────┘ │
│  ┌───────────┐  │  barrier)    │   ║  ┌──────────────┐              │
│  │ LiPo      │  └──────────────┘   ║  │ MCP3201      │              │
│  │ connector │                     ║  │ (impedance   │              │
│  └───────────┘  ┌──────────────┐   ║  │  ADC)        │              │
│                 │ B0515S-1WR3  │   ║  └──────────────┘              │
│  [E-STOP BTN]   │ (straddles   │   ║                                │
│                 │  barrier)    │   ║  ┌──────────────┐  ┌─────────┐ │
│  [STATUS LED]   └──────────────┘   ║  │ DC-block caps│  │Electrode│ │
│                                    ║  │ 10µF film ×2 │  │connectors││
│                                    ║  └──────────────┘  └─────────┘ │
│                                    ║                                 │
│  ◄──── Digital ground plane ──────►║◄── Isolated analog ground ────►│
│                                    ║                                 │
│  Ground connection: SINGLE POINT   ║  (Only through B0515S and      │
│  through B0515S-1WR3 parasitic     ║   Si8621 parasitic capacitance)│
│  capacitance only — NO copper      ║                                 │
│  connection between GND planes     ║                                 │
│                                    ║                                 │
└──────────────────────────────────────────────────────────────────────┘

Layer stack (4-layer, 1.6mm):
  Layer 1 (Top):     Components + signal routing
  Layer 2 (Inner 1): Ground plane (split: GND_DIGITAL | GND_ISO)
  Layer 3 (Inner 2): Power planes (+5V, +15V, -15V)
  Layer 4 (Bottom):  Components + signal routing

CRITICAL ROUTING RULES:
  1. NO traces cross the ground plane split (creates antenna)
  2. SPI traces from ESP32 to isolators route entirely over GND_DIGITAL
  3. SPI traces from isolators to DAC route entirely over GND_ISO
  4. Isolator ICs and B0515S-1WR3 straddle the split — their pins on each side
     connect to the respective ground plane
  5. BLE antenna keepout: no ground plane or traces within 10mm of antenna
  6. Analog components (DAC, op-amp, sense resistors) clustered within 15mm
     on the isolated side — short traces for sensitive signals
```

---

## 9. Build Order — Dependency Analysis and Critical Path

### 9.1 Dependency Graph

```
                    BUILD ORDER DEPENDENCY GRAPH
                    
Phase 1: TENS Hack (existing hardware, add safety + control)
═══════════════════════════════════════════════════════════
  Not in scope for custom PCB architecture. Parallel validation path.
  Uses modified commercial TENS for clinical parameter validation.


Phase 2-3: Custom Hardware + Firmware (interleaved)
═══════════════════════════════════════════════════════════

LEVEL 0 — FOUNDATION (no dependencies, start immediately)
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  [A] ESP32-S3 Dev Kit          [B] Schematic Design         │
│      + FreeRTOS Bringup            (KiCad)                  │
│      + Hardware timer ISR          Full circuit before       │
│      + SPI driver                  ordering any boards       │
│      + NimBLE basic init                                    │
│      + LittleFS init              [C] LTspice Simulation    │
│                                       V-to-I + load model   │
│  Deliverable: Timer ISR fires         H-bridge timing       │
│  at 25Hz, toggles GPIO,              AC/DC analysis         │
│  writes SPI to breadboard DAC        Stability check        │
│                                                             │
└──────────────────────┬──────────────────┬───────────────────┘
                       │                  │
                       ▼                  ▼
LEVEL 1 — SINGLE-CHANNEL ANALOG (requires Level 0)
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  [D] Breadboard Prototype:                                  │
│      MCP4922 + OPA2277 + DRV8871 + 1kΩ dummy load          │
│      (NO isolation yet — digital ground = analog ground)    │
│                                                             │
│      Validate:                                              │
│        • DAC output linearity (0–0.5V in 100 steps)         │
│        • V-to-I accuracy (0.1–5mA into 1kΩ, measure with   │
│          DMM across sense resistor)                         │
│        • H-bridge biphasic waveform on oscilloscope         │
│        • Charge balance (AC-coupled scope, 10s timebase)    │
│        • Dead-time visible on scope (zoom to transitions)   │
│        • DAC glitch energy with/without LDAC pin            │
│                                                             │
│  Deliverable: Clean biphasic waveform, 0.1mA accuracy      │
│  on scope across 1kΩ load                                   │
│                                                             │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
LEVEL 2 — SAFETY + IMPEDANCE (requires Level 1)
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  [E] Add Overcurrent Protection:                            │
│      LM393 + 10Ω sense resistor + SR latch + LED indicator  │
│      Test: set DAC to 6mA equivalent, verify latch trips    │
│      Test: verify latch holds after MCU reset               │
│      Test: verify manual reset button clears latch          │
│                                                             │
│  [F] Add Impedance Sensing:                                 │
│      MCP3201 ADC + test current injection                   │
│      Test: measure known resistors (100Ω, 1kΩ, 5kΩ, 10kΩ)  │
│      Test: measure during REST period of active stimulation │
│      Verify no cross-talk with stimulation waveform         │
│                                                             │
│  [G] Firmware Waveform State Machine:                       │
│      Full state machine (IDLE → RAMP → STIM → RAMP → IDLE) │
│      Charge balance counter                                 │
│      Compliance voltage monitoring                          │
│      Safety state machine (BOOT → SELFTEST → READY → ...)  │
│                                                             │
│  Deliverable: Single-channel prototype that self-tests,     │
│  measures impedance, stimulates, and self-protects          │
│                                                             │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
LEVEL 3 — ISOLATION BOUNDARY (requires Level 2 — highest risk)
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  [H] Add Isolation:                                         │
│      B0515S-1WR3 + Si8621 digital isolators                 │
│      Split breadboard power rails into digital/isolated     │
│                                                             │
│      ★ CRITICAL VALIDATION POINT ★                          │
│      Test: SPI through isolator — verify DAC still works    │
│        at 20MHz (may need to reduce to 10MHz)               │
│      Test: Measure noise on ±15V rails with scope           │
│        → if >20mV ripple, add LC filter                     │
│      Test: Megohmmeter between GND_DIG and GND_ISO          │
│        → must read >10MΩ                                    │
│      Test: Leakage current with 500V applied                │
│        → must be <1µA                                       │
│      Test: Stimulation accuracy unchanged after isolation   │
│        → compare waveform quality pre/post isolation        │
│                                                             │
│  [I] Heartbeat Gate Circuit:                                │
│      100Hz PWM → RC integrator → comparator → MOSFET gate  │
│      Test: disable heartbeat PWM, verify analog power dies  │
│      Test: reset ESP32, measure time to analog shutoff      │
│        → must be <100ms                                     │
│                                                             │
│  Deliverable: Isolated single-channel with all safety       │
│  layers functional. This is the MINIMUM VIABLE SAFE UNIT.   │
│                                                             │
└──────────────────────┬──────────────────────────────────────┘
                       │
           ┌───────────┴───────────┐
           ▼                       ▼
LEVEL 4a — DUAL CHANNEL          LEVEL 4b — BLE (parallel)
┌──────────────────────┐         ┌──────────────────────────┐
│                      │         │                          │
│  [J] Second Channel: │         │  [K] BLE Peripheral:     │
│  Duplicate analog    │         │  GATT server for phone   │
│  stage for Channel B │         │  Parameter write/read    │
│  Independent V-to-I, │         │  Status notifications    │
│  H-bridge, overcurr  │         │  Command (start/stop)    │
│                      │         │                          │
│  Add binaural stagger│         │  [L] BLE Central:        │
│  to timer ISR        │         │  Scan + connect Polar H10│
│                      │         │  Subscribe to HRS notify │
│  Test: both channels │         │  Parse RR intervals      │
│  on scope, verify    │         │  Store to LittleFS       │
│  20ms offset         │         │                          │
│  Test: independent   │         │  [M] Dual-Role Test:     │
│  current per channel │         │  Phone connected while   │
│                      │         │  H10 streaming RR data   │
│                      │         │  No disconnections over  │
│                      │         │  30-min session           │
└──────────┬───────────┘         └────────────┬─────────────┘
           │                                  │
           └──────────┬───────────────────────┘
                      │
                      ▼
LEVEL 5 — INTEGRATION (requires Level 4a + 4b)
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  [N] Full System Integration (breadboard):                  │
│      Dual-channel stim + BLE control + H10 HRV + logging   │
│      30-minute soak test on dummy load                      │
│      Verify: no BLE disconnections during stimulation       │
│      Verify: RR intervals logged continuously               │
│      Verify: impedance readings stable                      │
│      Verify: charge balance counter stays near zero         │
│                                                             │
│  [O] HRV Post-Processing:                                   │
│      Python script reads session + RR data from device      │
│      Computes RMSSD, pNN50, SDNN, LF/HF                    │
│      Generates session report                               │
│                                                             │
│  Deliverable: Complete working prototype on breadboard      │
│                                                             │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
LEVEL 6 — PCB (requires Level 5 integration success)
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  [P] KiCad Schematic Finalization                           │
│      Transfer validated breadboard circuit to KiCad         │
│      DRC, ERC clean                                         │
│                                                             │
│  [Q] PCB Layout                                             │
│      4-layer, isolation barrier split                       │
│      Analog/digital zone enforcement                        │
│      BLE antenna clearance                                  │
│      Thermal analysis for DRV8871 + OPA2277                 │
│                                                             │
│  [R] Fabrication + Assembly                                 │
│      JLCPCB order (5 boards, ~$20)                          │
│      SMD assembly (hand-solder or JLCPCB SMT service)       │
│                                                             │
│  [S] PCB Validation                                         │
│      Repeat all Level 1-5 tests on PCB                      │
│      Additional: isolation megohm test, EMC self-test       │
│                                                             │
│  Deliverable: Working PCB with all features validated       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 9.2 Critical Path Analysis

```
CRITICAL PATH (longest sequential dependency chain):

  [A] ESP32 bringup (1 week)
    → [D] Single-channel breadboard (1 week)
      → [E+F+G] Safety + impedance + firmware SM (2 weeks)
        → [H+I] Isolation boundary (1–2 weeks) ★ HIGHEST RISK
          → [J] Dual channel (1 week)
            → [N] Integration (1 week)
              → [P+Q+R+S] PCB design + fab + validate (3–4 weeks)

  TOTAL CRITICAL PATH: ~10–12 weeks

PARALLEL PATHS:
  • [B] Schematic design starts at Level 0, iterates through all levels
  • [C] LTspice simulation at Level 0, informs [D]
  • [K+L+M] BLE development at Level 4b, parallel to [J] dual channel
  • [O] HRV Python script anytime after [L] is working
  • Phase 1 TENS hack is entirely parallel — different hardware path
```

### 9.3 Highest-Risk Validation Points

| Risk | Level | What Could Go Wrong | Early Mitigation |
|------|-------|--------------------|--------------------|
| **Isolation SPI integrity** | 3 [H] | Digital isolator adds propagation delay (~50ns per channel). At 20MHz SPI, bit period is 50ns. Race condition between SCK and MOSI edges corrupts data. | Test isolator SPI first at 5MHz, increase to 10MHz, verify with logic analyzer. MCP4922 works up to 20MHz, but through-isolator may need ≤10MHz. Budget: if SPI is limited to 5MHz, a full DAC update takes ~3.2µs instead of ~0.8µs. Still fits within 250µs pulse window with margin. |
| **Isolated DC-DC noise** | 3 [H] | B0515S-1WR3 switching noise (100kHz, 100mV ripple) couples into analog signal chain, corrupts impedance measurements, adds noise to stimulation waveform. | Test noise on breadboard BEFORE committing to PCB layout. If ripple >20mV after LC filter, add linear post-regulation (LM317/LM337). Budget: reduces compliance by ~2V (10.5V instead of 12.5V). Acceptable tradeoff. |
| **BLE dual-role stability** | 4b [M] | NimBLE dual-role drops connections during long sessions. Phone and H10 connection events collide. | Test on dev kit with actual Polar H10 EARLY (Level 4b). Run 30-min sessions, log disconnect events. If unstable, implement fallback: phone forwards H10 data. |
| **Compliance voltage adequacy** | 1 [D] | Real electrode impedance higher than expected. Op-amp saturates at lower impedance than calculated. | Test with variable resistor load (100Ω to 10kΩ) and characterize actual compliance point. If <2kΩ, consider higher voltage isolated DC-DC (B0524S for ±24V, at the cost of halving current capacity per rail). |
| **H-bridge driver availability** | 1 [D] | DRV8871 goes EOL or is out of stock. | Verify stock on Mouser/Digi-Key before design. Identify drop-in alternatives: DRV8876 (pin-compatible), or fall back to discrete MOSFET H-bridge with MCPWM dead-time. |

### 9.4 What Can Be Developed in Parallel

| Stream | Activities | Team/Effort |
|--------|-----------|-------------|
| **Analog Hardware** | LTspice sim → Breadboard → Validation → PCB layout | Requires oscilloscope, DMM, bench supply |
| **Firmware Core** | Timer ISR → State machine → Safety SM → Integration | Requires ESP32-S3 devkit only |
| **BLE** | NimBLE peripheral → Central → Dual-role soak test | Requires devkit + phone + Polar H10 |
| **Electrode Design** | Cymba conchae clips, Ag/AgCl contacts, gel protocol | Independent of electronics |
| **Python HRV** | RR interval parsing, RMSSD/pNN50 computation, plotting | Independent of hardware |
| **Documentation** | Clinical evidence summary, safety document, assembly guide | Independent of hardware |

---

## 10. Key Design Decisions Summary

| Decision | Choice | Rationale | Confidence |
|----------|--------|-----------|------------|
| **Current source topology** | V-to-I with Rsense, NOT Howland | H-bridge provides floating load. Howland adds oscillation risk and complexity with no benefit when H-bridge exists. | HIGH |
| **Rsense value** | 100Ω | Best SNR for overcurrent comparator (50mV at 5mA). Costs 0.5V compliance. Good tradeoff. | HIGH |
| **Op-amp** | OPA2277 (dual) | Low offset (±20µV), ±18V supply, stable, affordable (~$3), proven in instrumentation. | HIGH |
| **H-bridge** | DRV8871 (TI) | 45V rating covers compliance voltage. Built-in shoot-through protection + dead-time. 6-pin package. | HIGH |
| **DAC placement** | Patient side (after isolation) | Only digital signals cross barrier. No analog isolation needed. Cleanest approach. | HIGH |
| **Isolation** | Si8621BB + B0515S-1WR3 | Standard industry approach. 1500V rated. Proven reliability. | HIGH |
| **Post-regulation** | Skip for v1 (LC filter only) | Saves 2V compliance headroom. Add LDOs in Rev 2 if noise is problematic. | MEDIUM |
| **BLE stack** | NimBLE (not Bluedroid) | Better dual-role support, smaller footprint, shorter critical sections. | HIGH |
| **Core assignment** | Core 0=BLE, Core 1=Stimulation | Guarantees BLE cannot preempt stimulation ISR. Industry standard ESP32 dual-core pattern. | HIGH |
| **Timer architecture** | Single HW timer, dual channel | Eliminates inter-channel clock drift. Deterministic stagger. Simpler ISR. | HIGH |
| **Impedance ADC** | MCP3201 on patient side | Avoids routing analog across isolation barrier. Shares SPI bus with DAC. Low cost. | HIGH |
| **PCB layers** | 4-layer | Proper ground plane management essential for mixed-signal + isolation boundary. ~$2 premium over 2-layer. Non-negotiable for safety. | HIGH |
| **Heartbeat watchdog** | Hardware RC + comparator, not software GPIO | Independent of firmware state. Analog power dies within 50ms of MCU crash. Cannot be disabled by software bug. | HIGH |

---

## 11. Anti-Patterns to Avoid

### AP-1: Software-Only Safety
**What:** Relying on `if (current > MAX) { stop(); }` as the only overcurrent protection.
**Why bad:** Firmware crashes, ISR stalls, race conditions, and cosmic rays can all defeat software checks. A stuck GPIO leaves current flowing.
**Instead:** Hardware comparator + latch that physically disconnects output. Software adds logging and graceful recovery ON TOP of hardware protection.

### AP-2: MOSFET H-Bridge Without Dead-Time Hardware
**What:** Using 4 discrete MOSFETs controlled by individual `digitalWrite()` calls.
**Why bad:** Between `digitalWrite(Q1, LOW)` and `digitalWrite(Q2, HIGH)`, the CPU might service an interrupt. Both MOSFETs conduct simultaneously. Current spike into patient. Potentially destructive.
**Instead:** Use DRV8871 with built-in dead-time, or use ESP32-S3 MCPWM peripheral with hardware dead-time if using discrete MOSFETs.

### AP-3: `delayMicroseconds()` for Pulse Timing
**What:** `setDAC(value); delayMicroseconds(250); setDAC(0);` in the main loop.
**Why bad:** Not preemption-safe. FreeRTOS can preempt between the DAC set and the delay, stretching or shortening the pulse. BLE stack can hold CPU for milliseconds.
**Instead:** Hardware timer ISR with compare-match interrupts. Timer peripheral manages timing. ISR only executes state transitions.

### AP-4: Shared Ground Between Digital and Patient
**What:** Single ground plane connecting ESP32's GND to electrode return.
**Why bad:** USB charger leakage (50–500µA) flows through patient. BLE TX spikes modulate the analog reference. Any MCU fault can impress voltage on patient ground.
**Instead:** Complete galvanic isolation with split ground planes connected only through the isolation barrier's parasitic capacitance.

### AP-5: Howland Pump for This Application
**What:** Using an improved Howland current pump because "it's the standard neurostim topology."
**Why bad:** It IS standard — for implantable stimulators with ±60V compliance and dedicated stimulus ASICs. For a battery-powered, low-compliance portable device with an H-bridge, it adds oscillation risk (electrode capacitance), resistor matching requirements (0.01% for high output impedance), and unnecessary complexity.
**Instead:** Simple V-to-I with sense resistor feedback. The H-bridge provides the floating/biphasic capability that Howland normally provides.

### AP-6: Continuous BLE Scanning
**What:** Running `NimBLEDevice::getScan()->start(0)` (continuous scan) while maintaining peripheral connections.
**Why bad:** Continuous scanning saturates the BLE radio scheduler. Connection events get delayed or missed. Phone connection drops. H10 connection drops. Stimulation ISR sees no BLE events but the radio is so busy it brown-outs other tasks.
**Instead:** Scan only when explicitly requested (user presses "Connect H10" in BLE app). Scan for 10 seconds. Connect or timeout. Stop scanning immediately upon connection. Never scan during active stimulation unless in DUTY_OFF period.

---

## 12. Scalability Considerations

| Concern | Single User (Now) | Community (100 builders) | Research Use (10 labs) |
|---------|-------------------|--------------------------|----------------------|
| **Parameter reproducibility** | golden_params.h | Pin all versions (platformio.ini) | Per-experiment parameter logging + version stamp |
| **Firmware updates** | USB serial flash | BLE OTA with signature verification | Tagged releases with changelog |
| **Data format** | JSON session files | Standardized schema (versioned) | CSV export for statistical software |
| **PCB variations** | Single rev | BOM alternates documented for component shortages | Rev tracking on silkscreen |
| **Safety testing** | Self-test on boot | Documented test protocol for builders | Formal verification checklist |
| **Electrode variability** | User-fabricated | Recommended BOM with exact Ag/AgCl part numbers | Electrode impedance included in session data for normalization |

---

## Sources and Confidence

| Topic | Primary Source | Confidence | Notes |
|-------|---------------|------------|-------|
| V-to-I converter topology | Training data: analog IC design textbooks (Horowitz & Hill, Franco), TI application notes (SLOA097) | HIGH | Well-established circuit topology, decades of use |
| DRV8871 specs | Training data: TI DRV8871 datasheet (SLVSCY4) | HIGH | Popular part, widely documented |
| MCP4922 specs | Training data: Microchip MCP4922 datasheet (DS21897) | HIGH | Mature product, well-characterized |
| ESP32-S3 dual-core | Training data: ESP-IDF documentation, Arduino-ESP32 docs | HIGH | Extensively documented by Espressif |
| NimBLE dual-role | Training data: NimBLE-Arduino docs, ESP-IDF BLE examples, Meshtastic project | MEDIUM | Functional in production products, but edge cases exist with specific phone BLE stacks |
| B0515S-1WR3 isolation | Training data: MORNSUN/MEAN WELL datasheets | HIGH | Standard isolated DC-DC module |
| Si8621 digital isolator | Training data: Silicon Labs Si862x datasheet | HIGH | Industry-standard digital isolator |
| IEC 60601-1 patient isolation | Training data: IEC 60601-1:2005+AMD1:2012, applied devices classification | HIGH | Regulatory standard, well-defined requirements |
| Howland pump instability | Training data: A. Devices AN-1515, TI SLOA097, published neurostim literature | HIGH | Well-documented failure mode in neurostim community |
| Electrode impedance ranges | Training data: Peuker & Filler (2002), clinical taVNS studies (2018–2024) | MEDIUM | Varies significantly with electrode type, skin prep, and individual anatomy |

---

*Last updated: 2025-07-13 — Architecture research for OpenBinaural-taVNS custom PCB*
