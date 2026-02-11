# Multi-Account Profile Setup for Claude Code & Codex CLI

## Context

You want to run 2 different accounts for both Claude Code and Codex CLI without manually swapping config files. Neither tool has built-in multi-account support, but both expose an environment variable that redirects their entire config directory:

- **Claude Code**: `CLAUDE_CONFIG_DIR` (default: `~/.claude`). When set, it also moves where `.claude.json` is read from (`$CLAUDE_CONFIG_DIR/.claude.json` instead of `~/.claude.json`), and creates a **separate macOS Keychain entry** with a hash suffix — so OAuth tokens are fully isolated per profile.
- **Codex CLI**: `CODEX_HOME` (default: `~/.codex`). Redirects everything including `auth.json` (file-based OAuth tokens).

The plan: create a profile directory for each account, copy your current settings into both, authenticate each separately, and add shell functions to switch between them instantly.

---

## Step 1: Create Profile Directories

```bash
mkdir -p ~/.claude-profiles/personal
mkdir -p ~/.claude-profiles/work
mkdir -p ~/.codex-profiles/personal
mkdir -p ~/.codex-profiles/work
```

## Step 2: Move Shared Resources to a Central Location

Your hooks and commands should live in one place and be symlinked into each profile, so edits apply everywhere. Skills are already shared via `~/.agents/skills/`.

```bash
mkdir -p ~/.agents/hooks
mkdir -p ~/.agents/commands

# Copy hooks and commands to shared location
cp ~/.claude/hooks/notify-local-tts.sh ~/.agents/hooks/
cp ~/.claude/commands/auto-fix.md ~/.agents/commands/
cp ~/.claude/commands/create-handoff.md ~/.agents/commands/
```

## Step 3: Populate Claude Code Profiles

### Profile A (personal — your current xyzt70@gmail.com account)

```bash
# Copy config files
cp ~/.claude.json ~/.claude-profiles/personal/.claude.json
cp ~/.claude/settings.json ~/.claude-profiles/personal/settings.json
cp ~/.claude/settings.local.json ~/.claude-profiles/personal/settings.local.json
cp ~/.claude/CLAUDE.md ~/.claude-profiles/personal/CLAUDE.md

# Symlink shared resources
ln -s ~/.agents/skills ~/.claude-profiles/personal/skills
ln -s ~/.agents/hooks ~/.claude-profiles/personal/hooks
ln -s ~/.agents/commands ~/.claude-profiles/personal/commands
```

### Profile B (work — new account)

```bash
# Copy settings (identical starting point)
cp ~/.claude-profiles/personal/settings.json ~/.claude-profiles/work/settings.json
cp ~/.claude-profiles/personal/settings.local.json ~/.claude-profiles/work/settings.local.json
cp ~/.claude-profiles/personal/CLAUDE.md ~/.claude-profiles/work/CLAUDE.md

# Empty .claude.json — Claude Code will prompt for login on first run
echo '{}' > ~/.claude-profiles/work/.claude.json

# Symlink shared resources
ln -s ~/.agents/skills ~/.claude-profiles/work/skills
ln -s ~/.agents/hooks ~/.claude-profiles/work/hooks
ln -s ~/.agents/commands ~/.claude-profiles/work/commands
```

### Fix hook paths in both profiles' settings.json

The current `settings.json` references `/Users/madda/.claude/hooks/notify-local-tts.sh`. Update both profiles to use the shared path:

**File**: `~/.claude-profiles/personal/settings.json` and `~/.claude-profiles/work/settings.json`

**Change**: `/Users/madda/.claude/hooks/notify-local-tts.sh` → `/Users/madda/.agents/hooks/notify-local-tts.sh`

## Step 4: Populate Codex CLI Profiles

### Profile A (personal — current ChatGPT account)

```bash
cp ~/.codex/config.toml ~/.codex-profiles/personal/config.toml
cp ~/.codex/auth.json ~/.codex-profiles/personal/auth.json
cp ~/.codex/AGENTS.md ~/.codex-profiles/personal/AGENTS.md
cp -R ~/.codex/rules ~/.codex-profiles/personal/rules
ln -s ~/.agents/skills ~/.codex-profiles/personal/skills
```

### Profile B (work — new account)

```bash
cp ~/.codex/config.toml ~/.codex-profiles/work/config.toml
cp ~/.codex/AGENTS.md ~/.codex-profiles/work/AGENTS.md
cp -R ~/.codex/rules ~/.codex-profiles/work/rules
ln -s ~/.agents/skills ~/.codex-profiles/work/skills
# NO auth.json copy — Codex will prompt for login on first run
```

## Step 5: Add Profile Switching to ~/.zshrc

Append the following to `~/.zshrc`:

```bash
# ============================================================
# Multi-Account Profiles for Claude Code & Codex CLI
# ============================================================

CLAUDE_PROFILE_DIR="$HOME/.claude-profiles"
CODEX_PROFILE_DIR="$HOME/.codex-profiles"

# Default profile
: "${CLAUDE_PROFILE:=personal}"
: "${CODEX_PROFILE:=personal}"

claude-profile() {
  local profile="${1}"
  if [[ -z "$profile" ]]; then
    echo "Active: $CLAUDE_PROFILE"
    echo "Available: $(ls -1 "$CLAUDE_PROFILE_DIR" 2>/dev/null | tr '\n' ' ')"
    return 0
  fi
  [[ ! -d "$CLAUDE_PROFILE_DIR/$profile" ]] && echo "Not found: $profile" && return 1
  export CLAUDE_CONFIG_DIR="$CLAUDE_PROFILE_DIR/$profile"
  export CLAUDE_PROFILE="$profile"
  echo "Claude Code → $profile"
}

codex-profile() {
  local profile="${1}"
  if [[ -z "$profile" ]]; then
    echo "Active: $CODEX_PROFILE"
    echo "Available: $(ls -1 "$CODEX_PROFILE_DIR" 2>/dev/null | tr '\n' ' ')"
    return 0
  fi
  [[ ! -d "$CODEX_PROFILE_DIR/$profile" ]] && echo "Not found: $profile" && return 1
  export CODEX_HOME="$CODEX_PROFILE_DIR/$profile"
  export CODEX_PROFILE="$profile"
  echo "Codex CLI → $profile"
}

ai-profile() {
  local profile="${1}"
  if [[ -z "$profile" ]]; then
    echo "Claude: $CLAUDE_PROFILE | Codex: $CODEX_PROFILE"
    return 0
  fi
  claude-profile "$profile"
  codex-profile "$profile"
}

alias ai-personal='ai-profile personal'
alias ai-work='ai-profile work'

# --- Short Aliases ---
# c / cx  = open Claude / Codex with PERSONAL profile (default)
# cs / cxs = open Claude / Codex with WORK profile
alias c='claude-profile personal > /dev/null && claude'
alias cx='codex-profile personal > /dev/null && codex'
alias cs='claude-profile work > /dev/null && claude'
alias cxs='codex-profile work > /dev/null && codex'

# Apply defaults on shell startup
claude-profile "$CLAUDE_PROFILE" > /dev/null
codex-profile "$CODEX_PROFILE" > /dev/null
```

## Step 6: Authenticate Each Profile

```bash
# Reload shell
source ~/.zshrc

# --- Claude Code ---
ai-personal
claude          # Should work immediately (existing account)
# Verify: type /config — should show xyzt70@gmail.com

ai-work
claude          # Will open browser for OAuth — sign in with the NEW account
# A new keychain entry is created automatically (isolated by hash)

# --- Codex CLI ---
ai-personal
codex login status   # Should show logged in (copied auth.json)

ai-work
codex login          # Will open browser — sign in with the NEW ChatGPT account
```

## Step 7: Verify

```bash
# Check profiles switch correctly
ai-personal && echo $CLAUDE_CONFIG_DIR && echo $CODEX_HOME
# → ~/.claude-profiles/personal and ~/.codex-profiles/personal

ai-work && echo $CLAUDE_CONFIG_DIR && echo $CODEX_HOME
# → ~/.claude-profiles/work and ~/.codex-profiles/work

# Check keychain has two separate Claude entries
security dump-keychain 2>&1 | grep "Claude Code"
# Should show: "Claude Code-credentials" AND "Claude Code-credentials-<hash>"

# Check shared resources are linked
ls -la ~/.claude-profiles/personal/skills  # → ~/.agents/skills
ls -la ~/.claude-profiles/work/skills      # → ~/.agents/skills
```

---

## Daily Usage

| Command | Effect |
|---------|--------|
| `c` | Launch Claude Code with **personal** account |
| `cx` | Launch Codex CLI with **personal** account |
| `cs` | Launch Claude Code with **work** account |
| `cxs` | Launch Codex CLI with **work** account |
| `ai-personal` | Switch shell env to personal (then run `claude`/`codex` normally) |
| `ai-work` | Switch shell env to work (then run `claude`/`codex` normally) |
| `ai-profile` | Show which profiles are active |
| `claude-profile work` | Switch only Claude Code's profile |
| `codex-profile personal` | Switch only Codex's profile |

The `c`/`cs`/`cx`/`cxs` aliases are fire-and-forget — they set the profile and launch the tool in one shot. The `ai-*` and `*-profile` commands just switch the environment so subsequent `claude`/`codex` calls use that profile.

---

## Gotchas to Know About

1. **Original `~/.claude/` and `~/.claude.json` become orphaned** — once you always have `CLAUDE_CONFIG_DIR` set, the defaults are never read. Keep them as backups.
2. **IDE integration** — If you use Claude Code in VS Code, add `CLAUDE_CONFIG_DIR` to your VS Code terminal env settings so the IDE terminal picks up the right profile.
3. **Non-interactive shells** — If you run scripts or use tmux that don't source `.zshrc`, consider also adding the exports to `~/.zshenv`.
4. **Project-level `.claude/` dirs** are unaffected — `CLAUDE_CONFIG_DIR` only changes the user-level config, not project-level settings.
5. **Settings drift** — After initial setup, the two profiles are independent. Editing settings in one doesn't affect the other. The shared symlinks (skills, hooks, commands) keep the important stuff in sync.

---

## Optional: Per-Project Auto-Switching with direnv

If certain repos should always use a specific account:

```bash
brew install direnv
echo 'eval "$(direnv hook zsh)"' >> ~/.zshrc

# In a work project:
cd ~/dev/work-project
echo 'export CLAUDE_CONFIG_DIR="$HOME/.claude-profiles/work"
export CODEX_HOME="$HOME/.codex-profiles/work"' > .envrc
direnv allow
```

Now `cd`-ing into that directory auto-switches the profile.
