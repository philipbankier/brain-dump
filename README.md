# 🧠 brain-backup

Opinionated, encrypted backup for AI agent memory.

Wraps [restic](https://restic.net/) with sane defaults for backing up AI agent
memory directories to S3-compatible storage (Backblaze B2, AWS S3, local).

## Install

```bash
curl -fsSL https://raw.githubusercontent.com/philipbankier/brain-backup/main/install.sh | bash
```

Or manually:

```bash
git clone https://github.com/philipbankier/brain-backup.git
cd brain-backup
chmod +x brain-backup
export PATH="$PWD:$PATH"
```

## Requirements

| Dependency | Version | Install |
|-----------|---------|---------|
| bash | 3.2+ | pre-installed on macOS |
| [restic](https://restic.net/) | 0.16+ | `brew install restic` |
| [yq](https://github.com/mikefarah/yq) | 4.0+ | `brew install yq` |
| [jq](https://stedolan.github.io/jq/) | 1.6+ | `brew install jq` |

## Quick Start

```bash
# 1. Set credentials (add to ~/.zshenv or ~/.bashrc)
export RESTIC_PASSWORD="your-strong-password"
export B2_ACCOUNT_ID="your-b2-key-id"
export B2_ACCOUNT_KEY="your-b2-application-key"

# 2. Initialize
brain-backup init

# 3. Take your first backup
brain-backup snapshot

# 4. Check health
brain-backup status

# 5. Schedule hourly backups (macOS)
brain-backup schedule install
```

## Commands

### `brain-backup init`
Interactive setup. Choose backend (B2, S3, local), bucket, and agent profiles.
Creates `~/.config/brain-backup/config.yaml` and initializes the restic repo.

```bash
brain-backup init                  # Interactive
brain-backup init --backend b2     # Skip backend prompt
```

### `brain-backup snapshot`
Take an incremental backup of all configured agent memory directories.

```bash
brain-backup snapshot              # All profiles
brain-backup snapshot --profile openclaw  # Single profile
brain-backup snapshot --dry-run    # Preview without running
brain-backup snapshot --json       # Machine-readable output
```

### `brain-backup restore`
Restore files from a snapshot. **Never auto-overwrites** existing files.

```bash
brain-backup restore latest                           # Restore to ~/brain-backup-restore-<timestamp>
brain-backup restore abc123 --target ~/my-restore     # Specific snapshot + target dir
brain-backup restore latest --dry-run                 # Preview
```

### `brain-backup list`
Show backup history (only brain-backup-tagged snapshots).

```bash
brain-backup list                  # All snapshots
brain-backup list --latest 5       # Last 5
brain-backup list --json           # Machine-readable
```

### `brain-backup status`
Show backup health: repo reachable, latest snapshot age, total stored, schedule.

```bash
brain-backup status                # Human-readable
brain-backup status --json         # Machine-readable
```

### `brain-backup prune`
Apply retention policy from config (runs `restic forget --prune`).

```bash
brain-backup prune                 # Apply retention
brain-backup prune --dry-run       # Preview what would be removed
```

### `brain-backup schedule`
Manage macOS launchd scheduled backups.

```bash
brain-backup schedule install      # Install hourly launchd job
brain-backup schedule remove       # Remove launchd job
brain-backup schedule status       # Show schedule state
```

### `brain-backup doctor`
Check all dependencies, config, credentials, repo, paths, and schedule.

```bash
brain-backup doctor                # Human-readable
brain-backup doctor --json         # Machine-readable
```

### `brain-backup config`
Show resolved configuration.

```bash
brain-backup config                          # Full config
brain-backup config --path repository.backend  # Single value
```

## Supported Agent Presets

| Preset | Paths | Notes |
|--------|-------|-------|
| **openclaw** | `~/.openclaw` | Excludes browser/, media/, delivery-queue/, *.log |
| **hermes** | `~/.hermes` | Excludes logs/, sessions/ |
| **claude-code** | `~/.claude` | Excludes cache/, debug/, downloads/, CachedData/, Code Cache/ |
| **codex** | `~/.codex` | Excludes log/, tmp/ |
| **windsurf** | `~/.codeium/windsurf/memories` | Full memories backup |

## Configuration

Config file: `~/.config/brain-backup/config.yaml`

```yaml
version: 1
repository:
  backend: b2           # b2, s3, or local
  bucket: my-backup     # B2 bucket name or S3 bucket
  # endpoint: ""        # Optional: custom S3 endpoint

profiles:
  - name: my-agents
    preset: openclaw    # Use preset defaults
    paths:
      - ~/.openclaw
      - ~/my-workspace  # Add custom paths
    exclude:
      - "*.log"         # Add custom excludes
    include:
      - "important.log" # Remove from excludes

  - name: custom
    paths:
      - ~/my-data
    exclude:
      - "*.tmp"

schedule:
  interval: 3600        # Seconds between backups

retention:
  hourly: 24            # Keep 24 hourly snapshots
  daily: 30             # Keep 30 daily snapshots
  monthly: 12           # Keep 12 monthly snapshots
```

### Merge Rules

When a profile uses a preset:
- **Paths**: Profile paths **override** preset paths entirely
- **Excludes**: Profile excludes **merge** with preset excludes (additive)
- **Include**: Profile include items are **removed** from merged excludes

## Credentials

brain-backup never stores credentials. Set them as environment variables:

### Backblaze B2
```bash
export RESTIC_PASSWORD="your-repo-password"
export B2_ACCOUNT_ID="your-key-id"
export B2_ACCOUNT_KEY="your-application-key"
```

### AWS S3
```bash
export RESTIC_PASSWORD="your-repo-password"
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
```

### Local
```bash
export RESTIC_PASSWORD="your-repo-password"
```

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Config error |
| 2 | Missing dependency |
| 3 | Credentials error |
| 4 | Restic error |
| 5 | No paths to back up |

## Agent Skill

brain-backup includes an [agentskills.io](https://agentskills.io) compliant skill
at `skill/brain-backup/`. Install it in your agent's skill directory to let AI
agents trigger backups and check status.

Restore always requires user confirmation — agents never auto-overwrite.

## How It Works

brain-backup is a thin bash wrapper around restic:

1. **`snapshot`**: Resolves profiles → deduplicates paths → runs `restic backup`
   with appropriate tags and excludes → applies retention policy
2. **`restore`**: Runs `restic restore` to a timestamped temp directory (never
   overwrites existing files)
3. **`schedule`**: Generates a macOS launchd plist and loads it via `launchctl`
4. Everything else is config/validation/formatting

No daemons, no background processes, no network services. Just bash + restic.

## License

[MIT](LICENSE)
