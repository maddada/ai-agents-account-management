# Multi-account profile setup is complete for both Claude Code and Codex CLI.

<img width="500" alt="2026-02-12_Code_00-25-16@2x" src="https://github.com/user-attachments/assets/bf10f8c3-cbcd-4c13-b450-82efbfbf4844" />
<br />

### Directories created:

~/.claude-profiles/personal/ — your current xyzt70@gmail.com account<br />
~/.claude-profiles/work/ — new account (empty .claude.json, will prompt for OAuth on first run)<br />
~/.codex-profiles/personal/ — your current ChatGPT account (auth.json copied)<br />
~/.codex-profiles/work/ — new account (no auth.json, will prompt for login)<br />

Shared resources moved to ~/.agents/ and symlinked into all profiles:

~/.agents/hooks/ — notification scripts<br />
~/.agents/commands/ — auto-fix.md, create-handoff.md<br />
~/.agents/skills/ — already existed<br />

### Aliases in ~/.zshrc to launch agent clis:<br />
c => Claude Code with personal account<br />
cs => Claude Code with work account<br />
cx => Codex with personal account<br />
cxs => Codex with work account<br />
ai-personal / ai-work Switch both tools env at once

Each profile has its own isolated keychain entry (Claude Code) and auth file (Codex), so there are zero conflicts between accounts.
