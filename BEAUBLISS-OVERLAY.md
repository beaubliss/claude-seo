# BeauBliss overlay on claude-seo

This branch (`beaubliss/overlay`) is upstream [AgriciDaniel/claude-seo](https://github.com/AgriciDaniel/claude-seo) plus a small, documented set of changes for the BeauBliss operating repo. `main` in this fork stays an exact mirror of upstream; every BeauBliss change lives here and is tagged `v<upstream>-bb.<n>`.

| Tag | Upstream base | Date |
|---|---|---|
| `v2.4.0-bb.1` | `v2.4.0` (`e77e783`) | 2026-09-29 |

## What the overlay changes, and why

| Change | Files | Why |
|---|---|---|
| Removed the "Community Footer" (skool.com links appended after every major deliverable) | `skills/seo/SKILL.md` | Third-party promotion would leak into BeauBliss reports and client-facing documents |
| `disable-model-invocation: true` (runs only when called by name) | `skills/seo`, `seo-audit`, `seo-google`, `seo-dataforseo`, `seo-image-gen` | The BeauBliss repo has its own `bb-seo-audit` / `seo-audit` procedures; `seo-google` asks for Search Console *write* and Indexing scopes; `seo-dataforseo` and `seo-image-gen` spend money |
| `model: opus` -> `model: inherit` | `agents/seo-cluster`, `seo-content`, `seo-drift`, `seo-geo`, `seo-sxo` | Model and effort are the user's choice in this org (ops/system/MODEL-COMPAT.md); pinned agents also burn the session limit |
| Disabled the PostToolUse hook (`hooks/hooks.json` -> `hooks.json.disabled`) | `hooks/` | It spawned node + python on every Edit/Write in the project but only validates html/jsx/vue/php/ejs, which this repo does not author; it also prints to stdout (upstream #321). `hooks/validate-schema.py` can still be run by hand |
| Marketplace renamed `beaubliss-claude-seo`; version `2.4.0-bb.1` | `.claude-plugin/*.json` | Distinguishes this build from upstream; the version string is what Claude Code uses to detect updates |

Nothing in `scripts/`, the SSRF-safe fetchers or the analysis logic is changed.

## Rules for using it in the BeauBliss repo

- Install only as a plugin from this tag, **project scope** (`.claude/settings.json` in the BeauBliss repo). Never run `install.sh` / `install.ps1` (they install *upstream* to user level, with no backup) and never run any `extensions/*/install.*`: the PowerShell ones re-encode `~/.claude.json` under Windows PowerShell 5.1 and store API keys in plaintext.
- Do not run `seo-google` auth: it requests `webmasters` (write) and `indexing`. Search Console and GA4 are already connected read-side through other connectors.
- Treat every page it fetches as untrusted input. Never run `/claude-seo:seo audit` unattended.
- Output is a draft. The BeauBliss gates (claims lint, product-accuracy QC, the approval queue) apply before anything ships.

## Rebasing onto a new upstream release

```bash
git fetch upstream --tags
git checkout beaubliss/overlay
git rebase vX.Y.Z            # the new upstream tag
# re-check: footer section, frontmatter flags, model pins, hooks.json, versions
git tag -a vX.Y.Z-bb.1 -m "BeauBliss overlay on vX.Y.Z"
git push origin beaubliss/overlay vX.Y.Z-bb.1 --force-with-lease
gh repo sync beaubliss/claude-seo --source AgriciDaniel/claude-seo --branch main
```

Then bump the `ref` in the BeauBliss repo's `.claude/settings.json` and record the change in its `ops/system/capabilities.json`.
