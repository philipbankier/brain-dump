# 🧠 brain-backup

Opinionated, encrypted backup for AI agent memory.

Wraps [restic](https://restic.net/) with sane defaults for backing up AI agent
memory directories to S3-compatible storage (Backblaze B2, AWS S3, Cloudflare R2).

## Install

```bash
curl -fsSL https://raw.githubusercontent.com/philipbankier/brain-backup/main/install.sh | bash
```

## Quick Start

```bash
brain-backup init        # Configure backend + profiles
brain-backup snapshot    # Take your first backup
brain-backup status      # Check health
```

## Supported Agents

OpenClaw · Hermes · Claude Code · Codex · Windsurf

## Status

🚧 Under active development. Not yet released.

## License

MIT
