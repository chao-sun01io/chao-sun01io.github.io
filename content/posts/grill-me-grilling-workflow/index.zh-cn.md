---
aliases: []
tags: [AI]
title: grill-me 的「烤问」工作流
slug: grill-me-grilling-workflow
date: 2026-07-19T00:00:00Z
lastmod: 2026-07-19T00:00:00Z
summary: 介绍 Matt Pocock 开源的 grill-me / grill-with-docs 系列 skill——一套让 AI 先把你「烤问」清楚再动手写代码的 Plan-then-Agent 工作流。
category: posts
---

> "Grill" 在英文里有「盘问、拷问」之意。将其译为「烤问」，既保留了 grill 的本义，又与中文「拷问」谐音，在 AI 编程的语境里颇为贴切——它指的不是你问 AI，而是 AI 一条一条地追问你，把模糊想法逼成清晰方案。本文是对介绍这套工作流的介绍。

在用 AI辅助编程大行其道的今天， 很多人对于 Plan-then-Agent 的模式已经不陌生了，但是我们还希望Plan 得更好一些，与AI沟通更加充分，通过把意图反复考察清楚，并且拆分为一条条可执行的实现任务，从而达到更好的效果。 grill-me 是一个github 收藏量超过17万的近期热门 skill,  可以帮助用户更好地把问题想清楚再生成代码。 

---

## 一、Matt Pocock 简介

Matt Pocock 是一位常驻英国的全职技术教育者，自称"TypeScript 界的 Rodney Mullen"（Rodney Mullen 是发明了滑板绝大多数基础动作的传奇人物，意指他不仅在用，也在为社区"造词汇"）。他从六年声乐教练起步，辗转 XState 核心团队与 Vercel 开发者布道师，最终创办 Total TypeScript 与 AI 工程训练营 AI Hero，把"把复杂事向非技术人员讲清楚"的能力一路带进了 TypeScript 与 AI 编程教育。

他在 GitHub 开源了 `mattpocock/skills` 仓库——一套他"每天自己都在用"的 AI Agent 技能（Skills）。仓库 README 的第一句话定下了基调：

> "My agent skills that I use every day to do real engineering — not vibe coding."

（我每天用来做**真正的工程**——而不是 vibe coding——的 agent 技能。）

---
## 二、工作流（Workflow）

### 1. 示意图

下图概括了"烤问"工作流的全貌：一条五步主链，一个面向大计划的可选上游，以及一个贯穿全流程的领域语言支撑层。

![grill-me 烤问工作流示意图](diagram.png)

### 2. 工作流解读

这套工作流要解决的核心问题，是 Matt Pocock 所说的 AI 编程"失败模式之首"——**"AI 没做我想要的"**。他引用 Frederick P. Brooks 在《设计的设计》中的概念：当多方协同设计时，参与者之间会漂浮一个隐形的"设计概念（design concept）"共识。人与 AI 之间缺的正是这个共识——你说"加个登录"，AI 不会问你"要不要记住设备、失败几次锁账号、session 多久过期"，而是直接铺一套它自认为合理的方案，等你 review 时往往已写了几百行，回炉成本很高。

"烤问"工作流把交互反转过来：**不是你问 AI，而是 AI 先问到你**。其主线是一条从"对齐"到"交付"的五步链：

```
grill-with-docs  →  to-spec  →  to-tickets  →  implement  →  code-review
   拷问对齐          生成规格      拆解工单        实现工单        代码审查
```

- **第一步 拷问对齐**：在写任何代码之前，agent 沿着设计决策树的每个分支逐条追问，每问一个问题就给出推荐答案，一次只问一个，直到双方达成共识。能从代码库读到的答案，agent 自己读，不来问你。
- **第二步 生成规格**：把拷问中得到共识的理解，凝结成一份规格（spec），作为"目的地"。
- **第三步 拆解工单**：把规格切成一组"曳光弹式"的垂直切片工单（tracer-bullet vertical slices），每个工单声明自己的阻塞依赖。
- **第四步 实现工单**：每个工单在独立的 agent 会话里实现，遵循 TDD、定期类型检查。
- **第五步 代码审查**：实现后由 `/code-review` 在"是否符合编码规范"与"是否忠实于规格"两个轴上并行审查。

对于**太大、太模糊、一个会话装不下**的计划，工作流在上游提供了一个可选入口 `wayfinder`：它把整个努力绘成一张"决策地图"存进 issue tracker，每个决策被切成一个会话体量的工单，通过阻塞关系串联，逐个关掉，路线清晰后再回到主链。`domain-modeling` 则不是一个固定步骤，而是一条**贯穿全流程的支撑层**：它在拷问和建模过程中实时维护项目的"通用语言（ubiquitous language）"，把术语写进 `CONTEXT.md`，把难以逆转的决策写进 `docs/adr/`。

Matt Pocock 自己的总结是：**AI 不会替你做决策，它只会加速你已经做出的决策；如果你自己都没想清楚，AI 加速的只是混乱。**

### 3. 几个重要的 Skill 介绍

#### grill-with-docs（与 grill-me）

`grill-with-docs` 是当前主链的入口，也是 Matt Pocock 如今推荐的版本。它在拷问的同时**留下纸面痕迹**：每当一个模糊术语被敲定，就实时写入根目录的 `CONTEXT.md` 术语表（惰性创建，不预先脚手架）；只有当一个决策"难以逆转、没有上下文会令人意外、且是真实权衡的结果"三者齐备时，才 sparingly 地记一条 ADR。多数会话只会产出一个更锋利的术语表、零到极少 ADR——这是它的预期形态。

`grill-me` 则是更早、更轻的版本：**只拷问、不落文档**。它只做"无情的逐题追问，直到决策树被解决"，每题给出推荐答案并等待反馈。Matt Pocock 已在文档中注明：**他已不再用 `grill-me` 做编码**，改为推荐 `grill-with-docs`（需要与代码库对齐时）或 `domain-model`（需要对照代码库语言塑形时）。`grill-me` 仍可作为"只想做一次压力测试式访谈"的轻量工具使用。

两者的共同引擎是一个共享的"grilling"参考技能。v1.1 版本针对用户反馈做了三项修复：更明确地禁止"一次问多个问题"、在拷问结束与开始实现之间加入确认闸门（"未得到我确认前不得动手"）、以及区分"事实（可从代码库读到）"与"决策（需要用户拍板）"以防止 agent 自我拷问。

#### domain-model（domain-modeling）

`grill-me` 的官方页面写道："Matt 现在通常推荐 `domain-model` 而非 `grill-me`。" 该技能在文档站上以 `domain-modeling` 之名出现，可视为同一脉络的演进。它借鉴 Eric Evans《领域驱动设计》中的"通用语言"思想：让代码、开发者与领域专家说同一套术语。

`domain-modeling` 是**主动**的纪律，而非被动查阅：当你正在"改变"模型——敲定一个规范术语、发现代码与你刚说的话相矛盾、记录一个难以逆转的决策——它才介入。它会主动把代码与你的陈述交叉比对，例如"你的代码取消的是整个 Order，但你刚说支持部分取消——哪个对？"，强制语言与代码一致。它产出的 `CONTEXT.md` 被严格限定为纯术语表，不含实现细节、不含规格；ADR 则保持高门槛，避免沦为流水账。

#### to-spec（前身 to-prd）

`/to-prd` 在 v1.1 中被更名为 `/to-spec`。更名理由是：它产出的其实不是 PRD（产品需求文档，描述产品本身），而是一份更宽泛的**规格**——可以是技术的、非技术的或两者混合。"spec"由此成为整套技能里贯穿始终的统一术语。它把拷问阶段得到的共识，在不重新访谈的前提下，合成为一份规格。

#### to-tickets（前身 to-issues，并合并了 to-plan）

v1.1 把 `/to-plan` 与 `/to-issues` 合并为 `/to-tickets`。更名理由是"issues"一词偏向 GitHub 与 Linear 的术语，而这里的概念更通用——规格之下，是一组带你实现规格的工单。它把规格切成曳光弹式的垂直切片，每个切片声明自己的阻塞边（blocking edges）。同一份产物有两种用法：本地 `tickets.md`（边以文本表达，手工自上而下推进）或真实 issue tracker（边变成原生阻塞链接，任何前置完成的工单都在"前沿"上，可多 agent 并行）。

#### wayfinder

`/wayfinder` 是 v1.1 引入、Matt Pocock 自称"最兴奋"的新技能，面向"一个会话装不下、路线还在雾里"的大型计划，在许多场景下替代 `grill-with-docs` 作为上游入口。它的设计理念是：

> 一个松散的想法到来了，大到放不进一个 agent 会话，且裹在雾里。从这里到目的地的路还看不见。这个技能把路绘成仓库 issue tracker 上的一张共享地图，然后一张工单一张工单地推进，直到路线清晰。

它把决策切成"会话体量"的工单，用阻塞关系串联，工单类型分四种：Research（agent 离线调研）、Grilling（拷问做决策）、Prototype（快速原型抬高讨论保真度）、Task（配置等不需拷问的杂活）。全部工单关掉后，所有信息连同原始工单一起留在地图上，可作为原始资料再转成规格。Matt Pocock 特别强调原型对前端工作的价值——"它该长什么样 / 该怎么交互"是关键问题时，先做一个便宜粗糙的具体物件来反应，比空对空讨论高效得多。

#### ask-matt（技能路由器）

前面这些 skill 各有专攻，但一个现实问题是：**你得先记得它们存在，才知道该用哪个**。`/ask-matt` 就是为此而生的"路由器"——它本身**不做任何实际工作**，不拷问、不写规格、不改代码，只负责"定向"。

你向它描述当前处境（比如"我有个想法但不知从哪开始"、"堆了一堆 bug 报告不知该不该走 `/triage`"、"两个技能看着像不知道区别"），它就告诉你该用哪个 skill 或哪条 flow、按什么顺序跑。它既覆盖那些需要手动触发的 user-invoked 技能（没人会替你自动唤起它们），也指向按名可调的 model-invoked 技能——`/tdd`、`/diagnosing-bugs`、`/prototype`、`/code-review`，以及两份词汇参考 `/domain-modeling`、`/codebase-design`。它回答的是"哪一个，以及何时"。

它给使用者用来思考的核心概念是 **flow（流程）**——一条穿过若干 skill 的路径，而非单个工具。官方的拓扑大致是：一条**主流程**（idea → ship：grill → spec → tickets → implement → review）；两条**匝道**汇入主流程——一条是处理外部涌入 bug 与请求的 triage 匝道，一条是产出改进想法的 codebase-health 匝道；其余都是独立调用的 **standalone**。问一个问题，你会被放到正确的 flow、正确的步骤上，而不是只被塞一个工具。

定位上，`ask-matt` 是**盖在整套技能之上的地图**：它从不身处某条链之中，而是指向每一条链。各技能的文档页也都回链到它。从它出发，最常见的落点是主流程起点 `grill-with-docs`，或处理"非你自己产生的工作"的入口 `triage`。

一句话总结：**不确定用哪个 skill 时，先 `/ask-matt`**——它把"记住整套技能库"的认知负担从你这里卸载到 agent 那里。

### 4. 安装（Installation）

安装通过 `npx` 一行命令完成，可勾选要装哪些 skill、装到哪些 agent（Claude Code、Codex、Cursor 等均支持），对应的 `SKILL.md` 会被放进各自目录：

```bash
# 安装全部 skills（交互式勾选）
npx skills add mattpocock/skills
```

在具体的项目中，如果决定采用这套工作流，建议首先运行 `/setup-matt-pocock-skills`，这是一个一次性配置 skill，会问三个问题：issue tracker 用什么（GitHub / GitLab / 本地 markdown / 其他）、triage label 用什么词、文档放在哪里——配置完即得到一套相对完整的工程脚手架。

后面就可以按照 grill-me 工作流进行了。

### 5. 使用场景（Use cases）

官方与社区归纳的适用时机包括：

- 在写 PRD / 规格之前，先把需求逼问清楚；
- 在让 agent 实现一个功能之前，先对齐设计；
- 在敲定数据模型或 API 形状之前，压力测试几个互相依赖的设计选择；
- 当你希望 agent **反驳你而非附和你**时；
- 绿地项目或大型功能建设，路线尚不明朗时（先走 `wayfinder`）；
- 已有代码库、需要对齐术语并同步建立文档时（走 `grill-with-docs`）。

社区有用户反馈，第一次用 `grill-me` 时 agent 连续问了 38 个问题，其中近一半是自己从未认真想过的边界情况（错误处理、第三方集成失败的回退逻辑、用户数据隔离粒度等）。一个被反复提及的使用姿势是：**不要在"我想做什么"阶段用，而在"我已经有第一版方案"之后再用**——带着具体设计去接受质疑，比带着模糊想法去发散更高效。

---
## 三、局限与批评

这套工作流并非没有争议与短板。综合官方自述与社区（Reddit、博客、技术媒体）讨论，主要的局限与批评集中在以下几点。

**1. 追问疲劳（question fatigue）。** 这是被提及最多的体验问题。一次会话动辄 18–24 个问题，有人遇到 38 甚至 50 个问题。日方一篇深度拆解直言"被追问的疲劳是真实的"，但作者权衡后认为这远小于下游返工的疲劳。即便如此，对小型任务而言，这种强度明显过重。

**2. 分析瘫痪与确认偏误。** 有分析文章指出，过度拷问可能导致"分析瘫痪"（analysis paralysis）；当既有文档质量差时，拷问还可能把错误的共识固化下来，形成"对糟糕文档的确认偏误"。

**3. 早期实现 bug（v1.1 已修）。** v1.1 之前用户频繁报告三类问题：agent 偶尔无视指令一次问多个问题、拷问刚结束就跳进实现、以及在某些情况下 agent "自我拷问"（自己探索代码库自己拷问，不需要人输入）。官方称 v1.1 通过更明确的指令、确认闸门与"事实/决策"区分已显著缓解。

**4. 高度个人化、单人维护。** 仓库虽 star 数巨大，提交却几乎全来自 Matt Pocock 一人（另有 AI 共同署名）。这意味着"巴士因子"较低——维护连续性高度依赖作者个人。

**5. 安全性是 skills 生态的共性问题。** 这并非针对此仓库的特定批评，但值得在此提示：Snyk 的 ToxicSkills 研究曾审计第三方 skills，发现其中 36% 存在提示注入、26% 至少含一处漏洞，生态尚无包签名或校验标准，风险近似早期的 npm/PyPI。安装任何第三方 skill 都应像安装 npm 包一样先审计。

**6. 不是完整方法论。** 与 Superpowers、BMAD 等不同，这套 skills 更像"小而锋利的工具箱"，把控制权留在开发者手里，而非用 hook 与流程替开发者踩刹车。对想要强制流程纪律、多人协作规范化的团队，它的约束力可能不足。

---

## 四、相似的项目

在"如何让 AI 按工程流程做事"这一赛道上，同类项目大致按"愿意把多少控制权交给流程"排成一条光谱。下表汇总几款代表性项目，star 数为 2026-07-19 经 GitHub API 核实值（四舍五入到千位）：

| 项目                                                     | 核心侧重                                                             | 控制权交给流程的程度          | 更适合谁                                  |
| ------------------------------------------------------ | ---------------------------------------------------------------- | ------------------- | ------------------------------------- |
| **Matt Pocock Skills**（mattpocock/skills，约 177K）       | 小而锋利的开发者工具箱（拷问、领域建模、handoff、原型、架构）                               | 低：开发者仍是主驾驶          | 一线开发者与技术负责人；在真实代码库里按需调用，既借 AI 又保留人的判断 |
| **Superpowers**（obra/superpowers，约 257K）               | 强制工作流规范：brainstorm → spec → plan → TDD → review，用 hook 防止跳步      | 中高：用自动触发的流程约束 agent | 想让 agent 少自作主张、强制按流程走的个人/小团队          |
| **GitHub Spec Kit**（github/spec-kit，约 122K）            | 规格驱动、产物驱动，四阶段 `/specify → /plan → /tasks → /implement`，每阶段间有人工审批 | 中：把实现前的约束交给文档       | 需求边界清楚、要可追踪规格与验收标准、跨团队协作的项目           |
| **BMAD Method**（bmad-code-org/BMAD-METHOD，约 51K）       | 多角色敏捷：用 PM、架构师、开发、QA 等 agent 人格组织完整敏捷流程                          | 高：把产品流程交给多角色 agent  | 想用多角色 agent 跑完整产品流程、企业级复杂项目           |
| **GSD / get-shit-done**（gsd-build/get-shit-done，约 65K） | runtime 与自动执行，用状态机、长期任务、自动推进接管工作流                                | 很高：让 runtime 接管持续推进 | 想把 agent 当持续执行系统、应对超长复杂任务             |
| **OpenSpec**（Fission-AI/OpenSpec，约 62K）                | 规格驱动开发（SDD），偏棕地项目与多人协作                                           | 中                   | 已有代码库、需对齐需求的多协作场景                     |
| **Anthropic Skills**（anthropics/skills，约 163K）         | 官方技能目录，最广覆盖                                                      | 低：只提供技能，不强制方法论      | 想要可复用技能库、自行组合方法论者                     |


需要说明的是，这些项目并非互相替代，更像是回答同一个问题——"工程师愿意把多少控制权交给流程"——的不同答案。Matt Pocock Skills 的差异化定位在于：它不试图"拥有整个流程"，而是把工程基本功（拷问、统一语言、TDD、原型、架构）封装成可按需调用的模块，控制权始终在开发者手中；代价是它不提供 Superpowers/BMAD 那种强制纪律。

---

## 五、参考（Reference）

- Matt Pocock. *v1.1: /wayfinder, /to-spec, /to-tickets, /grilling improvements, and much more*. https://www.aihero.dev/skills/skills-changelog-v1-1-wayfinder-to-spec-to-tickets-grilling-improvements
- Matt Pocock. *grill-with-docs: Align Before You Build*. https://www.aihero.dev/grill-with-docs
- Matt Pocock. *The /grill-me Skill*. https://www.aihero.dev/skills-grill-me
- Matt Pocock. *The /domain-modeling Skill*. https://aihero.dev/skills-domain-modeling
- Matt Pocock Skills 仓库. https://github.com/mattpocock/skills
- "Matt Pocock Skills 深度拆解". https://zhichai.net/topic/177981536
- "The Solo Architect: Matt Pocock"（zread.ai 仓库解读）. https://zread.ai/mattpocock/skills/5-about-contributors
- "Agentic Skills Frameworks Compared"（Ry Walker Research，含安全性与生态对比）. https://rywalker.com/research/agentic-skills-frameworks
- "grill-me Skill Deep Dive"（oflight.co.jp，含追问疲劳与模型选择讨论）. https://www.oflight.co.jp/en/columns/grill-me-agent-skill-2026-07
- "Deep Dive into Grill Me"（ArceApps，含分析瘫痪与确认偏误批评）. https://arceapps.com/blog/grill-me-claude-skill-deep-dive

---

*本文为基于公开资料的中立介绍，star 数与功能细节可能随项目演进而变化，请以官方仓库与文档站为准。*
