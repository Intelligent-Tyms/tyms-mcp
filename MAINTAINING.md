# Maintaining this repo

## Source of truth

This repository is a **RENDERED COPY**. Do not edit bundle metadata here directly.

- Registry metadata (names, titles, descriptions, endpoint URLs) lives in the
  internal workspace at `mcp-distribution/server.json` (the `drafts` array).
- Demo queries live in `mcp-push/demos/<bundle>.md`.
- Distribution conventions (venue UTM attribution, deeplink format) live in
  `mcp-distribution/playbook.md` (section 0.3) and `mcp-distribution/listing-copy.md`.

**Rule:** any change to bundle inventory, endpoint URLs, tool surfaces, or
descriptions requires re-rendering this repo in the same change.

## Sync steps

1. Edit the source of truth:
   - `mcp-distribution/server.json` for registry/bundle metadata.
   - `mcp-push/demos/<bundle>.md` for demo queries.
2. Re-render this repo:
   - Copy `mcp-distribution/server.json` verbatim to `server.json`
     (registry `remotes[].url` stays canonical, no query params).
   - Copy the 12 bundle demo files into `demos/`, applying the render-time
     sanitization below. Never copy `*-report.md`, `retest-*`,
     `issues.md`, or `demo-runbook.md`.
   - Regenerate the README bundle table, `llms-install.md` server table,
     `mcp.json`, `gemini-extension.json`, `.cursor-plugin/plugin.json` and
     `.claude-plugin/plugin.json` from the bundle list (install/config URLs
     carry per-venue `?venue=` attribution per playbook section 0.3).

**Adding a future bundle:** in the SAME commit and push, also generate its
Cursor one-click deeplink
(`https://cursor.com/en/install-mcp?name=tyms-<slug>&config=<base64url of {"url":"<endpoint>?venue=cursor"}>`)
and add it to the Cursor deeplink list in `llms-install.md`, alongside the
README table row, the `llms-install.md` server-table row, the demo file, and
the plugin/extension manifests. A bundle is not rendered until its deeplink
ships with everything else.
3. Render-time sanitization (hard rule — the internal demo files record the
   test environment; the public copies must not):
   - Replace `Environment: <preview-host> (...)` lines with
     `Environment: pre-production preview deployment.`
   - Replace `Environment: production via connected ... Worker version ...`
     lines with `Environment: production.`
   - Remove any mention of internal hostnames, branch names, commit hashes,
     worker names, access tokens, or environment variables.
4. Secret-scan the tree before every push:
   `grep -rEi 'tok_|Bearer|eyJ|ccb45bd|preview read token|mi-api-preview|[0-9a-f]{32,}' .`
   must return nothing outside this file (which lists the patterns as
   documentation).
5. Commit and push to `Intelligent-Tyms/tyms-mcp`.

## What this repo is NOT

- No runtime code. The servers are hosted at `mi-api.tyms.ai`; this repo only
  carries manifests, registry drafts and docs.
- Not the place for privacy policy or terms text — link to tyms.ai pages once
  they exist (currently TODO in README).
