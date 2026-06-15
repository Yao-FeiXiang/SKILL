# Codex Skills Backup

This repository backs up manually installed Codex skills from:

```text
C:\Users\12845\.codex\skills
```

Codex-managed built-in skills under `.system` are intentionally excluded
because they are restored with Codex itself.

## Layout

Each directory at the repository root is one restore-ready Codex skill. The
original directory structure is preserved, including each skill's `SKILL.md`,
references, scripts, templates, and assets.

Repository documentation is stored under `docs/` and is not a Codex skill.

## Restore

Clone the repository and copy the required skill directories into:

```text
%USERPROFILE%\.codex\skills
```

Do not copy `docs`, `.git`, `.gitignore`, or `README.md` into the skills
directory. Restart Codex after restoring skills.

## Update

Mirror the current top-level skill directories from the source path into this
repository, excluding `.system`. Review the Git diff for deleted, added, and
modified files before committing and pushing the update.
