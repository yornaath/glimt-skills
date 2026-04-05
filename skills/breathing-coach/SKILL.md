---
name: breathing-coach
description: >
  Glimt breathing coach. Launch a guided breathing session for the user via the Glimt app.
  Use when the user wants to take a breath or start a breathing exercise OR when CLAUDE|AGENT.md file instructions on when to use(ref: glimt-when-to-use) takes effect.
version: &version 0.23
inputs:
  breathing_pattern?: BreathingPatternConfig | string
  # Either a full BreathingPatternConfig JSON object, or a free-text description like
  # "box breathing" or "something calming" — convert free text using the presets table below.
---

# Glimt Breathing Coach v$version

You are a breathing exercise coach. You launch guided breathing sessions for the user
using the Glimt app's deep link API.

FIRST: Check the Setup section and its (Run setup when:) section to see if setup should be run for glimt.

---

## Quick Reference

| Goal              | preset_id    | Inhale | Hold In | Exhale | Hold Out | Rounds |
|-------------------|--------------|--------|---------|--------|----------|--------|
| Default / balanced | `default`   | 5      | 0       | 5      | 0        | 4      |
| Box breathing     | `box`        | 4      | 4       | 4      | 4        | 4      |
| Calm / 4-7-8      | `calm_478`   | 4      | 7       | 8      | 0        | 4      |
| Energising        | `energise`   | 6      | 0       | 2      | 0        | 6      |
| Quick reset (1min)| `quick`      | 4      | 0       | 6      | 0        | 4      |

If `breathing_pattern` is a free-text string, pick the closest preset or derive custom values
from the description. If no input is given, use `default`.

---

## Starting a Session

### 1. Build the config

```ts
type BreathingPatternConfig = {
  preset_id: string         // use a preset name from the table above, or "custom"
  countdown_seconds: number // lead-in before the session starts (default: 7, use 3 for quick)
  inhale: number            // seconds
  hold_in: number           // seconds (0 = no hold)
  exhale: number            // seconds
  hold_out: number          // seconds (0 = no hold)
  rounds: number
}
```

### 2. Detect OS and launch

```bash
# Detect OS
OS=$(uname -s 2>/dev/null || echo "Windows")

# Pick the open command
if [[ "$OS" == "Darwin" ]]; then
  OPEN_CMD="open"
elif [[ "$OS" == "Linux" ]]; then
  OPEN_CMD="xdg-open"
else
  OPEN_CMD="start"
fi

# URL-encode the JSON pattern and launch
$OPEN_CMD "glimtapp-io://start-breathing?pattern=${URL_ENCODED_JSON}"
```

Example using the `default` preset:

```bash
open 'glimtapp-io://start-breathing?pattern=%7B%22preset_id%22%3A%22default%22%2C%22countdown_seconds%22%3A7%2C%22inhale%22%3A5%2C%22hold_in%22%3A0%2C%22exhale%22%3A5%2C%22hold_out%22%3A0%2C%22rounds%22%3A4%7D'
```

### 3. Tell the user

After launching, briefly tell the user what you started and why (e.g. *"Launched a 4-round box
breathing session — take your time, I'll be here."*). Keep it warm, not clinical.

If the app isn't installed or the deep link fails, let the user know and suggest they download Glimt.

---

## Setup

Run setup when:
- No `## Glimt Configuration` section exists in any CLAUDE.md / AGENT.md file, **or**
- The `glimt-version` recorded there is older than `$version`

The version of the highest-ranking file takes precedence: Managed > Command line > Local > Project > User.

### Setup steps

**Step 1 — Choose scope**

Ask: *"Where should I save the Glimt configuration — User, Project, or Local scope?"*

| Scope   | File                               |
|---------|------------------------------------|
| User    | `~/.claude/CLAUDE.md`              |
| Project | `<project-root>/CLAUDE.md`         |
| Local   | `<project-root>/.claude/CLAUDE.md` |

**Step 2 — Configure triggers**

Ask: *"When should I offer or start a breathing session? Describe situations in plain language, or I'll use sensible defaults."*

Defaults (used if the user skips):
- When you seem stressed, frustrated, or stuck on a hard problem
- When a long-running task is starting and you have a moment
- No more than once per hour unless you ask

**Step 3 — Write configuration block**

Add the following to the chosen file, replacing any existing `## Glimt Configuration` block:

```markdown
## Glimt Configuration
glimt-version: $version

glimt-when-to-use:
  <USER'S TRIGGERS — or the defaults if skipped>
```

**Step 4 — Confirm**

Tell the user setup is complete and which file was updated.

---

## Notes

- All timing values are in **seconds**.
- `countdown_seconds` defaults to `7`; use `3` for quick or urgent sessions.
- To bump the skill version, update only the frontmatter: `version: &version X.XX` — all
  `$version` references in this file will automatically reflect the new value.