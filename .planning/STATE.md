# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2025-07-14)

**Core value:** A fully open, clinically-grounded taVNS device that any technically-capable person can build, replicate, and verify — with firmware, hardware schematics, BOM, and HRV verification protocol all included.
**Current focus:** Phase 1 — TENS Hack — Clinical Parameter Validation

## Current Position

Phase: 1 of 9 (TENS Hack — Clinical Parameter Validation)
Plan: 0 of TBD in current phase
Status: Ready to plan
Last activity: 2025-07-14 — Roadmap created. Research complete. Requirements defined (41 v1 reqs).

Progress: [░░░░░░░░░░] 0%

## Performance Metrics

**Velocity:**
- Total plans completed: 0
- Average duration: —
- Total execution time: 0 hours

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| — | — | — | — |

**Recent Trend:**
- Last 5 plans: —
- Trend: Not started

*Updated after each plan completion*

## Accumulated Context

### Decisions

Decisions are logged in PROJECT.md Key Decisions table.
Recent decisions affecting current work:

- [Research]: V-to-I converter chosen over Howland pump — unconditionally stable with electrode capacitance, H-bridge provides floating capability
- [Research]: DRV8871 integrated H-bridge over discrete BSS138/BSS84 — built-in shoot-through protection, 45V rating handles ±15V compliance
- [Research]: OPA388 auto-zero op-amp in V-to-I topology — ≤5µV offset eliminates dominant DC tissue damage risk
- [Research]: ME6211 LDO selected, AMS1117 rejected — 1.1V dropout incompatible with LiPo direct input
- [Research]: B0515D-1WR3 (dual ±15V) not B0515S (single +15V) — need both rails for op-amp supply
- [Research]: NimBLE ≥2.5.0 required — 2.3.x has rc=519 regression breaking ESP32-S3 dual-role BLE
- [Research]: Anti-inflammatory preset corrected from 10Hz to 25Hz — no published human taVNS evidence at 10Hz
- [Research]: RESET-AF preset set to unilateral left ear — bilateral not validated in original trial protocol

### Pending Todos

None yet.

### Blockers/Concerns

- [Phase 3/4]: SPI through digital isolator at speed is highest-risk unknown — must bench-test at 5/10/20 MHz before PCB commitment
- [Phase 3/4]: B0515D-1WR3 DC-DC noise on ±15V rails — may need LM317/LM337 post-regulation if ripple >20mV
- [Phase 6]: Polar H10 BLE HRS notification format needs hands-on validation with actual hardware
- [Phase 8]: B0515D-1WR3 stock intermittently constrained — RECOM RB-0515D identified as backup

## Session Continuity

Last session: 2025-07-14
Stopped at: Roadmap and state initialized. Ready to plan Phase 1.
Resume file: None
