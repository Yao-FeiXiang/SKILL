# Codex Skills Backup

This repository backs up manually installed Codex skills from:

```text
C:\Users\12845\.codex\skills
```

Codex-managed built-in skills under `.system` are intentionally excluded
because they are restored with Codex itself.

## Layout

Most directories at the repository root are restore-ready Codex skills. The
original directory structure is preserved, including each skill's `SKILL.md`,
references, scripts, templates, and assets.

`auto-claude-code-research-in-sleep/` is a vendored ARIS repository rather
than a single skill. Its Codex-native skills live under
`auto-claude-code-research-in-sleep/skills/skills-codex/`; keeping the complete
repository also preserves the helper tools, templates, MCP servers, and update
scripts those skills use.

Repository documentation is stored under `docs/` and is not a Codex skill.

## Restore

Clone the repository and copy the required skill directories into:

```text
%USERPROFILE%\.codex\skills
```

Do not copy `docs`, `.git`, `.gitignore`, or `README.md` into the skills
directory. Restart Codex after restoring skills.

For ARIS, install or link each directory under
`auto-claude-code-research-in-sleep/skills/skills-codex/` into
`%USERPROFILE%\.codex\skills`. Set `%USERPROFILE%\.aris\repo` to the absolute
path of `auto-claude-code-research-in-sleep` so ARIS skills can resolve their
shared helper tools.

## Update

Mirror the current top-level skill directories from the source path into this
repository, excluding `.system`. Review the Git diff for deleted, added, and
modified files before committing and pushing the update.
