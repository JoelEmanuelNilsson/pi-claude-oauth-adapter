# pi-claude-oauth-adapter

Anthropic OAuth / Claude Code compatibility adapter for Pi.

**Install:** `pi install npm:pi-claude-oauth-adapter`

This package patches Anthropic OAuth / Claude Pro/Max sessions in Pi. It strips the docs-only Pi section out of the system prompt, removes the Claude Code identity block, reinjects Pi docs context outside the system prompt when needed, and makes sure the Claude billing header is present for OAuth requests.

## What's new in 0.1.2

- Pi can now show `✓ Claude OAuth ready`, `✓ Claude OAuth active`, or `⚠ Claude OAuth setup` in the footer while the adapter is running.
- The adapter now exposes `claude-oauth-ready` and `claude-oauth-issue` status keys so Pi runtimes can suppress the generic Anthropic subscription warning only when the adapter is actually healthy.

The second point depends on the Pi runtime version. The package publishes the readiness signal; Pi still has to consume it.

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

If you use this repo directly, add the local package path to Pi settings:

```json
{
  "packages": [
    "../../Developer/dotfiles-agents/packages/pi-claude-oauth-adapter"
  ]
}
```

`./setup.sh` in this repo already provisions that path when the active Pi settings symlink points at this checkout.

## Verify it is active

In an Anthropic OAuth session, the package should either:

- show `✓ Claude OAuth ready` before the first request
- show `✓ Claude OAuth active` after a normalized Anthropic OAuth request
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
  - optional billing-header overrides

For most users, no env vars are required.

If you want the stripped docs context available for every request instead of only Pi-related prompts:

```bash
PI_CLAUDE_OAUTH_REINJECT_SCOPE=always pi
```

## Release notes

See [CHANGELOG.md](./CHANGELOG.md).

## Maintainer release flow

```bash
cd packages/pi-claude-oauth-adapter
npm pack --dry-run
npm publish --access public
```

Or from repo root:

```bash
npm publish ./packages/pi-claude-oauth-adapter --access public
```

## Notes

- This package does **not** implement Anthropic auth itself. Pi already has built-in Anthropic OAuth support.
- This package is the compatibility layer on top of Pi's Anthropic OAuth flow.
- It is designed to work both with already-patched Pi builds and older/provider builds that still need the billing header injected at request time.
