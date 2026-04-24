# taVNS Design Review — Consolidated Action Items (2026-04-24)

> Output of 4 independent warm-onboarded agent reviews (2× GPT-5.4 "Kai", 2× Opus "Mara").
> All findings cross-referenced. Sorted by priority.

---

## 🔴 CRITICAL — Must resolve before breadboard

### 1. Replace DRV8871 with DRV8837DSGR
**Found by:** All 4 reviewers (unanimous)
**Issue:** DRV8871 has 6.5V minimum VM. In V-to-I topology, VM = op-amp output, goes as low as 0.3V. Below UVLO for most of the operating range.
**Also:** Discrete BSS84+BSS138 H-bridge (the "recommended" alternative) ALSO fails — P-channel can't be turned off by 5V logic when source exceeds ~6V (Mara-Opus v2 catch).
**Resolution:** DRV8837DSGR — 0–11V VM, SOT-23-6, built-in dead-time. Compliance capped at ~10V.
**Component to source:** DRV8837DSGR × 4 (2 per channel + 2 spares)
**AGENTS.md:** ✅ Updated

### 2. Add hardware pulse-duration limiter
**Found by:** Kai-GPT v2
**Issue:** If MCU crashes mid-pulse with DAC nonzero, DC-block cap charges at 500 V/s. 130 µC fault charge in 26ms — the 50ms heartbeat watchdog is too slow. Overcurrent comparator won't trip because current is still "only" 5mA.
**Resolution:** Hardware monostable or retriggerable timer that forces H-bridge disable if a phase persists beyond ~1ms. Alternative: require complementary phase completion to re-arm.
**Component to source:** TBD — monostable (74LVC1G123 or similar) or output timeout circuit
**AGENTS.md:** Needs update — add to safety architecture

---

## 🟡 SIGNIFICANT — Must decide before schematic capture

### 3. Add voltage reference MCP1501-10E/SN
**Found by:** Mara-Opus v1 + v2, Kai-GPT v2
**Issue:** MCP4922 has no internal Vref. Neither the precision reference nor the resistor divider is in the final BOM. DAC literally outputs nothing without Vref.
**Resolution:** MCP1501-10E/SN (1.024V, 0.08%, 10ppm/°C, SOIC-8). With 2× gain: Imax=10.24mA, resolution=2.5µA/LSB.
**Component to source:** MCP1501-10E/SN × 2 (1 + spare)
**AGENTS.md:** ✅ Updated

### 4. Add SPI MISO reverse channel — swap SI8621EC for SI8622EC
**Found by:** Mara-Opus v1 + v2
**Issue:** No reverse isolator channel for impedance ADC (MCP3201) readback. Current allocation: 9F + 1R, the 1R is used for fault flag.
**Resolution:** Swap SI8621EC-B-IS (1F+1R) for SI8622EC-B-IS (0F+2R). Gets 2 reverse channels (fault + MISO). Move heartbeat from SI8621's forward to one of SI8380P's 8 channels. 10 total channels, 2 ICs, no BOM count change.
**Component to source:** SI8622EC-B-IS × 2 (1 + spare). Also: MCP3201 × 2 if going proportional impedance.
**AGENTS.md:** Needs update — change SI8621 to SI8622, update channel allocation

### 5. Address PCN1 unbalanced rail loading
**Found by:** Mara-Opus v2
**Issue:** +15V draws ~33mA, -15V draws ~2mA (16:1 imbalance). Unregulated converter cross-regulation will cause +15V to sag.
**Resolution:** Add bleeder resistor footprint on -15V rail (1kΩ, DNP by default). Measure on breadboard. Also add 15V Zener clamp diodes (BZX84C15) on both rails.
**Component to source:** BZX84C15 × 4 (Zener clamps). 1kΩ 0603 bleeder (from misc resistors).

### 6. Use brake mode during H-bridge dead time
**Found by:** Mara-Opus v2
**Issue:** During dead time (50µs), if H-bridge goes Hi-Z (coast), Rsense current drops to 0, op-amp saturates to +14V. Recovery overshoot when H-bridge reconnects.
**Resolution:** Use brake mode (both low-side FETs ON) during dead time. DRV8837: IN1=IN2=0. Keeps feedback loop closed, op-amp in linear region.
**Implementation:** Firmware — Phase 2 (Waveform Engine)

### 7. DC-block cap — add series redundancy
**Found by:** Mara-Opus v2 (new angle)
**Issue:** MLCC fails short-circuit (cracked ceramic). Film fails open-circuit (self-healing). For a DC-blocking safety cap, short = defeats purpose.
**Resolution:** Two MLCC in series per channel. Effective 5µF still adequate (0.2V droop at 5mA/200µs). Cost: +$0.02/channel.
**Component to source:** Samsung CL31B106KBHNNNE × 6 (4 needed + 2 spare). Already have qty 2 on prior BOM — increase to 6.

---

## ℹ️ MINOR — Address during schematic/PCB

### 8. Emergency stop — upgrade for bedside use
Replace tactile switch with ≥12mm pushbutton or slider. Phase 8 (PCB design).

### 9. BSS138 gate pull-down
Add 100kΩ pull-down from gate to source. Prevents parasitic cap holding gate high after USB disconnect.

### 10. LM339 open-collector pull-ups
Add 10kΩ pull-up to 5V_ISO on all LM339 outputs. Verify latch power-up default is "safe" (no stimulation).

### 11. 10nF feedback cap — start DNP
Leave C_F1/C_F2 footprint on PCB but don't populate. Add only if bench testing shows HF issues.

### 12. Bleed resistor — consider 100kΩ
Reduces DC-block cap discharge from τ=10s to τ=1s. 1µA load at 5mA = 0.02% error.

### 13. TPS61023 EN pull-up
Add 10kΩ pull-up to VBAT on EN pin to ensure boost runs when BSS138 is OFF (no USB).

---

## 📄 DOCUMENTATION — Reconcile before schematic

### 14. Merge DESIGN-REVIEW.md and CIRCUIT-DESIGN-REVIEW.md
Two documents contradict each other on: H-bridge (DRV8871 vs discrete), Vref (MCP1501 vs divider), DC-block (film vs MLCC), TPS61023 package (SOT-563 vs SOT-23-5), switching frequency (4MHz vs 1.8MHz), comparator (LM339 vs LM393). Establish one canonical source of truth.

### 15. Compliance language
Change "meets Type BF" → "designed toward BF-style isolation" throughout. Compliance not demonstrated until schematic/layout/leakage testing.

### 16. Loop stability
Change "89° phase margin" → "estimated 50-70° phase margin from hand analysis; verify with SPICE or bench." Conclusion (stable) is correct; the number is overstated.

---

## Components to Source (New/Changed)

| Component | Part Number | Qty | Why |
|-----------|-------------|-----|-----|
| H-Bridge | **DRV8837DSGR** | 4 | Replaces DRV8871. 0-11V VM, SOT-23-6 |
| Voltage reference | **MCP1501-10E/SN** | 2 | MCP4922 Vref. 1.024V precision. SOIC-8 |
| Digital isolator (swap) | **SI8622EC-B-IS** | 2 | Replaces SI8621. Gets MISO reverse channel |
| Impedance ADC | **MCP3201** | 2 | Patient-side impedance measurement |
| Zener clamps | **BZX84C15** | 4 | ±15V rail overvoltage protection. SOT-23 |
| Pulse limiter | **TBD** | - | Hardware pulse-duration timeout |
| DC-block caps (additional) | Samsung CL31B106KBHNNNE | 4 | Series redundancy (2 per channel) |

---

*Generated from 4 independent agent reviews. Cross-reference with individual reports:*
- `.planning/REVIEW-GPT54-WARM-2026-04-24.md` (Kai Round 1)
- `.planning/REVIEW-OPUS-WARM-2026-04-24.md` (Mara Round 1)
- `.planning/REVIEW-GPT54-WARM-V2-2026-04-24.md` (Kai Round 2)
- `.planning/REVIEW-OPUS-WARM-V2-2026-04-24.md` (Mara Round 2)
