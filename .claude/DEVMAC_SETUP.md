# DevMac Setup Instructions

## Status: PENDING — Run this from a Claude Code session on DevMac

These instructions were generated on 2026-03-16 from a Halbert session that fixed ccstatusline and all supporting scripts.

---

## Problem

ccstatusline disappeared from status line on both Halbert and DevMac. Root cause: old `dist/ccstatusline.js` build was routing stdout→stderr. Claude Code only reads stdout.

## Fix Steps for DevMac

### 1. Rebuild ccstatusline

```bash
cd ~/projects/ccstatusline
git pull origin main   # Get latest from vbonk/ccstatusline
bun run build          # Rebuilds dist/ccstatusline.js — fixes stderr bug
```

### 2. Create wrapper script

Create `/Users/tony/.claude/ccstatusline.sh`:

```bash
#!/bin/bash
exec node /Users/tony/projects/ccstatusline/dist/ccstatusline.js
```

Then make executable:

```bash
chmod +x /Users/tony/.claude/ccstatusline.sh
```

### 3. Create stop-check-issues.sh

Create `/Users/tony/.claude/scripts/stop-check-issues.sh`:

```bash
#!/bin/bash
# Stop hook: check for in-progress GitHub issues
in_progress=$(gh issue list --label "status:in-progress" --state open --json number,title --limit 5 2>/dev/null)
if [ -z "$in_progress" ] || [ "$in_progress" = "[]" ]; then
  echo '{"ok": true}'
  exit 0
fi
title=$(echo "$in_progress" | python3 -c "import json,sys; issues=json.load(sys.stdin); print(issues[0]['title'] if issues else '')" 2>/dev/null)
count=$(echo "$in_progress" | python3 -c "import json,sys; print(len(json.load(sys.stdin)))" 2>/dev/null)
if [ -n "$title" ] && [ "$count" -gt 0 ]; then
  reason="$count issue(s) still in-progress: \"$title\""
  reason=$(echo "$reason" | python3 -c "import json,sys; print(json.dumps(sys.stdin.read().strip()))" 2>/dev/null)
  echo "{\"ok\": false, \"reason\": $reason}"
else
  echo '{"ok": true}'
fi
```

Then: `chmod +x /Users/tony/.claude/scripts/stop-check-issues.sh`

### 4. Replace ~/.claude/settings.json

Replace the ENTIRE contents of `/Users/tony/.claude/settings.json` with:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  },
  "permissions": {
    "defaultMode": "bypassPermissions"
  },
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "/Users/tony/.claude/scripts/session-restore.sh"
          },
          {
            "type": "command",
            "command": "/Users/tony/.claude/hooks/session-start.sh"
          }
        ]
      },
      {
        "matcher": "compact",
        "hooks": [
          {
            "type": "command",
            "command": "echo '⚠️  POST-COMPACTION: Check in_progress task for compaction_checkpoint metadata. Run TaskGet() to recover context.'"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "/Users/tony/.claude/hooks/validate-bash-safety.sh"
          }
        ]
      },
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "/Users/tony/.claude/hooks/require-read-first.sh"
          },
          {
            "type": "command",
            "command": "/Users/tony/.claude/hooks/validate-halbert-contracts.sh"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Read",
        "hooks": [
          {
            "type": "command",
            "command": "/Users/tony/.claude/hooks/track-read.sh"
          }
        ]
      },
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "/Users/tony/.claude/hooks/validate-edit.sh"
          }
        ]
      }
    ],
    "PreCompact": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "/Users/tony/.claude/scripts/pre-compact-save.sh"
          },
          {
            "type": "command",
            "command": "/Users/tony/.claude/scripts/update-project-state.sh"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "/Users/tony/.claude/scripts/stop-check-issues.sh"
          }
        ]
      }
    ]
  },
  "statusLine": {
    "type": "command",
    "command": "/Users/tony/.claude/ccstatusline.sh",
    "padding": 0
  },
  "enabledPlugins": {
    "code-review@claude-plugins-official": false,
    "commit-commands@claude-plugins-official": true,
    "context7@claude-plugins-official": true,
    "feature-dev@claude-plugins-official": false,
    "frontend-design@claude-plugins-official": true,
    "greptile@claude-plugins-official": false,
    "hookify@claude-plugins-official": true,
    "linear@claude-plugins-official": false,
    "playwright@claude-plugins-official": false,
    "pr-review-toolkit@claude-plugins-official": false,
    "supabase@claude-plugins-official": false,
    "vercel@claude-plugins-official": false,
    "agent-sdk-dev@claude-plugins-official": false,
    "security-guidance@claude-plugins-official": true,
    "plugin-dev@claude-plugins-official": false,
    "slack@claude-plugins-official": false,
    "document-skills@anthropic-agent-skills": false,
    "example-skills@anthropic-agent-skills": true,
    "typescript-lsp@claude-plugins-official": true,
    "stripe@claude-plugins-official": false,
    "github@claude-plugins-official": false,
    "sentry@claude-plugins-official": false,
    "ralph-loop@claude-plugins-official": true
  },
  "alwaysThinkingEnabled": true,
  "effortLevel": "high",
  "skipDangerousModePermissionPrompt": true
}
```

### 5. Create custom-command scripts (optional)

These scripts power the status line widgets. Create them at `/Users/tony/.claude/scripts/`:

- `weekly-reset-timer.sh` — reads from `~/.cache/ccstatusline/usage.json`
- `cc-version-check.sh` — shows Claude Code version
- `cc-mode.sh` — shows permission mode
- `gh-issues-count.sh` — counts open issues for current repo
- `task-count.sh` — counts pending tasks
- `gh-ci-status.sh` — shows CI status
- `gh-repo-visibility.sh` — shows repo visibility
- `docker-health.sh` — shows docker container count
- `gh-repo-name.sh` — shows owner/repo name

All scripts are in the Halbert repo at `/home/claude-runner/.claude/scripts/` — copy and adjust paths from `/home/claude-runner/` to `/Users/tony/`.

### 6. Verify

```bash
echo '{"model":{"id":"claude-opus-4-6[1m]"}}' | /Users/tony/.claude/ccstatusline.sh
```

Should produce colored output on **stdout** (not stderr). If output goes to stderr, rebuild with `bun run build`.

---

## What Was Fixed on Halbert (2026-03-16)

1. Rebuilt `dist/ccstatusline.js` — old build routed stdout→stderr
2. Switched statusLine to `ccstatusline.sh` wrapper with `padding: 0`
3. Created 9 custom-command scripts in `~/.claude/scripts/`
4. Fixed Stop hook: `prompt` type → `command` type (gh issues check)
5. Removed broken SubagentStop prompt hook
6. Fixed git remotes: origin=vbonk, upstream=sirmalloc
7. Set `gh repo set-default vbonk/ccstatusline`
8. Added `effortLevel: "high"` to settings
9. Updated README, CONTRIBUTING, package.json, typedoc.json to reference vbonk
10. Updated App.tsx GITHUB_REPO_URL to vbonk

## Git Remote Safety

**CRITICAL**: User's GitHub is `vbonk`. ALL repos under `github.com/vbonk/`.
`sirmalloc/ccstatusline` is a THIRD-PARTY upstream — NEVER write to it.

If `git remote -v` shows `origin` pointing to sirmalloc:
```bash
git remote rename origin upstream
git remote set-url origin https://github.com/vbonk/ccstatusline.git
git branch --set-upstream-to=origin/main main
gh repo set-default vbonk/ccstatusline
```
