# Codex Skills Backup Design

## Goal

Back up manually installed Codex skills from `C:\Users\12845\.codex\skills` to
`git@github.com:Yao-FeiXiang/SKILL.git` in a form that is readable, versioned,
and easy to restore.

## Scope

- Copy every top-level skill directory under `C:\Users\12845\.codex\skills`.
- Exclude `.system`, because Codex installs and maintains those built-in skills.
- Preserve each skill's directory structure, including references, templates,
  scripts, and assets.
- Add a repository-level `README.md` and `.gitignore`.

## Repository Layout

Each backed-up skill is stored directly at the repository root using its
existing directory name. This keeps restore operations simple: selected skill
directories can be copied back into `%USERPROFILE%\.codex\skills`.

## Safety

- Do not copy files outside the skills directory.
- Scan filenames and content for likely credentials before committing.
- Exclude transient files such as Python caches, editor metadata, and OS files.
- Abort the push if Git detects files larger than GitHub's 100 MB limit.

## Update Strategy

Future backups mirror the current non-system skill directories into this
repository, review the diff, then create and push a new commit. Git history
records additions, updates, and removals over time.

## Verification

- The source and repository skill-directory lists must match, excluding
  `.system` and repository metadata.
- The committed tree must contain no likely credential files or oversized
  files.
- The final commit must exist on the remote default branch.
