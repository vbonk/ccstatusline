# ccstatusline

Project-specific instructions and state.
---

## CRITICAL: Git Remote Safety

**origin = vbonk/ccstatusline (USER'S REPO)**
**upstream = sirmalloc/ccstatusline (THIRD PARTY - READ ONLY)**

ALWAYS verify `git remote -v` and `gh repo view --json nameWithOwner` before any write operation.
NEVER push, close issues, or comment on non-vbonk repos.

## SESSION CONTINUITY STATE (Auto-Updated)

**Last Updated:** 2026-03-16T21:55:00Z
**Phase:** Status line restored, scripts created, remotes fixed
**Task List:** Not configured

### Session Summary (2026-03-16)
- Rebuilt dist/ccstatusline.js (old build routed stdout→stderr)
- Switched settings.json statusLine to ccstatusline.sh wrapper
- Created 9 custom-command scripts in ~/.claude/scripts/
- Fixed Stop hook: prompt→command (gh issues check)
- Removed broken SubagentStop prompt hook
- Fixed git remotes: origin→vbonk, upstream→sirmalloc
- Set gh repo default to vbonk/ccstatusline
- Wrote security lessons to global knowledge, project memory, MCP memory

### Current Blockers
None

### Next Actions
- **PRIORITY: Run DEVMAC_SETUP.md on DevMac** — open session in ~/projects/ccstatusline, read .claude/DEVMAC_SETUP.md, execute all steps
- Address active upstream bugs: #241 (underscores), #239 (effort), #238 (model name), #221 (multi-line truncation)

### Recovery Checklist
1. Verify `git remote -v` shows origin=vbonk
2. Run `gh repo view --json nameWithOwner` — must show vbonk/ccstatusline
3. Run `TaskList()` to see current tasks
4. Check memory: `feedback_vbonk_repos.md` for security rules

---
