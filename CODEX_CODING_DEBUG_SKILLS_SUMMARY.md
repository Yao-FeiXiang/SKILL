# Codex 写代码 / Debug Skills 汇总

用途：把本地 Codex 中适合“写代码、修 Bug、做代码审查、验证交付”的 skills 上传到服务器 Codex，并配置到 `~/.codex/skills`。

## 推荐安装清单

### 必装：开发与 Debug 主流程

| Skill 目录 | 触发场景 | 作用 |
|---|---|---|
| `superpowers-using-superpowers` | 每个会话开始 | 规定 Codex 先检查并使用匹配 skill，避免跳过既定流程。 |
| `superpowers-brainstorming` | 做功能、改行为、加组件前 | 先澄清需求和设计，得到用户确认后再实现。 |
| `superpowers-systematic-debugging` | 遇到 bug、测试失败、异常行为 | 强制先定位根因，再修复，避免凭猜测打补丁。 |
| `superpowers-test-driven-development` | 实现新功能或修 bug | 先写失败测试，再写最小实现，再重构。 |
| `superpowers-verification-before-completion` | 准备声明完成、修好、测试通过前 | 必须重新跑验证命令，用证据支撑结论。 |

### 建议安装：计划、执行、隔离与收尾

| Skill 目录 | 触发场景 | 作用 |
|---|---|---|
| `superpowers-writing-plans` | 多步骤需求、复杂改动前 | 生成可执行实现计划，拆分文件、测试和验证步骤。 |
| `superpowers-executing-plans` | 已有书面计划，需要按计划实现 | 读取计划、批判性检查、逐项执行并验证。 |
| `superpowers-using-git-worktrees` | 需要隔离开发环境或执行计划前 | 创建独立 worktree，减少污染当前工作区。 |
| `superpowers-finishing-a-development-branch` | 实现完成且测试通过后 | 指导 merge、PR、清理分支等收尾动作。 |
| `simplify` | 代码、架构、文档或计划过度复杂 | 在不改变行为和接口的前提下降低复杂度。 |

### 建议安装：代码审查与反馈处理

| Skill 目录 | 触发场景 | 作用 |
|---|---|---|
| `superpowers-requesting-code-review` | 完成任务、重大功能、合并前 | 调用独立 reviewer 检查需求符合度和代码质量。 |
| `superpowers-receiving-code-review` | 收到 review 意见后 | 先验证意见是否正确，再决定是否实现，避免盲改。 |
| `claude-code-review` | Codex 改完代码后想让 Claude Code 二次审查 | 通过 `~/.codex/bin/claude-review --base main` 做只读二审。服务器需要另配 Claude CLI/脚本才可用。 |

### 可选：并行与专项场景

| Skill 目录 | 触发场景 | 作用 |
|---|---|---|
| `superpowers-dispatching-parallel-agents` | 有 2 个以上互不依赖的问题或任务 | 把独立问题分发给子 agent 并行调查。 |
| `superpowers-subagent-driven-development` | 有实现计划，且任务可并行 | 每个任务用 fresh subagent，并在每项后做双阶段 review。 |
| `seg-f2p-selfcheck` | Long-Horizon-Bench seg 任务 strict F2P 失败 | 检查 Dockerfile、runner、测试依赖、用例兼容性，循环到 F2P 通过或跳过。 |

## 最小推荐组合

如果服务器只想保留一套轻量但完整的写代码/Debug 工作流，建议安装这些目录：

```text
superpowers-using-superpowers
superpowers-brainstorming
superpowers-systematic-debugging
superpowers-test-driven-development
superpowers-verification-before-completion
superpowers-writing-plans
superpowers-executing-plans
superpowers-using-git-worktrees
superpowers-requesting-code-review
superpowers-receiving-code-review
simplify
```

## 完整推荐组合

如果服务器上的 Codex 支持子 agent，建议安装完整开发组：

```text
superpowers-using-superpowers
superpowers-brainstorming
superpowers-systematic-debugging
superpowers-test-driven-development
superpowers-verification-before-completion
superpowers-writing-plans
superpowers-executing-plans
superpowers-using-git-worktrees
superpowers-finishing-a-development-branch
superpowers-requesting-code-review
superpowers-receiving-code-review
superpowers-dispatching-parallel-agents
superpowers-subagent-driven-development
simplify
claude-code-review
seg-f2p-selfcheck
```

## 上传到服务器

当前仓库 `A:\GitHub\SKILL` 已经包含这些 skill 目录。打包时要复制整个目录，不要只复制 `SKILL.md`，因为部分 skill 可能带 `references/`、`scripts/`、`assets/`。

### Windows 本地打包

在 `A:\GitHub\SKILL` 执行：

```powershell
$skills = @(
  "superpowers-using-superpowers",
  "superpowers-brainstorming",
  "superpowers-systematic-debugging",
  "superpowers-test-driven-development",
  "superpowers-verification-before-completion",
  "superpowers-writing-plans",
  "superpowers-executing-plans",
  "superpowers-using-git-worktrees",
  "superpowers-finishing-a-development-branch",
  "superpowers-requesting-code-review",
  "superpowers-receiving-code-review",
  "superpowers-dispatching-parallel-agents",
  "superpowers-subagent-driven-development",
  "simplify",
  "claude-code-review",
  "seg-f2p-selfcheck"
)
Compress-Archive -Path $skills -DestinationPath .\codex-coding-debug-skills.zip -Force
```

### 上传

```powershell
scp .\codex-coding-debug-skills.zip user@server:/tmp/
```

### 服务器解压

```bash
mkdir -p ~/.codex/skills
unzip -o /tmp/codex-coding-debug-skills.zip -d ~/.codex/skills
find ~/.codex/skills -maxdepth 2 -name SKILL.md | sort
```

## 配置建议

1. 服务器 Codex 的 skill 根目录通常放在 `~/.codex/skills`。
2. 解压后每个 skill 应该是 `~/.codex/skills/<skill-name>/SKILL.md`。
3. 如果 `claude-code-review` 要可用，服务器还需要有 `~/.codex/bin/claude-review` 和 Claude Code 相关配置；没有的话可以先不装或装了但不要触发。
4. 如果服务器不跑 Long-Horizon-Bench，可以不装 `seg-f2p-selfcheck`。
5. 如果服务器 Codex 不支持 subagent，`superpowers-dispatching-parallel-agents` 和 `superpowers-subagent-driven-development` 的收益会下降，但保留也没坏处。

