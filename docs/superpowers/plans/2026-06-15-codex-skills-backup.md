# Codex Skills Backup Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Mirror all manually installed Codex skills to the GitHub repository while excluding Codex-managed system skills.

**Architecture:** Use the repository root as the restore-ready skills directory. Copy each top-level directory from `%USERPROFILE%\.codex\skills` except `.system`, add repository documentation and ignore rules, then validate the source/destination directory sets and scan the staged content before pushing.

**Tech Stack:** PowerShell, Git, GitHub over SSH

---

### Task 1: Add Repository Metadata

**Files:**
- Create: `.gitignore`
- Create: `README.md`

- [ ] **Step 1: Add transient-file exclusions**

Create `.gitignore` entries for Python caches, editor directories, temporary files, and OS metadata without excluding any skill source files.

- [ ] **Step 2: Document backup and restore behavior**

Create `README.md` describing the source path, `.system` exclusion, repository layout, restore command, and update workflow.

- [ ] **Step 3: Review metadata**

Run: `git diff -- .gitignore README.md`

Expected: only backup documentation and transient-file exclusions are shown.

### Task 2: Mirror Skills

**Files:**
- Create or update: `<repository-root>/<skill-name>/**`

- [ ] **Step 1: Copy non-system skill directories**

Enumerate top-level directories under `C:\Users\12845\.codex\skills`, exclude `.system`, and recursively copy each remaining directory into the repository root.

- [ ] **Step 2: Compare directory sets**

Compare source and destination top-level skill names after excluding `.system`, `.git`, and `docs`.

Expected: no missing or unexpected skill directories.

### Task 3: Safety Review and Commit

**Files:**
- Stage: all repository backup files

- [ ] **Step 1: Scan likely credential filenames and content**

Search for private-key headers, common token assignments, GitHub tokens, AWS keys, and files named like credentials or environment secrets.

Expected: no findings requiring exclusion.

- [ ] **Step 2: Check GitHub file-size limit**

List files larger than 100 MB.

Expected: no files are returned.

- [ ] **Step 3: Stage and inspect changes**

Run: `git add -A` followed by `git status --short` and `git diff --cached --stat`.

Expected: skill directories, metadata, and plan documentation only.

- [ ] **Step 4: Commit backup**

Run: `git commit -m "backup: add installed Codex skills"`.

Expected: commit succeeds with the mirrored skill files.

### Task 4: Push and Verify

**Files:**
- No local file changes expected

- [ ] **Step 1: Push default branch**

Run: `git push -u origin main`.

Expected: GitHub accepts the branch and sets upstream tracking.

- [ ] **Step 2: Verify local and remote commit identity**

Compare `git rev-parse HEAD` with `git ls-remote origin refs/heads/main`.

Expected: both commit hashes are identical.

- [ ] **Step 3: Verify clean worktree**

Run: `git status --short`.

Expected: no output.
