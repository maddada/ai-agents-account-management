# ai-agents-account-management

<img width="500" alt="2026-02-12_CleanShot_00-38-32" src="https://github.com/user-attachments/assets/dea78275-d21b-46b1-98f6-0677d00d7529" />

<img width="500" alt="2026-02-12_Code_00-24-53@2x" src="https://github.com/user-attachments/assets/9133437e-e98a-42fa-8a7b-67671615b00b" />

Summary: Multi-account profile setup is complete for both Claude Code and Codex CLI.

Directories created:

~/.claude-profiles/personal/ — your current xyzt70@gmail.com account
~/.claude-profiles/work/ — new account (empty .claude.json, will prompt for OAuth on first run)
~/.codex-profiles/personal/ — your current ChatGPT account (auth.json copied)
~/.codex-profiles/work/ — new account (no auth.json, will prompt for login)

Shared resources moved to ~/.agents/ and symlinked into all profiles:

~/.agents/hooks/ — notification scripts
~/.agents/commands/ — auto-fix.md, create-handoff.md
~/.agents/skills/ — already existed

My aliases:
c => Claude Code with personal account
cs => Claude Code with work account
cx => Codex with personal account
cxs => Codex with work account
ai-personal / ai-work Switch both tools env at once

Each profile has its own isolated keychain entry (Claude Code) and auth file (Codex), so there are zero conflicts between accounts.
