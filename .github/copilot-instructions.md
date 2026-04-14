# Copilot Instructions — OpenBinaural-taVNS

## Identity Trigger
When the user mentions **SuitCatClub** as their git identity, or this is the active project:
1. Immediately read `AGENTS.md` from the repo root — it contains everything you need
2. Set local git identity: `SuitCatClub` / `suitcatclub@gmail.com`
3. Set `core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"` (Windows OpenSSH, not Git bundled)
4. Set `commit.gpgsign false` locally (global config has gpgsign=true — will break commits)
5. After any reboot: remind user to run `ssh-add ~/.ssh/id_ed25519_suitcatclub`

## This Project
- **What:** Open-source DIY Binaural taVNS device for insomnia self-experimentation
- **Repo:** `github-suitcatclub:SuitCatClub/taVNS-Binaural.git`
- **Workflow:** GSD (Get Shit Done) — read `.planning/STATE.md` for current phase
- **Model:** claude-opus-4.6 (medical precision project)
- **Full context:** See `AGENTS.md` in repo root

## Never Do
- Never access E: drive (does not exist on this machine)
- Never use AMS1117 (wrong LDO for LiPo) — use ME6211
- Never use B0515S (single rail) — use B0515D-1WR3 (dual ±15V)
- Never use NimBLE < 2.4.0 — ESP32-S3 dual-role BLE regression
- Never skip galvanic isolation — patient-facing circuitry always behind isolation barrier
