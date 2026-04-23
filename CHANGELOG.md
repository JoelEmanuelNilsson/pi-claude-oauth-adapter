# Changelog

All notable changes to `pi-claude-oauth-adapter` live here.

## 0.1.3 — 2026-04-23

- Surface Claude usage-limit state on the real 429 path by running a follow-up quota check and rewriting Anthropic's generic `rate_limit_error` into Claude-style limit/reset messages.
- Sync the injected Claude billing header to Claude Code `2.1.118` semantics (`cc_version`, fixed `cch=00000`, optional `cc_workload`).
- Stop Pi auto-retry thrash on Anthropic subscription limits by replacing the retryable `429` error text with the resolved Claude usage-limit message.
- Document that full `user-agent` parity belongs in `@mariozechner/pi-ai`, not this package, because package-level provider overrides are provider-wide rather than OAuth-scoped.

## 0.1.2 — 2026-04-17

- Added adapter health status in Pi's footer so Anthropic OAuth sessions can show `✓ Claude OAuth ready`, `✓ Claude OAuth active`, or `⚠ Claude OAuth setup`.
- Exposed the `claude-oauth-ready` and `claude-oauth-issue` status keys for Pi runtimes that want to gate the generic Anthropic subscription warning on real adapter readiness.
- Refreshed the package README so npm users get install, verification, and release guidance instead of repo-local notes only.

## 0.1.1 — 2026-04-17

- Hardened optional debug logging for adapter troubleshooting.

## 0.1.0 — 2026-04-08

- Initial public release of the Anthropic OAuth compatibility adapter for Pi.
