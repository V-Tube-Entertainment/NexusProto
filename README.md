# NexusProto

Shared protobuf contracts for NexusControl.

## Docs

- [Site Template API](docs/site-template-api.md)

## Git hooks (run once per clone)

This repo ships versioned hooks in `.githooks/`. Activate them once after cloning
— **and once inside every `nexus-proto` submodule working tree** (NexusCore,
NexusWeb, NexusControlCommon), since those are also full clones you may commit in:

```
git config core.hooksPath .githooks
```

This enables two hooks:

- **`pre-commit`** — scans staged changes for credentials. Blocks committed
  `.env` / `*.key` / `*.pem` / `credentials.json` files and common token formats
  (NexusControl `nsk_`, AWS, GitHub, Slack, PEM private keys). Bypass a false
  positive with `git commit --no-verify`.
- **`post-commit`** — self-healing auto-push. Because this repo is cloned in
  several places (standalone + three submodules), a plain push from a clone that
  is behind origin is rejected and leaves you stranded. This hook instead
  fetches, rebases your commit onto the remote tip, and pushes — so committing
  from any clone keeps every consumer in sync automatically. On a real conflict
  it aborts cleanly and tells you how to resolve; your commit is never lost.

> Previously the auto-push lived as an untracked `.git/hooks/post-commit` that
> existed on some clones but not others (and did a blind, divergence-prone
> push). It is now versioned here so every clone behaves identically.

## Updating proto consumers

After proto changes land on `origin/main`, bump the submodule pointer in each
consumer so it picks up the new contracts:

```
cd <consumer-repo>/nexus-proto && git fetch origin && git checkout origin/main
cd ..                          && git add nexus-proto && git commit -m "chore: bump nexus-proto"
```

(NexusCore additionally regenerates Go from the updated `.proto` files.)
