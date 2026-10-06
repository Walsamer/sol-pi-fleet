# sol-pi-fleet — Fleet-maintained fork of NVlabs/SoL-Pi

This repository is a **declared Fleet fork** of [`NVlabs/SoL-Pi`](https://github.com/NVlabs/SoL-Pi)
(Level 5 in the Fleet external-integration charter, see
`dev-ops-fleet/docs/architecture/external-integrations.md` §7). It carries the
minimal compatibility change required to run SoL-Pi on a Pi release at or after
1.0, where upstream is only validated against Pi 0.85.1.

## Provenance

| Field | Value |
| --- | --- |
| Upstream | `https://github.com/NVlabs/SoL-Pi` |
| Upstream base commit | `e1a586af0ad8956f42ae5b26bba20e48fbf30e00` (2026-10-01) |
| Fork name | `sol-pi-fleet` |
| Fork owner | Fleet (dev-ops-fleet) |
| License | MIT (unchanged; `LICENSE` preserved) |
| Fleet register | `config/integrations.json` → `mode: "fork"`, `fork_repo`, `expected_revision` |
| First fork commit | _recorded by Fleet when the integration is wired_ |

## Why a fork (Level-1–4 insufficiency)

SoL-Pi is not an executable Fleet invokes; it is **source loaded as an extension
inside Pi** (Level 4 staging). Pi 1.0 changes the contract between the harness
and an extension:

1. A tool definition's `execute(...)` receives `ExtensionToolContext`
   (`ExtensionContext` + `tools`/`executeTool`), not `ExtensionContext`.
2. Tool failures are returned as a result with `isError: true` instead of being
   thrown.

A Fleet-owned adapter or compatibility layer cannot translate these (they are
the harness↔extension boundary itself), and staging the unmodified upstream
source produces a runtime that does not typecheck against the peer dependency
and whose fused-command failure path silently reports success. There is no
upstream release that supports Pi 1.0.x. The lowest sufficient Fleet level is
therefore Level 5: a real, declared, versioned fork.

## Divergence policy

- **Minimal and documented.** The fork changes only what Pi 1.0.x requires; every
  change is listed below and in the fork commit message.
- **No silent divergence.** Any future Fleet-owned change must be recorded in
  this file and in the Fleet integration registry.
- **Rebase-friendly.** Changes are isolated so an upstream rebase is mechanical.
- **Provenance preserved.** SPDX headers, `LICENSE` and `THIRD_PARTY_NOTICES.md`
  are kept; upstream is retained as a git remote.

## Fork changes relative to the base commit

Compatibility:

- `src/sol-pi/extensions/action-fusion/then-run.ts` — accept
  `ExtensionToolContext`; treat `isError` results from the fused `then_run`
  command as a failure (`[then_run:failed]`) instead of the previous thrown
  error.

Tests / tooling:

- `tests/helpers.ts`, `tests/action-fusion.test.ts`,
  `tests/action-fusion-paths.test.ts` — construct `ExtensionToolContext` for
  tool execution.
- `package.json`, `package-lock.json` — Pi development dependencies raised from
  `0.85.1` to `1.0.4`.
- `agents-install.md`, `README.md`, `THIRD_PARTY_NOTICES.md`,
  `docs/compatibility.md` — state the validated Pi release.

## Validation

Zero-spend, deterministic:

```bash
npm ci --ignore-scripts
npm run check                       # typecheck + 195 tests + pack
npx vitest run tests/pi-package-integration.test.ts   # real AgentSession
```

Result recorded by Fleet in `runtime/sol-pi-smoke/real-runtime-result.json`.

## Rollback

Revert to the upstream base commit (`git checkout e1a586af...`) and disable the
Fleet `sol_pi` integration (`config/workers.json` → `"sol_pi": {"enabled": false}`).
No database migration or queue reconciliation is involved.
