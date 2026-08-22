# pi-claude-oauth-adapter

Anthropic OAuth / Claude Code compatibility adapter for Pi.

**Install:** `pi install npm:pi-claude-oauth-adapter`

This package patches Anthropic OAuth / Claude Pro/Max sessions in Pi. It strips the docs-only Pi section out of the system prompt, removes the Claude Code identity block, reinjects Pi docs context outside the system prompt when needed, and makes sure the Claude billing header is present for OAuth requests.

## What's new in 0.2.0

- The injected Claude billing header now tracks Claude Code `2.1.226`; `cch=00000` is only included for first-party Anthropic requests.
- The quota probe now uses Claude Code's external CLI user agent, sends only the OAuth beta for the Haiku probe, and tries `GET /api/oauth/usage` before falling back to a tiny messages request.
- Unified usage-limit parsing now understands `7d_oi` / Fable 5 limits, overage utilization, overage in-use state, usage-credit wording, and grace-window warnings.

## What's new in 0.1.4

- The injected Claude billing header now tracks Claude Code `2.1.126` (`cc_version` hash input, entrypoint, fixed `cch=00000`, optional `cc_workload`).
- Unified usage-limit parsing now preserves the latest `anthropic-ratelimit-unified-fallback` and `anthropic-ratelimit-unified-upgrade-paths` fields.
- 429 responses that include representative-claim or overage headers but omit `anthropic-ratelimit-unified-status` are treated as rejected, matching Claude Code's current fallback path.

## What's new in 0.1.3

- Pi now surfaces Claude unified usage-limit state on the real 429 path, including a follow-up quota check that rewrites generic Anthropic rate-limit failures into Claude-style messages like `You've hit your limit · resets 10:30pm (Asia/Colombo)`.
- The injected Claude billing header now matches Claude Code `2.1.118` semantics more closely: updated `cc_version`, fixed `cch=00000`, and optional `cc_workload` passthrough.
- The adapter still exposes `claude-oauth-ready` and `claude-oauth-issue` status keys so Pi runtimes can suppress generic Anthropic subscription warnings only when the adapter is actually healthy.
- On 429s, the adapter patches Anthropic's generic `rate_limit_error` into the resolved Claude usage-limit message so Pi's auto-retry logic stops thrashing.

The last point depends on the Pi runtime version. The package publishes the readiness signal; Pi still has to consume it.

## When this package does anything

It only activates when both of these are true:

- the selected provider is `anthropic`
- Pi is using Anthropic OAuth / subscription auth

If you use `ANTHROPIC_API_KEY` only, this package stays inactive.

## Install

### From npm

```bash
pi install npm:pi-claude-oauth-adapter
pi list
```

Then inside Pi:

1. run `/login`
2. choose **Claude Pro/Max**
3. pick an Anthropic model in `/model`
4. start using Pi normally

### From a local checkout

```bash
git clone git@github.com:minzique/pi-claude-oauth-adapter.git
pi install ./pi-claude-oauth-adapter
```

Or add the checkout path to Pi settings:

```json
{
  "packages": [
    "../../Developer/pi-claude-oauth-adapter"
  ]
}
```

## Verify it is active

In an Anthropic OAuth session, the package should either:

- show `✓ Claude OAuth ready` before the first request
- show `✓ Claude OAuth active` after a normalized Anthropic OAuth request
- show a real Claude usage status like `You've hit your limit · resets 10:30pm (Asia/Colombo)` when Anthropic rejects the request and the adapter resolves quota state via the follow-up check
- show `⚠ Claude OAuth setup` if the adapter is enabled but missing the docs context it needs

If you are using API-key auth instead of OAuth, none of those statuses should appear.

## Config

Environment variables:

- `PI_CLAUDE_OAUTH_REINJECT_SCOPE=never|always|pi-only`
  - default: `pi-only`
- `PI_CLAUDE_OAUTH_REINJECT_MODE=prepend-custom-message|append-custom-message|user-reminder|none`
  - default: `prepend-custom-message`
- `PI_CLAUDE_OAUTH_LOG_FILE=/path/to/log.jsonl`
  - optional debug logging
- `PI_CLAUDE_OAUTH_DOCS_FILE=/path/to/pi-docs-only.txt`
  - optional docs fallback override
- `PI_CLAUDE_CODE_VERSION=...`
- `PI_CLAUDE_CODE_ENTRYPOINT=...`
- `PI_CLAUDE_CODE_WORKLOAD=...`
- `PI_CLAUDE_CODE_SUBSCRIPTION_TYPE=...`
  - optional billing-header / footer-label overrides

For most users, no env vars are required.

If you want the stripped docs context available for every request instead of only Pi-related prompts:

```bash
PI_CLAUDE_OAUTH_REINJECT_SCOPE=always pi
```

## Release notes

See [CHANGELOG.md](./CHANGELOG.md).

## Development

```bash
npm install
npm run check
npm pack --dry-run
```

## Maintainer release flow

From the repository root:

```bash
npm run check
npm publish --access public
```

## Notes

- This package does **not** implement Anthropic auth itself. Pi already has built-in Anthropic OAuth support.
- This package is the compatibility layer on top of Pi's Anthropic OAuth flow.
- It is designed to work both with already-patched Pi builds and older/provider builds that still need the billing header injected at request time.
- Full `user-agent` / provider-header parity with Claude Code does **not** belong in this package. Pi's provider override API is provider-wide, not OAuth-scoped, so that part should land in `@mariozechner/pi-ai` instead of changing all Anthropic traffic from here.
