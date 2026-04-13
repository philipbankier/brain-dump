# brain-dump → brain-dump Rename Checklist

**Generated:** 2026-04-13
**Total occurrences:** 142 strings + 3 file/dir names

---

## Files/Directories to Rename

| Old | New | Type |
|-----|-----|------|
| `./brain-dump` | `brain-dump` | Main CLI script |
| `./launchd/com.brain-dump.plist.template` | `com.brain-dump.plist.template` | Launchd template |
| `./skill/brain-dump/` | `skill/brain-dump/` | Agent skill directory |

---

## Content Replacements (142 total)

| File | Count | Notes |
|------|-------|-------|
| `brain-dump` | 58 | Main CLI — self-references, help text, errors |
| `README.md` | 48 | Docs, examples, URLs |
| `install.sh` | 15 | Paths, messages |
| `skill/brain-dump/SKILL.md` | 9 | Skill description |
| `launchd/com.brain-dump.plist.template` | 4 | Label, paths |
| `lib/config.sh` | 3 | Comments, paths |
| `lib/presets.sh` | 2 | Comments |
| `lib/output.sh` | 1 | Comments |
| `skill/brain-dump/scripts/status.sh` | 1 | CLI call |
| `skill/brain-dump/scripts/snapshot.sh` | 1 | CLI call |

---

## External References (Manual Action Required)

| Location | Status | Action |
|----------|--------|--------|
| GitHub repo name | `philipbankier/brain-dump` | Rename to `philipbankier/brain-dump` |
| Installed launchd plist | `~/Library/LaunchAgents/com.brain-dump.plist` | Unload, remove, reinstall with new name |
| B2 bucket | `vic-brain-dump` | Optional: can stay, data is unchanged |
| Config file path | `~/.config/brain-dump/` | **NOT changing** — existing configs remain valid |
| Skill directory (installed) | `~/.local/share/brain-dump/` | **NOT changing** — existing installs remain valid |

---

## Proposed Execution Order

1. **Content replacements** (single sed pass)
   ```bash
   find . -type f -not -path './.git/*' -exec sed -i '' 's/brain-dump/brain-dump/g' {} +
   ```

2. **File/Directory renames**
   ```bash
   mv brain-dump brain-dump
   mv launchd/com.brain-dump.plist.template launchd/com.brain-dump.plist.template
   mv skill/brain-dump skill/brain-dump
   ```

3. **Verify no references remain**
   ```bash
   grep -r "brain-dump" . --exclude-dir=.git
   # Should return nothing
   ```

4. **Test CLI still works**
   ```bash
   ./brain-dump --help
   ./brain-dump status
   ```

5. **Commit changes**
   ```bash
   git add -A
   git commit -m "refactor: rename brain-dump → brain-dump"
   ```

6. **Manual external actions** (after commit):
   - Rename GitHub repo
   - Reload launchd with new plist
   - Update any local shortcuts/aliases

---

## Config/Backward Compatibility Notes

- **Config directory**: Keeping `~/.config/brain-dump/` unchanged — existing configs work
- **B2 bucket**: Keeping `vic-brain-dump` unchanged — existing backups remain accessible
- **Install dir**: Keeping `~/.local/share/brain-dump/` unchanged for v0.1 installs
- **Launchd label**: Changing `com.brain-dump` → `com.brain-dump` (requires reinstall)
