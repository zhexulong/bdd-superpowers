# BDD Superpowers

[English](README.md)

BDD Superpowers 是 [Superpowers](https://github.com/obra/superpowers) 的 fork。它保留 Superpowers 原本的 skills-first 软件开发工作流，并在设计和计划链路里加入行为评估。

基础工作流仍然是 Superpowers：brainstorm 设计，写 spec，写 implementation plan，用 TDD 实现，review，verification。变化在于：非平凡行为改动现在会在纵向 implementation slice 之外，增加一条横向 behavior/control harness。

更具体地说：TDD 继续检查局部实现是否正确；Behavior Evaluation 和 Behavior Coverage 检查整条流程是否仍然在做用户真正想要的行为。这个能力在代码已经大到人不会逐行阅读、接近黑盒时尤其重要。

这不是官方 Superpowers 发行版。它是一个 Superpowers-derived fork，重点是 BDD-style 行为审查、pipeline 级约束，以及 design-plan-code 的对齐。

## Quickstart

给你的 agent 安装 BDD Superpowers：[Codex CLI](#openai-codex-cli)、[OpenCode](#opencode)、[Claude Code、Cursor、Copilot 和 Gemini](#claude-codecursorcopilot-和-gemini)、[Kimi Code](#kimi-code)、[Antigravity](#antigravity)、[Pi](#pi)。

内部 skill namespace 仍然保留为 `superpowers:*`，以兼容已有 agent 和配置。

## 它如何工作

起点和 Superpowers 一样。当你让 coding agent 做一个东西时，它不应该直接开始写代码，而是先用 brainstorming skill 理解你真正想做什么，探索方案，并把对话收敛成一份可审查的设计。

BDD Superpowers 在这一步加入 bounded behavior grill。agent 会压力测试具体例子、行为边界、failure signals、invariants 和 correction paths，但不会把会话变成几个小时的穷尽式问卷。能从代码、文档、已有约定中回答的问题，它应该自己查；只有答案会改变设计路线时才问用户。

设计写出来后，spec 可以包含 `Behavior Evaluation` 章节。这个章节不是 implementation plan，也不是方案解释。它说明哪些行为必须可观察，预期结果是什么，哪些信号说明系统偏离了，哪些 invariant 必须横跨流程成立，以及证据失败时应该回到哪里修正。

设计批准后，writing-plans skill 仍然生成 Superpowers 风格的 implementation plan：具体任务、文件路径、测试和验证步骤。如果 spec 包含 Behavior Evaluation，plan 还会增加 `Behavior Coverage`：把场景和 invariant 横向映射到 implementation tasks 和验证证据。纯技术任务仍然可以是 `technical-only`；目标不是强迫每个局部 slice 都伪造 BDD。

最后，review 不只看局部测试是否通过。BDD Superpowers 在文档审查里加入 architecture ownership 检查，并在 code review 里加入一个更短的版本。reviewer 会检查两类问题：一类是实现细节都对，但行为 pipeline 错了、不完整，或者不再绑定用户想要的结果；另一类是 convenience glue、cache、wrapper、fallback path、debug artifact、eval artifact 悄悄变成了产品 contract 或 runtime authority。

这很重要，因为很多 agent 错误不是孤立的“代码写烂了”，而是 ownership 错了：一个临时 support mechanism 开始决定 routing、truth、method、answer shape、read order 或 policy。reviewer 会追问这个机制现在实际控制了哪个更高层行为，它是否应该拥有这个权力，以及它应该被保留、削薄、移到 private/eval-only 边界后面、迁移到显式 contract/spec 层，还是直接删除。

## 相较上游 Superpowers 的优化

- **Brainstorming 增加 bounded behavior grill**：在设计定稿前做针对性压力测试，避免一次性问几十个低价值问题。
- **Spec 增加 Behavior Evaluation**：为非平凡行为改动记录 concrete examples、expected results、failure signals、invariants、observable evidence 和 correction paths。
- **Plan 增加 Behavior Coverage**：把 spec 中的行为场景映射到计划任务和验证证据，同时允许无关技术任务保持 `technical-only`。
- **Design document self-review 增强**：spec reviewer 会检查行为证据缺失、failure signal 模糊、invariant 太弱、correction path 不清楚，以及 hidden ownership、accidental architecture、support mechanism 变成 product contract 这类架构风险。
- **Plan document review 增强**：plan reviewer 会拒绝假的 per-task behavior coverage，检查横向场景是否真的贯穿到计划，并阻止 support/eval/debug 机制变成产品 contract。
- **Code review 增强**：code review 会检查 flow-level drift 和 hidden ownership，也就是局部测试通过但用户意图或 pipeline 没有被保住，或者实现胶水悄悄接管 truth、method、routing、policy ownership。

## 安装

不同平台安装方式不同。关键规则是：安装这个 fork，不要安装官方 Superpowers marketplace 包。

从上游 Superpowers 迁移时，使用“先删除旧安装，再新安装 BDD Superpowers”的流程：

先确定你要切换的是哪个 agent 的安装：Codex、OpenCode、Claude Code、Cursor、Copilot、Gemini，或者其它本地 skill/plugin 入口。很多人会同时装多个；这里只能针对你选中的那个 agent 做卸载和重装。

1. 删除旧的上游 Superpowers plugin entry、symlink、junction 或 clone。
2. 使用下面的 git 地址安装 BDD Superpowers。
3. 清理这个选定 agent 的滞后 cache。
4. 在这个选定 agent 里跑下面的 smoke-test 对话；只有平台确实需要重启才能重新发现插件时，才先重启。只要回答没有清楚说明 `Behavior Coverage` 什么时候出现、`technical-only` 是什么、以及它和 TDD 有什么区别，就当成 stale cache。

如果你想让 agent 代为迁移，可以直接给它这一句话：

```text
先问我需要替换哪个 agent 的 Superpowers 安装，然后只针对那个选定的 agent 卸载上游 Superpowers，使用 https://github.com/zhexulong/bdd-superpowers.git 安装 BDD Superpowers，清理滞后的 skill/plugin cache，帮我跑 README 里的 smoke-test 对话；只有该平台确实需要重启才能重新发现插件时才提示重启。如果 `superpowers:writing-plans` 没有清楚说明 `Behavior Coverage` 什么时候出现、`technical-only` 是什么、以及它和 TDD 有什么区别，就当成 stale cache。
```

Smoke-test prompt:

```text
Use superpowers:writing-plans. Answer only with three bullets: when does the plan include Behavior Coverage, what does technical-only mean, and how is Behavior Coverage different from TDD? If the loaded skill does not mention Behavior Coverage, say STALE SUPERPOWERS CACHE.
```

如果平台支持非交互命令，就直接跑这条最小对话：

- OpenCode：`opencode run 'Use superpowers:writing-plans. Answer only with three bullets: when does the plan include Behavior Coverage, what does technical-only mean, and how is Behavior Coverage different from TDD? If the loaded skill does not mention Behavior Coverage, say STALE SUPERPOWERS CACHE.'`
- Codex：`codex exec 'Use superpowers:writing-plans. Answer only with three bullets: when does the plan include Behavior Coverage, what does technical-only mean, and how is Behavior Coverage different from TDD? If the loaded skill does not mention Behavior Coverage, say STALE SUPERPOWERS CACHE.'`

缓存刷新说明见 [Refreshing Stale Superpowers Caches](docs/cache-refresh.md)。

使用 BDD Superpowers 仓库：

```text
https://github.com/zhexulong/bdd-superpowers.git
```

内部 skill namespace 目前仍保留为 `superpowers:*`，以兼容已有 agent 和配置。

### OpenCode

在你的 `opencode.json` 的 `plugin` 数组中加入 BDD Superpowers：

```json
{
  "plugin": ["superpowers@git+https://github.com/zhexulong/bdd-superpowers.git"]
}
```

如果需要，先重启 OpenCode。然后跑上面的 smoke test，并检查 `writing-plans` 是否说明 `Behavior Coverage` 什么时候出现、`technical-only` 是什么、以及它和 TDD 有什么区别。

如果你之前安装了官方 Superpowers，先删除旧 plugin entry，再添加这一项。不要同时启用官方 Superpowers 和 BDD Superpowers，因为它们暴露相同的 skill names。若重启后 skill 文本仍像上游版本，先按 [缓存刷新说明](docs/cache-refresh.md) 清理 stale cache。

### OpenAI Codex CLI

clone 这个 fork，并把 skills 目录 symlink 到 Codex native skill discovery：

```bash
git clone https://github.com/zhexulong/bdd-superpowers.git ~/.codex/bdd-superpowers
mkdir -p ~/.agents/skills
ln -s ~/.codex/bdd-superpowers/skills ~/.agents/skills/superpowers
```

安装后重启 Codex。更新：

```bash
cd ~/.codex/bdd-superpowers && git pull
```

如果 `~/.agents/skills/superpowers` 已经指向官方 Superpowers，替换这个 symlink，让它指向本 fork。

如果替换旧安装后，加载出来的 skill 文本仍然像上游 Superpowers，先按 [缓存刷新说明](docs/cache-refresh.md) 清理 stale cache。

### Claude Code、Cursor、Copilot 和 Gemini

官方 marketplaces 安装的是上游 Superpowers，不是这个 fork。如果你要使用 BDD Superpowers，暂时不要使用官方 marketplace entry。

平台支持 git-based install 时，使用本 fork 的 git URL；否则 clone 本仓库，并通过平台的本地 skill/plugin 机制暴露 `skills/` 目录。同一时间只保留一个 `superpowers` skill namespace provider。

### Kimi Code

如果 Kimi Code 支持 URL 安装，直接从本仓库安装：

```text
/plugins install https://github.com/zhexulong/bdd-superpowers
```

详细说明见 [docs/README.kimi.md](docs/README.kimi.md)。

### Antigravity

从本仓库安装：

```bash
agy plugin install https://github.com/zhexulong/bdd-superpowers
```

### Pi

从本仓库安装：

```bash
pi install git:github.com/zhexulong/bdd-superpowers
```

## 基础工作流

1. **brainstorming** - 写代码前触发。通过问题澄清想法，探索方案，分段展示设计，并在非平凡行为改动上运行 bounded behavior grill。

2. **Behavior Evaluation** - 写在 design/spec 中。定义 concrete examples、expected results、failure signals、invariants、observable evidence 和 correction paths。

3. **using-git-worktrees** - 设计批准后触发。创建隔离工作区，运行项目 setup，确认干净的测试基线。

4. **writing-plans** - 基于批准后的设计生成 implementation plan。任务包含文件路径、测试和验证步骤；如果 spec 有 Behavior Evaluation，则增加 Behavior Coverage。

5. **subagent-driven-development** 或 **executing-plans** - 基于 plan 执行。按任务派发 subagent 或分批执行并审查。

6. **test-driven-development** - 实现阶段触发。对局部实现工作执行 RED-GREEN-REFACTOR。

7. **requesting-code-review** - 按 plan 和 behavior coverage 做 review，按严重程度报告问题。关键问题阻断继续推进。

8. **finishing-a-development-branch** - 任务完成后触发。验证测试，给出集成选项，并清理 worktree。

agent 会在任何任务前检查相关 skills。这是强制 workflow，不是建议。

## 里面有什么

### Skills Library

**Testing**
- **test-driven-development** - RED-GREEN-REFACTOR cycle。

**Debugging**
- **systematic-debugging** - 4 阶段 root cause process。
- **verification-before-completion** - 在声明完成前确认工作真的被验证。

**Collaboration**
- **brainstorming** - Socratic design refinement 加 bounded behavior grill。
- **writing-plans** - 详细 implementation plans；适用时增加 Behavior Coverage。
- **executing-plans** - 带检查点的批量执行。
- **dispatching-parallel-agents** - 并发 subagent workflows。
- **requesting-code-review** - 带 behavior drift 检查的 review checklist。
- **receiving-code-review** - 以技术严谨性处理反馈。
- **using-git-worktrees** - 并行开发分支。
- **finishing-a-development-branch** - merge/PR 决策流程。
- **subagent-driven-development** - 带两阶段 review 的快速迭代。

**Meta**
- **writing-skills** - 按 Superpowers 方法创建和测试新 skills。
- **using-superpowers** - skills system 的入口说明。

## Eval 状态

当前证据范围是有意收窄的：

- 对齐后的 design/spec eval 在本 fork 上通过，在上游 `origin/main` 上因为缺少 behavior-evaluation 要求而失败。
- writing-plans 的 Behavior Coverage smoke eval 能把 retained-eval 场景产出为具体 tests/checks，但也抓到了未声明 `Scenario 2` 的 task-reference 失败。详见 [Behavior Coverage Writing-Plans Eval](docs/evals/behavior-coverage-writing-plans.md)。
- mutation 和 real-document replay checks 没有证明广泛优越性；它们更适合作为 non-regression 和诊断检查。
- 当前可支持的 claim 不是“全面优于 Superpowers”。可支持的 claim 是：这个 fork 增加了上游 Superpowers 当前没有强制执行的、可审查的 behavior/control harness 要求。

## Philosophy

- **先继承 Superpowers** - 这个 fork 扩展原 workflow，而不是替换它。
- **行为优先于文档体积** - BDD-style Markdown 的目标是让意图行为可审查，不是写更长的 spec。
- **横向反馈 + 纵向反馈** - TDD 检查局部实现；Behavior Coverage 检查整条流程是否仍绑定用户意图。
- **证据优先于声明** - Spec 和 plan 应该写清 observable evidence 和 failure signals。
- **人类终审** - Design review 可以过滤弱设计，但最终批准仍归人类。

## Lineage

BDD Superpowers fork 自 [obra/superpowers](https://github.com/obra/superpowers)，原项目由 Jesse Vincent 和 Prime Radiant 社区创建。

特别鸣谢 [linux.do](https://linux.do/) 社区的讨论、反馈和早期使用信号，这些内容推动了这个 fork 的形成。

特别致谢 [user71](https://linux.do/u/user71) 对 BDD 思路的贡献。

本 fork 保持 MIT license。详见 [LICENSE](LICENSE)。

## Contributing

沿用 Superpowers 自身的纪律：

1. Fork 仓库。
2. 为你的工作创建 branch。
3. 修改 skill 时使用 `writing-skills` skill。
4. 改变行为指导时，增加或更新 eval coverage。
5. 提交 PR，并清楚说明行为影响。

## Community and Issues

- 上游 Superpowers: https://github.com/obra/superpowers
- BDD Superpowers: https://github.com/zhexulong/bdd-superpowers
