# AEXYR Desktop — Changelog

All notable changes to this project will be documented in this file.

---

## [v0.1.0] — 2026-09-11 — Project inception and architecture planning
- Created: Initial project documentation (blueprint, changelog, pitfalls)
- Decided: Tauri v2 as framework (over Electron) — ~15MB binary, Rust backend, system webview
- Decided: Two-channel connection model — HTTPS/WSS for dashboards + SSH for agent interaction
- Decided: Three-layer auth (CF Access → SSH keys → API token)
- Decided: OS-native keychain for credential storage
- Decided: GitHub Actions CI/CD for cross-platform builds (macOS, Windows, Linux)
- Decided: Skip code signing for v1 ($0 cost), add in v1.1.0
- Designed: UI layout — Instance sidebar, Topology, Server Rack, Vitals, Agent Chat, Logs, Files, SSL
- Identified: All panels consume existing AEXYR APIs — zero backend changes needed
- Created: GitHub repo axonstellar/aexyr-desktop (public, Coming Soon README)
- Status: Pre-development — architecture and specification phase
- Files: service_blueprint.md, service_changelog.md, service_pitfalls.md
