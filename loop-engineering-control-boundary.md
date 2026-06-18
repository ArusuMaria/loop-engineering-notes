# Codex Win Loop Engineering 工作台搭建说明

> 目标：在一个全新的 GitHub 仓库中，搭建一套可运行、可审计、可清理知识库的 Codex Win Loop 工作文件。本文使用通用占位，不绑定任何具体业务项目。

## 一句话原则

人定义制度，AI 在制度里运行。

AI 可以高速执行、观察、整理和局部决策，但不应该自行扩大目标、放宽边界、修改验收标准、删除审计痕迹，或决定高风险动作是否可以发生。

## Loop Engineering 的最小定义

Loop Engineering 不是写一个更长的 prompt，而是设计一个反馈系统：让 Agent 在明确目标、边界条件、知识库、工具连接、隔离工作区、复核机制和 GitHub 审计记录之间反复运行，直到满足停止条件，或升级交给人类。

最小循环如下：

```text
Trigger 启动
-> Observe 观察状态
-> Boundary 在边界内决策
-> Act 行动
-> Verify 验证反馈
-> Record 记录审计
-> Clean 清理知识库
-> Stop / Continue / Escalate 停止、继续或升级给人
```

## 必要组件

不要把 MCP、GitHub、Worktree、Skill 当成 Loop 本身。它们是 Loop 的器官。

| 组件 | 本质 | Codex Win 中的对应 |
|---|---|---|
| Trigger 启动器 | 让系统开始工作 | Automation、Hook、PR comment、CI event、手动启动 |
| Workspace 隔离 | 避免并行任务互相污染 | Worktree、branch、PR |
| Knowledge Library 知识库 | 给 Agent 外部记忆 | `AGENTS.md`、`docs/`、skills、历史 PR |
| Boundary 边界条件 | 防止 Agent 为达目标而越界 | `AGENTS.md`、done criteria、sandbox、GitHub 权限 |
| MCP 连接器 | 给 Agent 感官和手 | GitHub MCP、Context7、OpenAI Docs MCP、Browser |
| Subagents 子 Agent | 分工和互相制衡 | Worker、Reviewer、Cleaner、Explorer |
| Verification 验证器 | 判断行动是否有效 | tests、lint、build、CI、review comments、领域校验脚本 |
| Recovery 失败处理 | 防止卡死或乱跑 | 重试上限、开 issue、请求人类 |
| Audit 审计记录 | 让人醒来后能信任结果 | GitHub PR、commit、diff、review、summary |
| Stop Rule 停止条件 | 定义什么时候停 | 全部通过、超预算、冲突、不确定、需审批 |

## 人与 AI 的职责划分

| 环节 | 主要谁定 | 说明 |
|---|---|---|
| Trigger 启动 | 人定为主，AI 可建议 | 人决定什么事件值得启动 loop，例如定时检查、PR 有评论、CI 失败、用户说整理一下。AI 可以建议触发器，但不应自行扩大监控范围。 |
| Observe 观察状态 | AI 执行，人定义观察范围 | AI 读取文件、PR、CI、日志、文档和运行结果。人规定能看什么、不能看什么、优先看什么。 |
| Boundary 在边界内决策 | 人定边界，AI 在边界内判断 | 人定义红线：可以改什么、不能改什么、哪些动作需要批准。AI 只能在这些边界内做局部决策。 |
| Act 行动 | AI 执行，人批准高风险动作 | AI 写代码、改文档、跑脚本、提交 PR。删除、合并、发布、改权限、动密钥、批量重跑等动作需要人批准。 |
| Verify 验证反馈 | 人定标准，AI 执行验证 | 人定义什么叫通过，例如测试全过、lint 全过、输出文件存在、截图正常、指标低于阈值。AI 负责运行验证并解释失败。 |
| Record 记录审计 | AI 写，人定格式和位置 | AI 记录做了什么、改了什么、验证结果和剩余风险。人规定记录进入 PR、Issue、CHANGELOG、docs/status.md 或其他位置。 |
| Clean 清理知识库 | 人定知识层级，AI 执行清理 | 人定义哪些进入 `AGENTS.md`、哪些进入 `docs/`、哪些进入 memory、哪些应删除。AI 负责去重、合并、迁移、删除过期信息。 |
| Stop / Continue / Escalate 停止、继续或升级 | 人定停止/升级条件，AI 判断触发 | 人规定什么时候停、什么时候继续、什么时候升级给人。AI 根据条件执行判断。 |

## 人必须定义的内容

- 目标是什么。
- 什么算完成。
- 什么绝对不能做。
- 哪些动作需要批准。
- 失败几次必须停止。
- 成本、权限、时间和数据边界在哪里。
- 验证标准是否可以被修改，以及谁有权修改。
- 审计记录和长期知识应该放在哪里。

## AI 可以定义或决定的内容

- 先检查哪些文件、日志、PR 或状态。
- 在边界内采用什么局部执行路径。
- 如何根据错误信息调整下一步。
- 如何组织中间结果。
- 如何撰写 PR 摘要、验证摘要和风险说明。
- 如何把重复信息合并进已有文档。
- 如何发现文档、代码和记忆之间的冲突。

## 不应交给 AI 自行决定的内容

- 是否扩大任务范围。
- 是否降低验收标准。
- 是否删除测试或审计记录。
- 是否合并 PR。
- 是否发布、部署或覆盖正式数据。
- 是否修改权限、密钥、环境变量或安全策略。
- 是否把临时经验升格为长期规则。
- 是否把失败解释为可以忽略。

## 推荐文件树

在 GitHub 仓库根目录创建这些文件。下面使用 `<project-root>` 表示任意项目，不绑定具体业务类型。

```text
<project-root>/
├─ AGENTS.md
├─ docs/
│  ├─ loop.md
│  ├─ done-criteria.md
│  ├─ decisions.md
│  ├─ known-issues.md
│  ├─ status.md
│  └─ prompts/
│     └─ task-template.md
├─ .agents/
│  └─ skills/
│     ├─ clean-repo-knowledge/
│     │  └─ SKILL.md
│     └─ pr-loop/
│        └─ SKILL.md
└─ .codex/
   ├─ hooks.json
   └─ hooks/
      └─ stop-reminder.ps1
```

## 1. AGENTS.md

`AGENTS.md` 放长期规则，不放临时任务日志。它是项目边界和 review 规则的入口。

```md
# AGENTS.md

## Project Rules

- 所有改动必须通过项目定义的验证命令。
- 不允许删除测试或降低验证标准来让任务通过。
- 不允许直接合并 PR；必须让人类确认。
- 长期规则写入 AGENTS.md。
- 项目决策写入 docs/decisions.md。
- 临时状态写入 docs/status.md。
- 已知问题写入 docs/known-issues.md。

## Loop Boundaries

- 每个自动修复任务最多尝试 3 轮。
- 如果同一验证连续失败 2 次，必须停止并请求人类。
- 如果要修改权限、密钥、CI 配置或删除文件，必须先请求人类。
- 如果上下文不足，不允许编造需求；应在 PR 中提出问题。

## Review Guidelines

- 检查实现和 docs 是否一致。
- 检查是否引入无来源假设。
- 检查是否绕过验证条件。
- 检查是否需要更新 docs/decisions.md。
```

## 2. docs/loop.md

这个文件说明本项目的 loop 如何工作。

```md
# Loop Design

## Trigger

- 手动：用户在 Codex Win 中启动线程。
- 定时：Codex Automation 定期检查打开的 PR 或状态文件。
- 事件：GitHub PR comment、CI failure、@codex review。
- 生命周期：Codex Stop hook 提醒清理知识库。

## Observe

Agent 每轮必须观察：

- 当前 PR diff
- GitHub review comments
- CI 状态
- AGENTS.md
- docs/done-criteria.md
- docs/decisions.md
- docs/known-issues.md

## Act

允许的行动：

- 修改实现文件
- 修改 docs
- 提交 commit
- 回复 PR comment
- 生成验证报告

禁止的行动：

- 直接 merge PR
- 删除测试
- 降低 done criteria
- 修改密钥、权限、token
- 在失败原因不明时继续大范围改动

## Verify

每轮行动后必须运行项目定义的验证命令，例如：

```powershell
<verification-command>
```

如果项目有多级检查，也按顺序运行：

```powershell
<format-or-lint-command>
<test-command>
<build-or-export-command>
<domain-specific-validation-command>
```

## Stop

满足任一条件则停止：

- 验证全部通过，review comments 已处理。
- 连续 2 次验证失败。
- 需要权限或高风险修改。
- 需求不明确。
- PR diff 已经超过人类可轻松 review 的范围。
```

## 3. docs/done-criteria.md

这个文件定义做完的机器可验证标准。

```md
# Done Criteria

每个 PR 必须满足：

- 项目定义的验证命令通过。
- 主要用户路径或核心输出可被检查。
- 新增功能必须有对应说明。
- 修改项目结构时必须更新 docs/decisions.md。
- 不允许删除测试或降低检查标准来通过验证。
- 如果某项无法自动验证，必须在 PR summary 中说明人工验收方式。
```

## 4. docs/decisions.md

这个文件只记录会影响未来工作的长期决策。

```md
# Decisions

## YYYY-MM-DD

- 记录一个会影响未来实现、测试、部署、文档或使用方式的长期决策。
- 说明为什么选择当前方案，以及放弃了哪些替代方案。
- 不记录一次性过程日志；过程日志属于 PR、commit 或 docs/status.md。
```

## 5. docs/known-issues.md

这个文件记录暂时不修但未来需要知道的问题。

```md
# Known Issues

- <issue-id-or-title>：问题描述、影响范围、暂不处理原因、后续触发条件。
- <issue-id-or-title>：如果需要人工判断，写清楚需要谁确认什么。
```

## 6. docs/status.md

这个文件记录当前状态，允许频繁变化。

```md
# Status

## Current

- 当前正在推进：<task-or-pr>
- 当前阻塞：<blocker-or-none>
- 当前验证状态：<verification-status>

## Next

- <next-step-1>
- <next-step-2>
- <next-step-3>
```

## 7. docs/prompts/task-template.md

这个文件把一次任务写成可重复启动的格式。

```md
# Task Template

## Objective

要达成什么？

## Trigger

这次任务由什么启动？手动、定时、PR 评论、CI 失败，还是 hook？

## Boundary

不能做什么？哪些文件不能碰？最多尝试几轮？

## State To Read

需要读取哪些状态？

## Actions Allowed

允许执行哪些动作？

## Verification

必须运行哪些检查？

## Recovery

失败怎么办？什么时候停？

## Human Review

人类需要看哪些结果？
```

## 8. .agents/skills/clean-repo-knowledge/SKILL.md

这是洁癖 skill。它不负责写功能，而是每轮结束时清理知识库，防止外部大脑变脏。

```md
---
name: clean-repo-knowledge
description: Clean project knowledge after a task, update durable docs, remove stale assumptions, and prepare PR summaries.
---

# Clean Repo Knowledge

Use this skill after a task finishes or before preparing a PR.

## Steps

1. Read:
   - AGENTS.md
   - docs/loop.md
   - docs/done-criteria.md
   - docs/decisions.md
   - docs/known-issues.md
   - docs/status.md

2. Decide what kind of knowledge was produced:
   - Durable rule -> update AGENTS.md
   - Project decision -> update docs/decisions.md
   - Temporary state -> update docs/status.md
   - Known future problem -> update docs/known-issues.md
   - One-time task log -> do not preserve unless needed for PR summary

3. Remove or mark stale assumptions.

4. Do not expand AGENTS.md with task history.

5. Produce a PR summary:
   - What changed
   - Why it changed
   - Verification run
   - Docs updated
   - Remaining risk
   - Human review needed
```

调用方式：

```text
$clean-repo-knowledge 清理本次 PR 的知识体系，并准备 PR 描述。
```

## 9. .agents/skills/pr-loop/SKILL.md

这个 skill 用来处理 GitHub PR 评论和 CI。

```md
---
name: pr-loop
description: Handle a GitHub PR loop by reading comments and CI status, making bounded changes, verifying them, and preparing a reviewable update.
---

# PR Loop

Use this skill when asked to process a GitHub PR, review comments, or CI failures.

## Required Inputs

- Repository
- PR number
- Allowed scope
- Verification command

## Loop

1. Read the PR diff.
2. Read review comments.
3. Read CI status.
4. Read AGENTS.md and docs/done-criteria.md.
5. Identify the smallest safe change.
6. Make the change in a branch or worktree.
7. Run verification.
8. Update docs if durable knowledge changed.
9. Commit the change.
10. Summarize:
    - comments addressed
    - verification results
    - remaining risks

## Boundaries

- Do not merge the PR.
- Do not delete tests.
- Do not lower verification standards.
- Stop after 3 unsuccessful attempts.
- Ask the user before changing permissions, secrets, CI configuration, or large architecture.
```

调用方式：

```text
$pr-loop 处理 <owner>/<repo> 的 PR #<number>，范围只限 <allowed-scope>。
```

## 10. .codex/hooks.json

Hook 不应该一开始就做太多。最小用途是提醒每轮结束时清理知识库。

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "powershell.exe -NoProfile -ExecutionPolicy Bypass -File .codex/hooks/stop-reminder.ps1",
            "statusMessage": "Checking knowledge hygiene"
          }
        ]
      }
    ]
  }
}
```

## 11. .codex/hooks/stop-reminder.ps1

```powershell
Write-Output "Before closing: check docs/status.md, docs/decisions.md, and PR summary. Run clean-repo-knowledge when needed."
exit 0
```

## 12. MCP 安装清单

如果 `codex` 命令无法直接运行，先找到真实 CLI，例如：

```powershell
$codex = "<path-to-codex.exe>"
& $codex --version
```

安装 GitHub CLI：

```powershell
winget install --id GitHub.cli
gh auth login
```

安装 OpenAI Docs MCP：

```powershell
& $codex mcp add openaiDeveloperDocs --url https://developers.openai.com/mcp
```

安装 Context7 MCP：

```powershell
& $codex mcp add context7 -- npx -y @upstash/context7-mcp
```

安装 GitHub MCP：

```powershell
[Environment]::SetEnvironmentVariable("GITHUB_PAT_TOKEN", "<your-github-pat>", "User")
& $codex mcp add github --url https://api.githubcopilot.com/mcp/ --bearer-token-env-var GITHUB_PAT_TOKEN
```

验证：

```powershell
& $codex mcp list
```

## 13. Codex Win 中的最小运行流程

1. 在 GitHub 创建一个新的工作仓库。
2. 用 Codex Win app 打开本地仓库。
3. 创建上面的工作文件。
4. 新建一个 Worktree 线程。
5. 输入任务：

```text
使用 $pr-loop 的结构，在 worktree 中实现 <task-name>。

目标：
- <machine-checkable-goal-1>
- <machine-checkable-goal-2>
- <verification-command> 通过。
- 更新 docs/decisions.md 或说明为什么不用更新。

边界：
- 不允许直接 merge。
- 不允许删除测试或降低验证标准。
- 最多尝试 3 轮。
- 如果需求不明确，停止并提问。
```

6. Codex 完成后创建 branch 和 PR。
7. 在 GitHub PR 中 review diff。
8. 如果有评论，让 Codex 读取 PR comment 并继续修。
9. 修完后调用：

```text
$clean-repo-knowledge 清理知识库并准备 PR 总结。
```

10. 人类确认后合并。

## 14. Automation 示例

等手动跑通两次后，再创建 Codex Automation：

```text
定期检查 <repo-name> 中打开的 PR。

每次启动时：
1. 使用 GitHub MCP 读取 PR 评论和 CI 状态。
2. 如果有新的 review comment，整理待办。
3. 如果 CI 失败，定位原因并尝试最小修复。
4. 修复后运行验证。
5. 调用 clean-repo-knowledge 检查 docs 是否需要更新。
6. 不要 merge PR。
7. 如果连续 2 次失败或需要权限修改，停止并报告。
8. 如果没有问题，简短报告无待处理事项。
```

## 15. 最小 Loop 模板

```md
# Loop Spec: <名称>

## Goal
<这个 loop 要达成什么>

## Trigger
<什么事件启动它>

## Observe
<AI 可以读取哪些状态>

## Boundary
<允许做什么，禁止做什么，哪些动作需要批准>

## Act
<AI 可以执行的动作清单>

## Verify
<机器可检查的完成条件>

## Record
<记录输出到哪里，格式是什么>

## Clean
<哪些知识需要进入 docs / AGENTS.md / memory，哪些需要删除或合并>

## Stop
<满足什么条件就停止>

## Continue
<什么情况下继续下一轮>

## Escalate
<什么情况下必须交给人>
```

## 16. 示例：PR 评论处理 Loop

```md
# Loop Spec: PR Review Comment Handler

## Goal
处理打开的 GitHub PR 中新增 review comment，并把修复提交回同一个 PR。

## Trigger
- 人手动说检查 PR 评论。
- 或 automation 定期运行。

## Observe
- GitHub PR 标题、描述、diff。
- 新增 review comments。
- CI 状态。
- 仓库中的 AGENTS.md 和 docs/done-criteria.md。

## Boundary
- 可以修改当前 PR 分支。
- 可以更新测试和文档。
- 不允许删除失败测试来让 CI 通过。
- 不允许合并 PR。
- 不允许改密钥、权限、部署配置。
- 如果评论要求改变产品方向，必须升级给人。

## Act
- 读取评论。
- 把评论整理成待办。
- 修改实现或文档。
- 运行验证。
- 提交到同一 PR 分支。

## Verify
- 相关测试通过。
- lint、build 或项目等价验证通过。
- 每条 review comment 都有处理说明。
- 未处理项明确列入 PR comment。

## Record
- 在 PR comment 中写：
  - 已处理评论。
  - 修改摘要。
  - 验证结果。
  - 剩余风险。

## Clean
- 如果产生稳定项目规则，更新 AGENTS.md。
- 如果产生使用说明，更新 docs。
- 不把一次性过程记录写进 AGENTS.md。

## Stop
- 所有评论已处理，验证通过。

## Continue
- CI 失败但错误明确且在权限范围内。

## Escalate
- 连续 2 次修复后 CI 仍失败。
- 需要改部署、密钥、权限、账单或合并策略。
- 评论涉及需求方向或验收标准变化。
```

## 17. 示例：知识库清理 Loop

```md
# Loop Spec: Knowledge Neat-Freak

## Goal
让项目知识库保持干净、准确、可交接，避免 AGENTS.md、README.md、docs 和 memory 互相冲突或膨胀。

## Trigger
- 用户说整理一下、同步一下、收尾。
- 一个阶段性任务完成。
- PR 合并前。

## Observe
- AGENTS.md 或 CLAUDE.md。
- README.md。
- docs/。
- memory 或历史记录。
- 本次 diff 和验证结果。

## Boundary
- 可以更新文档和知识库。
- 不允许把历史流水账塞进 AGENTS.md。
- 不允许删除仍然有效的规则。
- 遇到无法判断的新旧冲突时升级给人。

## Act
- 查找重复、过期、冲突和相对时间。
- 把稳定知识迁移到 docs。
- 把规则保留在 AGENTS.md。
- 删除或压缩过期记录。

## Verify
- AGENTS.md 只保留规则、红线、命令速查和文档索引。
- docs 里有面向新人的完整说明。
- 没有明显过期路径、命令或相对时间。
- 关键索引没有断链。

## Record
- 输出文档变更摘要。
- 说明新增、修改、删除和未处理冲突。

## Clean
- 合并重复条目。
- 删除已毕业到 docs 的临时 memory。
- 保持 docs 厚、memory 薄、AGENTS.md 精。

## Stop
- 文档、规则和记忆一致。

## Continue
- 发现可自动修正的重复、过期或断链。

## Escalate
- 两条长期规则互相矛盾。
- 无法判断旧决策是否仍然有效。
- 需要删除可能仍有价值的历史记录。
```

## 18. 判断一个 Loop 是否合格

一个合格 loop 至少回答这些问题：

- 它怎么启动？
- 它启动后看什么状态？
- 它的目标是什么？
- 它不能做什么？
- 它能调用哪些工具？
- 它在哪个隔离区行动？
- 谁负责执行，谁负责检查？
- 怎么验证？
- 失败几次后停？
- 哪些信息沉淀进知识库？
- 哪些信息只留在 PR 日志？
- 人类在哪里 review？

如果这些问题答不出来，它还不是 loop，只是一个任务提示词。

## 19. 设计检查清单

- [ ] 完成标准是否可以被机器验证？
- [ ] 禁止事项是否和完成标准写在一起？
- [ ] AI 是否不能自行修改验收标准？
- [ ] 高风险动作是否需要人批准？
- [ ] 连续失败后是否会停止，而不是无限循环？
- [ ] 审计记录是否能让人复盘发生了什么？
- [ ] 长期知识是否有明确归宿？
- [ ] 临时过程是否不会污染长期规则？
- [ ] GitHub PR 是否能承担审计账本？
- [ ] MCP 权限是否符合最小够用原则？

## 20. 最终心智模型

把 Codex Win 当成控制台，把 GitHub 当成状态账本，把 MCP 当成外部感官和手，把 skills 当成可复用工作流，把 hooks 和 automations 当成启动器，把 subagents 当成小团队，把 `AGENTS.md` 和 `docs/` 当成外部大脑。

真正的目标不是让 Agent 一次回答得更好，而是让整个系统每一轮都更接近目标，同时不污染知识库、不越过边界、不丢失审计记录。

Loop Engineering 的关键不是让 AI 自己决定一切，而是把怎么判断它做得对提前写成系统。

好的 loop 不是更长的 prompt，而是更清晰的制度：目标、边界、验证、记录、清理、停止条件。
