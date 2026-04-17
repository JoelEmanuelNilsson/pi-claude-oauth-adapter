# Changelog

All notable changes to `pi-claude-oauth-adapter` live here.

## 0.1.2 — 2026-04-17

- Added adapter health status in Pi's footer so Anthropic OAuth sessions can show `✓ Claude OAuth ready`, `✓ Claude OAuth active`, or `⚠ Claude OAuth setup`.
- Exposed the `claude-oauth-ready` and `claude-oauth-issue` status keys for Pi runtimes that want to gate the generic Anthropic subscription warning on real adapter readiness.
- Refreshed the package README so npm users get install, verification, and release guidance instead of repo-local notes only.

## 0.1.1 — 2026-04-17

- Hardened optional debug logging for adapter troubleshooting.

## 0.1.0 — 2026-04-08

- Initial public release of the Anthropic OAuth compatibility adapter for Pi.
