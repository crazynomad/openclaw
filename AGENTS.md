# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository

- Repo: https://github.com/openclaw/openclaw
- GitHub issues/comments/PR comments: use literal multiline strings or `-F - <<'EOF'` (or $'...') for real newlines; never embed "\\n".

## Quick Reference

```bash
# Install & setup
pnpm install              # Install dependencies
prek install              # Pre-commit hooks (same checks as CI)

# Development
pnpm openclaw <cmd>       # Run CLI in dev mode
pnpm gateway:dev          # Run gateway (skips channels)
bun <file.ts>             # Execute TypeScript directly

# Quality gates
pnpm build                # Type-check and build
pnpm check                # Lint + format (oxlint, oxfmt)
pnpm test                 # Run tests (vitest)
pnpm test:coverage        # Tests with coverage

# Single test file
bun vitest run src/path/to/file.test.ts

# Live tests (requires API keys)
OPENCLAW_LIVE_TEST=1 pnpm test:live

# Mac app
scripts/package-mac-app.sh  # Build macOS app
scripts/restart-mac.sh      # Restart gateway via app
```

## Architecture Overview

OpenClaw is a multi-platform AI assistant gateway that bridges messaging channels (WhatsApp, Telegram, Discord, Slack, Signal, iMessage, etc.) to AI providers (Anthropic, OpenAI, Google, etc.).

**Key architectural layers:**
- **CLI (`src/cli/`)** - Commander-based CLI with commands in `src/commands/`
- **Gateway (`src/gateway/`)** - WebSocket server that manages channels, agents, and message routing
- **Channels (`src/channels/`, `src/telegram/`, `src/discord/`, etc.)** - Messaging platform adapters
- **Agents (`src/agents/`)** - Pi-based AI agent runtime with tool support
- **Plugins (`extensions/*/`)** - Workspace packages for optional channel/feature extensions
- **Native apps (`apps/macos/`, `apps/ios/`, `apps/android/`)** - Platform-specific clients

**Data flow:** Incoming message → Channel adapter → Gateway → Agent → AI provider → Response → Channel adapter → Outgoing message

**Extension points:**
- `src/plugin-sdk/index.ts` - Plugin SDK exports for extensions
- `src/channels/plugins/types.ts` - Channel adapter interfaces
- `src/hooks/` - Lifecycle hooks system

## Project Structure

- **`src/`** - Source code (CLI in `src/cli`, commands in `src/commands`, infra in `src/infra`, media in `src/media`)
- **`src/gateway/`** - Gateway server (WebSocket, HTTP, channel coordination)
- **`src/channels/`** - Channel abstraction layer and plugin system
- **`extensions/*/`** - Workspace packages for plugins (keep deps in extension `package.json`)
- **`apps/`** - Native apps: `macos/`, `ios/`, `android/`, `shared/` (OpenClawKit)
- **`docs/`** - Mintlify documentation (hosted at docs.openclaw.ai)
- **`dist/`** - Built output
- Tests are colocated as `*.test.ts`; e2e tests as `*.e2e.test.ts`

**Plugin dependency rules:**
- Runtime deps must be in `dependencies` (install runs `npm install --omit=dev`)
- Avoid `workspace:*` in `dependencies`; use `devDependencies` or `peerDependencies` for `openclaw`

**Messaging channels:** When refactoring shared logic (routing, allowlists, pairing), consider ALL channels:
- Core: `src/telegram`, `src/discord`, `src/slack`, `src/signal`, `src/imessage`, `src/web` (WhatsApp)
- Extensions: `extensions/msteams`, `extensions/matrix`, `extensions/zalo`, etc.
- Review `.github/labeler.yml` for label coverage when adding channels

## Docs Linking (Mintlify)

- Docs are hosted on Mintlify (docs.openclaw.ai).
- Internal doc links in `docs/**/*.md`: root-relative, no `.md`/`.mdx` (example: `[Config](/configuration)`).
- Section cross-references: use anchors on root-relative paths (example: `[Hooks](/configuration#hooks)`).
- Doc headings and anchors: avoid em dashes and apostrophes in headings because they break Mintlify anchor links.
- When Peter asks for links, reply with full `https://docs.openclaw.ai/...` URLs (not root-relative).
- When you touch docs, end the reply with the `https://docs.openclaw.ai/...` URLs you referenced.
- README (GitHub): keep absolute docs URLs (`https://docs.openclaw.ai/...`) so links work on GitHub.
- Docs content must be generic: no personal device names/hostnames/paths; use placeholders like `user@gateway-host` and “gateway host”.

## Docs i18n (zh-CN)

- `docs/zh-CN/**` is generated; do not edit unless the user explicitly asks.
- Pipeline: update English docs → adjust glossary (`docs/.i18n/glossary.zh-CN.json`) → run `scripts/docs-i18n` → apply targeted fixes only if instructed.
- Translation memory: `docs/.i18n/zh-CN.tm.jsonl` (generated).
- See `docs/.i18n/README.md`.
- The pipeline can be slow/inefficient; if it’s dragging, ping @jospalmbier on Discord instead of hacking around it.

## exe.dev VM ops (general)

- Access: stable path is `ssh exe.dev` then `ssh vm-name` (assume SSH key already set).
- SSH flaky: use exe.dev web terminal or Shelley (web agent); keep a tmux session for long ops.
- Update: `sudo npm i -g openclaw@latest` (global install needs root on `/usr/lib/node_modules`).
- Config: use `openclaw config set ...`; ensure `gateway.mode=local` is set.
- Discord: store raw token only (no `DISCORD_BOT_TOKEN=` prefix).
- Restart: stop old gateway and run:
  `pkill -9 -f openclaw-gateway || true; nohup openclaw gateway run --bind loopback --port 18789 --force > /tmp/openclaw-gateway.log 2>&1 &`
- Verify: `openclaw channels status --probe`, `ss -ltnp | rg 18789`, `tail -n 120 /tmp/openclaw-gateway.log`.

## Build, Test, and Development

- **Runtime:** Node 22+ required (keep both Node + Bun paths working)
- **Package manager:** pnpm (`pnpm install`); Bun also supported (`bun install`)
- **TypeScript execution:** Prefer Bun for scripts/dev (`bun <file.ts>`)
- **Built output:** Node for `dist/*` and production installs
- **Mac packaging:** `scripts/package-mac-app.sh` (see `docs/platforms/mac/release.md`)

## Coding Style

- **Language:** TypeScript (ESM), strict typing, avoid `any`
- **Formatting:** Oxlint + Oxfmt; run `pnpm check` before commits
- **File size:** Aim for ~500-700 LOC max; split/refactor when it improves clarity
- **Patterns:** Use `createDefaultDeps()` for dependency injection; extract helpers instead of "V2" copies
- **Comments:** Brief comments for tricky logic only
- **Naming:** **OpenClaw** for product/docs headings; `openclaw` for CLI/package/paths/config

## Release Channels (Naming)

- stable: tagged releases only (e.g. `vYYYY.M.D`), npm dist-tag `latest`.
- beta: prerelease tags `vYYYY.M.D-beta.N`, npm dist-tag `beta` (may ship without macOS app).
- dev: moving head on `main` (no tag; git checkout main).

## Testing

- **Framework:** Vitest with V8 coverage (70% threshold for lines/branches/functions/statements)
- **Naming:** `*.test.ts` colocated with source; `*.e2e.test.ts` for e2e
- **Workers:** Do not exceed 16 workers
- **Live tests:** `OPENCLAW_LIVE_TEST=1 pnpm test:live` or `LIVE=1 pnpm test:live` (includes providers)
- **Docker tests:** `pnpm test:docker:live-models`, `pnpm test:docker:live-gateway`, `pnpm test:docker:onboard`
- **Mobile:** Prefer real devices over simulators when available
- **Changelog:** Pure test fixes don't need changelog entries unless they alter user-facing behavior
- Full testing guide: `docs/testing.md`

## Commits & Pull Requests

**Commits:**
- Use `scripts/committer "<msg>" <file...>` (avoid manual `git add`/`git commit`)
- Concise, action-oriented messages (e.g., `CLI: add verbose flag to send`)
- Group related changes; don't bundle unrelated refactors

**Changelog:**
- Keep latest released version at top (no `Unreleased` section)
- Add entry with PR # and thank contributor when working on PRs
- Reference issue # when working on issues

**PR Review:**
- Use `gh pr view`/`gh pr diff`; do NOT change branches
- Run `git pull` first; stop if local changes/unpushed commits exist

**PR Merge:**
- Goal: merge PRs. Prefer **rebase** (clean history) or **squash** (messy history)
- Flow: temp branch from `main` → merge/squash PR → add changelog + thanks → run full gate (`pnpm build && pnpm check && pnpm test`) → merge to `main`
- Always add PR author as co-contributor when squashing
- Leave PR comment with merge explanation and SHA hashes
- New contributors: add avatar to README "clawtributors" list; run `bun scripts/update-clawtributors.ts`

**Shorthand:** `sync` = commit dirty changes → `git pull --rebase` → `git push` (stop on conflicts)

## Configuration & Security

- **Credentials:** `~/.openclaw/credentials/` (rerun `openclaw login` if logged out)
- **Sessions:** `~/.openclaw/sessions/` (base directory not configurable)
- **Agent sessions:** `~/.openclaw/agents/<agentId>/sessions/*.jsonl`
- Never commit real phone numbers, videos, or live config values; use fake placeholders
- Release flow: read `docs/reference/RELEASING.md` and `docs/platforms/mac/release.md` first

## Troubleshooting

- Rebrand/migration issues or legacy config/service warnings: run `openclaw doctor` (see `docs/gateway/doctor.md`).

## Version Locations

- `package.json` (CLI)
- `apps/android/app/build.gradle.kts` (versionName/versionCode)
- `apps/ios/Sources/Info.plist` + `apps/ios/Tests/Info.plist`
- `apps/macos/Sources/OpenClaw/Resources/Info.plist`
- `docs/install/updating.md` (pinned npm version)

## Agent-Specific Notes

**Vocabulary:** "makeup" = "mac app"

**Dependencies:**
- Never edit `node_modules`
- Never update the Carbon dependency
- Patched dependencies (`pnpm.patchedDependencies`) must use exact versions (no `^`/`~`)
- Patching deps requires explicit approval

**CLI/Terminal patterns:**
- Progress: use `src/cli/progress.ts` (not hand-rolled spinners)
- Tables: use `src/terminal/table.ts` for ANSI-safe wrapping
- Colors: use `src/terminal/palette.ts` (no hardcoded colors)
- Status: `--all` = read-only/pasteable, `--deep` = probes

**macOS:**
- Gateway runs as menubar app only (no separate LaunchAgent)
- Restart via app or `scripts/restart-mac.sh`
- Logs: `./scripts/clawlog.sh` for unified logs
- Do not rebuild macOS app over SSH

**SwiftUI:** Prefer `Observation` framework (`@Observable`, `@Bindable`) over `ObservableObject`/`@StateObject`

**Tool schemas (google-antigravity):**
- Avoid `Type.Union`, `anyOf`/`oneOf`/`allOf`
- Use `stringEnum`/`optionalStringEnum` for string lists
- Avoid raw `format` property names
- Top-level schema must be `type: "object"` with `properties`

**Multi-agent safety:**
- Do NOT create/apply/drop `git stash` unless explicitly requested
- Do NOT switch branches unless explicitly requested
- Do NOT modify `git worktree` checkouts
- "push" → may `git pull --rebase`; "commit" → scope to your changes only
- Focus on your edits; continue if safe when multiple agents touch same file

**Lint/format churn:**
- Auto-resolve formatting-only diffs without asking
- Only ask when changes are semantic (logic/data/behavior)

**Release guardrails:**
- Do not change version numbers without explicit consent
- Always ask before npm publish/release steps

## NPM + 1Password (publish/verify)

- Use the 1password skill; all `op` commands must run inside a fresh tmux session.
- Sign in: `eval "$(op signin --account my.1password.com)"` (app unlocked + integration on).
- OTP: `op read 'op://Private/Npmjs/one-time password?attribute=otp'`.
- Publish: `npm publish --access public --otp="<otp>"` (run from the package dir).
- Verify without local npmrc side effects: `npm view <pkg> version --userconfig "$(mktemp)"`.
- Kill the tmux session after publish.
