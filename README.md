# veritas-component

WarmHub component source repo for Veritas.

This repo is the source that will be registered as `warmhub/veritas` and installed via:

```bash
wh component install warmhub/veritas --repo <org>/<repo>
```

## Contents

- `warmhub/component.json` — component metadata
- `warmhub/manifest.json` — declarative WarmHub contract for Veritas

## Intended model

This manifest is written for the GH-2328 end state:

- WarmHub installs the four Veritas shapes directly from the manifest.
- The three Veritas subscriptions are marked `provisioning: "setup"` so the worker can create install-specific webhook URLs containing its own attached/install identifier.
- The repo-local `veritas-webhook-<org-name>-<repo-name>` credential set is created by the standard manifest installer. It also carries `CLI_SIGNING_SECRET`, used to authenticate the component CLI methods (see below).
- `runtimeAccess` declares the future minted runtime token scope:
  - reads: `Certainty`, `Support`, `Opposition`, `Consensus`, `Pair`
  - writes: `Consensus`, `ComponentConfig` (`ComponentConfig` lets the setup-call publish the worker's `cliBaseUrl` so CLI dispatch knows where to send method calls)

## CLI methods

The manifest exposes a `cli` surface so install-repo operators can read and adjust
Veritas source reputations (which live in Veritas's own data repo, not the install
repo). Each method authenticates with `CLI_SIGNING_SECRET` from the credential set
above; the worker verifies the canonical-request HMAC.

| Command | Verb | Route | Args | Requires |
|---------|------|-------|------|----------|
| `list-reputations` | `GET` | `/reputations` | `scope?`, `limit?` (1–500, default 50), `cursor?` | `repo:read` |
| `get-reputation` | `GET` | `/reputation` | `wref`, `scope` | `repo:read` |
| `upsert-reputation` | `PUT` | `/reputation` | `wref`, `scope`, `belief`, `disbelief`, `uncertainty` (each in [0, 1], summing to 1) | `repo:write` |

Routes are decoupled from the command names via each method's `path` field, which is
why `get-reputation` and `upsert-reputation` can share `/reputation` (distinguished by
verb). Operators invoke them as `wh veritas <command> --repo <org>/<repo>`.

## Registration target

The intended registration shape is:

```bash
wh component register veritas \
  --org warmhub \
  --source-url https://github.com/warmhub/veritas-component \
  --setup-url https://<veritas-worker>/setup \
  --cred-set veritas-setup \
  --minted-tokens \
  --description "Subjective Logic consensus computation"
```

## Transition note

Today the local Veritas worker still uses the legacy `/attachRepo` flow. This repo is meant to unblock the future registered-component path and should be paired with the worker migration to `/setup` plus setup/runtime token handling.

The reputation CLI endpoints (`/reputations`, `/reputation`) are already implemented on the worker; the `cli` surface above is what wires them into `wh`. The remaining gap is the registered-component setup path — the setup-call must publish `cliBaseUrl` into `ComponentConfig` and write `CLI_SIGNING_SECRET` into the credential set for dispatch to reach those endpoints.
