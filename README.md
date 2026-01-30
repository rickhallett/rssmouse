# rssmouse 🐭

Minimal RSS/Atom feed watcher with Clawdbot integration.

## Features

- **Minimal footprint**: Single Python script, stdlib only (no pip dependencies)
- **CRUD for feeds**: Add, list, update, remove RSS/Atom feed URLs
- **Deduplication**: SQLite-backed tracking of seen items (by GUID/link)
- **Clawdbot integration**: Spawns a subagent to deliver digests without interrupting the main session
- **Flexible operation**: Manual checks, cron-triggered, or daemon mode

## Installation

Already installed at `~/clawd/bin/rssmouse` with symlink at `~/.local/bin/rssmouse`.

```bash
# The tool is ready to use:
rssmouse --help
```

## Usage

### Add feeds

```bash
rssmouse add "https://github.com/hyprwm/Hyprland/releases.atom" --name "Hyprland"
rssmouse add "https://archlinux.org/feeds/news/" --name "Arch News"
```

### List feeds

```bash
rssmouse list
```

### Check for new items

```bash
rssmouse check           # Check all feeds, notify if new items
rssmouse check --quiet   # Silent mode (for cron)
```

### Watch continuously (daemon mode)

```bash
rssmouse watch --interval 300  # Check every 5 minutes
```

### Manage seen items

```bash
rssmouse seen 1     # List seen items for feed #1
rssmouse clear 1    # Clear seen items (will re-notify on next check)
```

### Remove a feed

```bash
rssmouse remove 1   # Remove feed #1
```

### Test notification

```bash
rssmouse notify "Test message from rssmouse"
```

## Cron Setup

For periodic checks without a daemon:

```bash
# Edit crontab
crontab -e

# Add: Check every 15 minutes
*/15 * * * * /home/mrkai/clawd/bin/rssmouse check --quiet 2>&1 | logger -t rssmouse
```

## Data Storage

- **Config/Data**: `~/.local/share/rssmouse/rssmouse.db` (SQLite)
- **Environment variables**:
  - `CLAWDBOT_GATEWAY_PORT`: Gateway port (default: 18789)
  - `CLAWDBOT_GATEWAY_TOKEN`: Auth token for Clawdbot API

## Clawdbot Integration

When new items are detected, rssmouse calls the Clawdbot `/tools/invoke` endpoint to spawn a subagent (`sessions_spawn`). The subagent:

1. Receives the digest message
2. Processes and acknowledges it
3. Announces back to the requester channel (webchat, WhatsApp, etc.)

This ensures the main session isn't interrupted, and notifications arrive naturally in your active channel.

## Feed Support

- **Atom**: GitHub releases, standard Atom feeds
- **RSS 2.0**: Most blog/news feeds
- Automatic detection based on XML structure

## Example Workflow

```bash
# 1. Add your feeds
rssmouse add "https://github.com/hyprwm/Hyprland/releases.atom" -n "Hyprland"
rssmouse add "https://archlinux.org/feeds/news/" -n "Arch Linux"

# 2. Initial check marks existing items as "seen"
rssmouse check

# 3. Set up periodic checks (cron or watch)
rssmouse watch --interval 600  # or add to crontab

# 4. When new items appear, you get a Clawdbot notification!
```

## Technical Notes

### Language Choice

Python was chosen over Go/Rust/shell for:
- Zero external dependencies (urllib, json, sqlite3 are all stdlib)
- Easy iteration and debugging
- Sufficient for the workload (periodic HTTP fetches)
- ~500 lines of readable, maintainable code

### Deduplication Strategy

Items are tracked by their GUID (or link as fallback, or SHA256 of title as last resort). This handles:
- Standard GUIDs in RSS/Atom
- GitHub's tag-based IDs
- Malformed feeds without proper identifiers

### Notification Workflow

Uses Clawdbot's `sessions_spawn` via the HTTP tools invoke API:
- Non-blocking: returns immediately with `runId` and `childSessionKey`
- Subagent processes the digest asynchronously
- Announce step posts results back to the requester channel

## License

MIT - part of the Clawdbot ecosystem.
