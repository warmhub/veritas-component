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
- The repo-local `veritas-webhook` credential set is created by the standard manifest installer.
- `runtimeAccess` declares the future minted runtime token scope:
  - reads: `Certainty`, `Support`, `Opposition`
  - writes: `Consensus`

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
