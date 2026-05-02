# NexusProto

Shared protobuf contracts for NexusControl.

## Docs

- [Site Template API](docs/site-template-api.md)

## Secret-leak prevention

This repo ships a pre-commit hook that scans staged changes for credentials. Run this once after cloning:

```
git config core.hooksPath .githooks
```

The hook blocks committed `.env` / `*.key` / `*.pem` / `credentials.json` files and common token formats (NexusControl `nsk_`, AWS, GitHub, Slack, PEM private keys). If you hit a false positive, you can bypass with `git commit --no-verify`.
