---
name: claude-code-review
description: Use when Codex has changed code, is asked to get Claude Code's opinion, or needs a second-agent review before reporting implementation work as complete.
---

# Claude Code Review

Use Claude Code as a read-only second reviewer. Do not ask Claude to edit files.

## Commands

From a git work tree:

```bash
~/.codex/bin/claude-review --base main
```

For risky changes:

```bash
~/.codex/bin/claude-review --base main --adversarial --prompt "Focus on rollback risk, data loss, races, hidden assumptions, and missing tests."
```

For working-tree-only changes when no useful base exists:

```bash
~/.codex/bin/claude-review
```

## Triage

- Treat Claude findings as review input, not truth.
- Verify each material finding against the code before changing anything.
- Fix only issues that are relevant to the task or clearly dangerous.
- Re-run project tests after fixes.
- Mention the review command and saved `~/.codex/cross-agent-review/` record in the final summary.

## Avoid

- Do not let Claude modify files during this review.
- Do not run two agents editing the same files in one work tree.
- Do not paste secrets, tokens, or private logs into external chat bridges.
