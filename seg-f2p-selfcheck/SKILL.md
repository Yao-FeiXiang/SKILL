---
name: seg-f2p-selfcheck
description: Use when validating Long-Horizon-Bench seg tasks that fail strict fail-to-pass (F2P) in validate_per_pr — including Dockerfile/stack drift, native test runner vs test file mismatch, missing build or test deps, or tests incompatible with the harness — and you need a repeatable loop until strict F2P passes or the seg is skipped.
---

# Seg 任务 F2P 自检流程（Long-Horizon-Bench，跨仓库通用）

## Overview
目标：让 `scripts/validate_per_pr.py` 在 **`--strict-fail-to-pass`、尽量不 `--stop-on-fail`** 的前提下跑完目标任务，用 JSON/日志做聚类，再形成可复现的 **修复环境 / 修复 runner / 修复或移除用例 / 跳过 seg** 决策。

核心原则：
1. **先核对「执行契约」再改镜像**：很多失败是 `test_unit_runner.py` 的 `LANG` 与真实测试文件扩展名不一致，或 `Dockerfile` 未安装上游跑测试所需的命令（`bundle`、`make`、完整 `npm` 依赖等），而不是补丁本身。
2. **先全量跑再分类**：用完整 JSON 看失败阶段（build / container / before_tests / patch / after_tests）分布，再批量处理。
3. **修复有上限**：若「合理可修」项超过阈值（默认 20），标记该 seg 跳过，只保留已明确的通用环境修复（避免无限拖尾）。

## When to Use
- 任意 `tasks/task_<repo>_segXX` 在 strict F2P 下失败：构建失败、容器起不来、before/after 测试异常、`fail_after_not_passing` 等。
- 怀疑 **镜像与 base 不对齐**、**runner 与测试类型不对齐**、或 **用例与 validate 执行模型不兼容**。

---

## Step 0：执行契约盘点（必做，先于 Dockerfile 大改）

### 0.1 三条线必须一致
| 线索 | 去哪看 | 你要确认什么 |
|------|--------|----------------|
| 校验脚本里的语言标签 | `scripts/validate_per_pr.py` 的 `REPO_LANG` + `extract_repo_id(task_name)` | 该 repo 在全局表里被归到哪一类（python / js / c / ruby / …） |
| 任务内 runner | `tasks/.../tests/test_unit_runner.py` 里的 `REPO_ID`、`LANG` | `LANG` 会走 `_run_test_command` 的哪条分支（`python` / `js` / `c` / `ruby` / …） |
| 真实测试文件 | `tasks/.../tests/<PR或unit_id>/...` 下扩展名 | 与上一步分支是否匹配 |

**静默假成功（最危险）**：若 `LANG == "python"` 但目录里只有 `.rb` / `.c` / `.js`，runner 里常见逻辑是「没有匹配扩展名 → 返回 0」，**测试根本没跑**却表现为通过。修 F2P 前必须先排除这种情况。

**分支缺失**：若 `LANG` 取值为 runner 里未实现的分支（例如只有 `c` 没有 `cpp`），可能落到 `unknown language` 之类命令，**同样可能 exit 0**。应对：把 `LANG` 改成已有分支，或扩展 `_run_test_command` 使用上游真实命令。

### 0.2 上游「官方」测试入口
在 `base/` 内快速查：**Makefile**（`make test`）、**package.json**（`npm test`）、**Rakefile / bin/test**、**README / CONTRIBUTING**。Dockerfile 里应能支撑这条入口（不仅是「把源码 COPY 进去」）。

---

## Step 1：按技术栈对齐 Dockerfile（通用模式，而非仅 pip）

**总目标**：镜像里能 **构建** base（若需要），且能执行 Step 0 确定的 **原生测试命令**。避免 `install ... || true` 吞错（除非有注释说明的极少数镜像修复）。

### 1.1 Python / Python+C 扩展（例：django、ansible、scikit-learn、**validate 表中的 pytorch**）
- **运行时版本**：`base` 的 `requires-python` / `python_requires` / `.python-version` 等。
- **测试依赖**：项目自带的 constraints / `requirements-dev.txt` / `tox.ini` / `units.txt` 等 — **优先用上游锁文件安装**，不要只手动 `pip install pytest`。
- **编译依赖**：`build-essential`、`cmake`、头文件包、`NO_CUDA` 等与上游构建脚本一致。

**Ansible 特化（仍适用）**：`test/lib/ansible_test/_data/requirements/constraints.txt` + `units.txt` + `test/units/requirements.txt` 是典型「三件套」；其它 Python 项目找等价物即可。

### 1.2 Node / NodeBB（`REPO_LANG` 多为 `js`）
- 镜像年代（旧 LTS）可能导致 **Debian 源失效**，需 `archive.debian.org` 等修复（任务里常有示例）。
- **`npm install` vs `npm ci`**、是否需 `build-essential` 编原生插件。
- Runner：`LANG` 应为 **`js`** 若用例是 `npm test` / Jest 等；若任务误标 `python`，先回到 Step 0。

### 1.3 C / OpenSSL（`REPO_LANG`：`c`）
- 典型：`ubuntu` + `build-essential` + `clang`/`cmake` 等；可能还需 **Perl** 等（看 `base` 测试脚本）。
- Runner：`LANG` 应为 **`c`** 且命令应与 OpenSSL 的 **`make test`** 或 `test/run_tests.pl` 等一致；**不要**在只有 `.c` 用例时仍用 `LANG=python` 走 pytest 路径。

### 1.4 Ruby / Rails（`REPO_LANG`：`ruby`）
- 镜像 **Ruby 版本** 与 base 一致或兼容。
- Dockerfile 常缺 **`bundle install`** / 系统库（如 **sqlite3**、**libsqlite3-dev**）——仅 `COPY base` 不足以跑 `rails test`。
- Runner：`LANG` 应为 **`ruby`**，用例为 `*_test.rb` 时与 `bundle exec rails test` 路径一致。

---

## Step 2：完整 strict F2P 验证（建议不中断）

```bash
python3 scripts/validate_per_pr.py \
  --task-dir tasks/task_<repo>_segXX \
  --strict-fail-to-pass \
  --json-out tmp/<repo>-segXX-full.json
```

从 JSON 关注：`overall_ok`、`summary.strict_fail`、`failure_stage`、各 PR 的首段 stderr/stdout（聚类用）。

---

## Step 3：阈值决策（>20 可修项 → 跳过 seg）

将失败粗分为 **环境/runner 可修** vs **用例与 harness 不兼容 / 与 gold 错配**。若前者累计需改文件数 **> 20**（或单次全量失败 PR 数过多），倾向 **跳过该 seg**，避免拖尾。

---

## Step 4：失败分类（通用）

### 4.1 优先修「环境与契约」
- 构建失败、缺包、错 Python/Ruby/Node 版本、缺系统库。
- **`test_unit_runner.py` 的 `LANG` 与扩展名 / `REPO_LANG` 不一致**。
- `run-tests.sh` 里对 pytest 的 `|| true` 掩盖安装失败 → 改为显式失败或固定安装方式。

### 4.2 移除或替换用例（与执行模型冲突）
- 需要 **完整 CI harness**、数据库、网络、或仓库专属集成环境，而 validate 只在容器里做 **有限次原生调用**。
- **no tests ran**、收集不到用例、或 INTERNALERROR 因选中了非测试文件。
- **测试与 `gold_patches/<PR>.diff` 无交集**：补丁改 A，失败在 B → after 永远不会按预期过。

### 4.3 修单个用例（小改动）
- 新语言版本下 **mock 签名**、API 行为变化（与 Ansible seg 中 `os.stat` 类问题同类）。
- **fixture 仅在 base 真实存在**时可从 base 补齐到 `tests/<PR>/...`；若 base 无此数据，优先移除用例而非「编假 fixture」。

---

## Step 5：对「应修复」项逐 PR 深潜

1. `requirements/.rename_map.json`、`slug_diff_map.json`
2. `requirements/<slug>.yaml`、`gold_patches/<PR>.diff`、`tests/<PR>/...`
3. 确认补丁是否触及失败断言；无交集 → 转 **4.2 移除**
4. 最小改动后回到 Step 2 全量复跑

---

## Common Mistakes（跨栈）
- **只改 Dockerfile 不查 runner**：`LANG` 与文件扩展名不一致导致假绿。
- **`|| true` 吞构建/安装错误**：失败推迟到测试阶段，难聚类。
- **把 Ansible 的 constraints/units 当成唯一范式**：其它栈应对照 **锁文件 + 官方测试文档**。
- **stop-on-fail 过早**：丢失全量分布，批量处理效率低。
- **为通过而编造 upstream 不存在的 fixture**：偏离真实仓库，优先移除用例。

---

## 参考：四个 seg01 任务的对比要点（泛化用）

| 任务 | 镜像/构建要点 | runner 注意 |
|------|----------------|-------------|
| NodeBB | 旧 Node 镜像 + Debian 源修复；`npm install` | `REPO_LANG` 为 `js`；用例若为 JS 测，勿用 `LANG=python` |
| OpenSSL | Ubuntu + 编译链；测试多为 `.c` | `REPO_LANG` 为 `c`；需 `make test`（或等价），勿对 `.c` 走 pytest |
| PyTorch | 老 Python + `setup.py` + 大量编译开关 | validate 中 pytorch 归 **python**；runner 的 `LANG` 须能执行 **实际复制的 `.py` 测试**（避免未实现分支） |
| Rails | Ruby 镜像 + **bundle / 系统库** | `REPO_LANG` 为 **ruby**；`.rb` 用例需 `LANG=ruby` + 可用 `bundle exec rails test` |

（具体路径以仓库内 `Dockerfile` / `test_unit_runner.py` 为准；上表用于提醒 **栈与契约** 维度，而非背诵命令。）
