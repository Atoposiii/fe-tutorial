# OpenSpec 学习笔记

> 基于 [https://github.com/Fission-AI/OpenSpec/tree/main/docs](https://github.com/Fission-AI/OpenSpec/tree/main/docs) 文档整理

---

## 一、文档总览



OpenSpec 是一个面向 AI 编程助手（Claude Code、Cursor、Windsurf 等）的**规格驱动开发工具**，帮助你和 AI 在写代码之前先就"要构建什么"达成共识。

### 核心理念（concepts.md）

OpenSpec 围绕四个原则设计：**流动而非僵化、迭代而非瀑布、简单而非复杂、存量优先**（brownfield-first，适合改造现有系统而非只建新项目）。

整个工作区分为两个核心目录：

- `openspec/specs/` — 系统当前行为的"事实来源"，按领域组织（如 `auth/`、`payments/`）
- `openspec/changes/` — 每个变更一个文件夹，包含 proposal、specs（delta）、design、tasks 四类制品

**Delta Spec** 是核心概念：用 `ADDED / MODIFIED / REMOVED` 三个区块描述相对于现有 spec 的变化，归档时自动合并进主 spec。

### 安装（installation.md）

需要 Node.js 20.19.0+，支持 npm / pnpm / yarn / bun 全局安装，也支持 Nix：

```bash
npm install -g @fission-ai/openspec@latest
cd your-project && openspec init
```

### 快速上手（getting-started.md）

默认工作流（`core` profile）：

```
/opsx:propose → /opsx:apply → /opsx:archive
```

扩展工作流（需手动开启）：

```
/opsx:new → /opsx:ff 或 /opsx:continue → /opsx:apply → /opsx:verify → /opsx:archive
```

每个变更文件夹包含四个制品，依次构建：`proposal.md`（为什么做）→ `specs/`（做什么）→ `design.md`（怎么做）→ `tasks.md`（实施清单）。

### OPSX 工作流（opsx.md）

OPSX 是对旧版 OpenSpec 的全面升级，核心改变是**从"阶段锁定"变为"流动动作"**。旧版把规划、实施、归档分成三个不可回退的阶段；OPSX 允许随时更新任何制品，依赖关系只是"使能器"而非"门禁"。

架构上，OPSX 使用外部 YAML Schema 和依赖图引擎，生成跨编辑器兼容的 Skill 文件，而旧版是把模板硬编码在 TypeScript 里。

### 命令参考（commands.md）

默认 `core` profile 提供四个命令：

| 命令 | 作用 |
|------|------|
| `/opsx:propose` | 一步创建变更并生成所有规划制品 |
| `/opsx:explore` | 在提交变更前自由探索想法 |
| `/opsx:apply` | 按 tasks.md 实施代码 |
| `/opsx:archive` | 归档已完成的变更 |

扩展工作流还有 `/opsx:new`、`/opsx:continue`、`/opsx:ff`、`/opsx:verify`、`/opsx:sync`、`/opsx:bulk-archive`、`/opsx:onboard`。

### 定制化（customization.md）

三个层次的定制：

1. **项目配置**（`openspec/config.yaml`）：设置默认 schema、注入项目上下文（tech stack、规范等）、为特定制品添加规则。上下文会被注入到每次 AI 规划请求中，比旧版 `project.md` 更可靠。
2. **自定义 Schema**：在 `openspec/schemas/` 下定义自己的工作流，可以 fork 内置 schema 修改，也可以从零创建。Schema 用 YAML 描述制品、依赖关系和 AI 指令模板。
3. **全局覆盖**：在 `~/.local/share/openspec/schemas/` 下放置 schema，跨项目共享。


### 支持的工具（supported-tools.md）

支持 25+ 款 AI 编程助手，包括 Claude Code、Cursor、Windsurf、GitHub Copilot、Cline、RooCode、Gemini CLI 等。`openspec init` 时可以选择配置哪些工具，也支持 `--tools all` 一次性配置全部。

---

## 二、官方工作流详解：`/opsx:propose → /opsx:apply → /opsx:archive`

这是默认的 `core` profile，也是最推荐的快速路径。

### 第一步：`/opsx:propose`

你只需要告诉 AI 你想做什么，它会自动创建 `openspec/changes/<change-name>/` 目录，并一次性生成四个制品：

**proposal.md** — 回答"为什么做、做什么"：

```markdown
# Proposal: Add Dark Mode
## Intent
用户反馈需要深色模式以减少夜间使用时的眼睛疲劳。
## Scope
In scope: 主题切换按钮、系统偏好检测、localStorage 持久化
Out of scope: 自定义颜色主题（留到下个版本）
## Approach
使用 CSS custom properties + React Context 管理状态。
```

**specs/ui/spec.md** — 回答"系统行为是什么"（Delta 格式）：

```markdown
# Delta for UI
## ADDED Requirements
### Requirement: Theme Selection
The system SHALL allow users to choose between light and dark themes.
#### Scenario: Manual toggle
- GIVEN a user on any page
- WHEN the user clicks the theme toggle
- THEN the theme switches immediately
- AND the preference persists across sessions
```

**design.md** — 回答"技术上怎么做"：架构决策、组件设计、数据流

**tasks.md** — 回答"具体步骤是什么"：

```markdown
# Tasks
## 1. Theme Infrastructure
- [ ] 1.1 Create ThemeContext with light/dark state
- [ ] 1.2 Add CSS custom properties for colors
- [ ] 1.3 Implement localStorage persistence
## 2. UI Components
- [ ] 2.1 Create ThemeToggle component
```

### 第二步：`/opsx:apply`

AI 读取 `tasks.md`，逐条实施，完成后打勾 `[x]`。中途发现设计有问题可以随时回头修改制品，这是 OPSX 相比旧版最大的改进——**没有阶段门禁，随时可以更新任何制品**。

### 第三步：`/opsx:archive`

AI 会：

1. 检查所有 tasks 是否完成（未完成会警告但不阻塞）
2. 询问是否将 delta specs 合并进主 `openspec/specs/`
3. 将整个变更文件夹移动到 `openspec/changes/archive/2025-01-24-add-dark-mode/`，保留完整历史

### 一个好的规范应该是什么样的

官方文档对此有明确的观点，核心是**行为契约，而非实现计划**。

**Spec 应该写什么：**

- 用户或下游系统可以观察到的行为
- 输入、输出、错误条件
- 外部约束（安全、隐私、兼容性）
- 可以被测试或验证的场景

**Spec 不应该写什么：**

- 内部类名、函数名
- 框架或库的选择（那是 design.md 的事）
- 逐步实施细节（那是 tasks.md 的事）

官方给了一个快速判断标准：**如果实现可以改变而外部可见行为不变，那这个内容就不属于 spec。**

**场景（Scenario）的质量标准：**

- 可测试——你能为它写一个自动化测试
- 覆盖正常路径和边界情况
- 使用 Given/When/Then 结构

**需求强度用 RFC 2119 关键词区分：**

- `MUST / SHALL` — 绝对要求
- `SHOULD` — 推荐，但有例外情况
- `MAY` — 可选

官方还强调了**渐进式严格度**的原则：大多数变更用"精简 spec"就够了（简短的行为描述 + 几个验收场景），只有跨团队、涉及 API 契约变更、安全/隐私相关的高风险变更才需要写完整的详细 spec。

---

## 三、搭配 Claude Code 使用

### 安装与初始化

```bash
npm install -g @fission-ai/openspec@latest
cd your-project
openspec init
```

`init` 时会询问你要配置哪些工具，选择 `claude`，它会在项目里生成：

```
.claude/
└── skills/
    ├── openspec-propose/SKILL.md
    ├── openspec-explore/SKILL.md
    ├── openspec-apply-change/SKILL.md
    └── openspec-archive-change/SKILL.md
```

Claude Code 启动时会自动检测这些 Skill 文件，之后你就可以在对话框里直接用 `/opsx:*` 命令了。

### 日常使用流程

**1. 开始一个新功能**

在 Claude Code 的对话框里输入：

```
/opsx:propose add-user-avatar
```

Claude 会读取 Skill 文件里的指令，然后在你的项目里创建 `openspec/changes/add-user-avatar/`，并生成 proposal、specs、design、tasks 四个制品。整个过程你只需要描述意图，Claude 负责结构化。

**2. 不确定怎么做时先探索**

```
/opsx:explore
```

然后跟 Claude 自由对话，比如"我们的上传服务现在是怎么工作的？头像应该存 OSS 还是数据库？"。Claude 会读你的代码库来回答，帮你想清楚再开始。

**3. 实施代码**

```
/opsx:apply
```

Claude 读取 `tasks.md`，逐条写代码、打勾。如果中途发现设计有问题，直接告诉它"这里需要调整"，它会先更新 design.md 或 tasks.md，再继续实施——不需要重来。

**4. 归档**

```
/opsx:archive
```

Claude 会把 delta specs 合并进主 `openspec/specs/`，然后把整个变更文件夹移到 `openspec/changes/archive/`。

### 让 Claude 更了解你的项目

`init` 时会引导你创建 `openspec/config.yaml`，这是最关键的配置，它的内容会被注入到每一次 Claude 生成制品的请求里：

```yaml
# openspec/config.yaml
schema: spec-driven

context: |
  Tech stack: TypeScript, React, Node.js, PostgreSQL
  API 风格: RESTful，接口文档在 docs/api.md
  测试框架: Vitest + Playwright
  代码规范: ESLint strict + Prettier
  所有公开 API 必须保持向后兼容

rules:
  proposal:
    - 包含回滚方案
    - 标明影响的模块和团队
  specs:
    - 使用 Given/When/Then 格式写场景
    - 优先复用已有模式，不要发明新的
  design:
    - 复杂流程需要画时序图
  tasks:
    - 每个子任务控制在 2 小时以内
```

有了这个配置，Claude 生成的每一份制品都会自动遵循你团队的规范，不需要每次都在对话里重复说明。

### 一个实际的对话示例

```
你：/opsx:propose 给用户中心添加头像上传功能

Claude：我来创建这个变更...
        [读取你的代码库，了解现有的用户模型和文件上传逻辑]

        ✓ openspec/changes/add-user-avatar/proposal.md
        ✓ openspec/changes/add-user-avatar/specs/user/spec.md
        ✓ openspec/changes/add-user-avatar/design.md
        ✓ openspec/changes/add-user-avatar/tasks.md

        已就绪，运行 /opsx:apply 开始实施。

你：/opsx:apply

Claude：开始实施 add-user-avatar...
        ✓ 1.1 创建 AvatarUpload 组件
        ✓ 1.2 添加文件类型和大小校验
        ✗ 1.3 集成 OSS 上传...
        
        发现问题：项目里已有 useFileUpload hook，
        建议复用而不是新建。我来更新 tasks.md...
        ✓ 更新后继续实施

你：/opsx:archive
```

### 几个实用技巧

**并行处理多个变更**时，用变更名区分：

```
/opsx:apply add-user-avatar
/opsx:apply fix-login-bug
```

**想开启更细粒度的控制**（逐步创建制品），运行：

```bash
openspec config profile   # 选择 workflows
openspec update           # 更新 Skill 文件
```

之后就可以用 `/opsx:new` → `/opsx:continue`（一次一个制品）→ `/opsx:ff`（一次全部）的扩展工作流。

**更新 Skill 文件**（升级 openspec 版本后）：

```bash
openspec update
```

然后重启 Claude Code，新的 Skill 会自动生效。

---
## 四、`openspec/config.yaml` 配置指南

### 让所有制品输出中文

在 `context` 里加一条说明就行：

```yaml
# openspec/config.yaml
schema: spec-driven

context: |
  Tech stack: TypeScript, React
  API 风格: RESTful

  Language: All artifacts (proposal.md, specs, design.md, tasks.md) MUST be written in Chinese (中文).
  所有制品内容必须使用中文撰写，包括需求描述、场景、设计说明和任务清单。
  技术术语（如 JWT、OSS、API 路径等）保持英文，其余内容全部用中文。

rules:
  specs:
    - 使用 Given/When/Then 格式撰写场景，关键词保持英文（GIVEN/WHEN/THEN），内容用中文
    - 需求强度关键词保持英文（SHALL/MUST/SHOULD）
  proposal:
    - 包含回滚方案
  tasks:
    - 每个子任务控制在 2 小时以内
```

**注意事项：**

`context` 里同时用中英文写了两遍，是因为 AI 对英文指令的遵循度通常更稳定，加上中文说明作为强化，效果更好。

`GIVEN/WHEN/THEN` 和 `SHALL/MUST/SHOULD` 建议保留英文，因为这是 OpenSpec 解析 delta spec 时依赖的结构化标记，改成中文可能影响 `/opsx:archive` 时的自动合并逻辑。

`ADDED / MODIFIED / REMOVED` 这三个 delta 区块标题同理，也建议保持英文。

其余所有内容——需求描述、场景内容、设计说明、任务清单——都可以全中文，不影响工具的正常运作。

---

## 五、`/opsx:explore` 使用指南

`/opsx:explore` 是在**正式开始一个变更之前**用来"想清楚"的命令，没有任何结构要求，就是跟 AI 自由对话。

### 典型使用场景

**场景一：需求模糊，不知道从哪里下手**

```
你：/opsx:explore
AI：你想探索什么？

你：我们要做一个消息通知中心，但不确定应该做站内信
    还是直接推送，或者两个都做

AI：让我先看看你现有的代码...
    [扫描代码库，找到现有的通知相关代码]
    
    你们目前有 email 通知的基础设施，但没有实时推送。
    站内信需要 WebSocket 或轮询，推送需要接入 APNs/FCM。
    
    几个问题帮你想清楚：
    1. 用户主要在 Web 还是 App 上？
    2. 通知的时效性要求高吗（秒级 vs 分钟级）？
    3. 用户需要查看历史通知吗？

你：主要是 Web，时效性要求不高，需要历史记录

AI：那站内信更合适。轮询方案实现简单，
    你们的用户量级用不着 WebSocket 的复杂度。
    要开始一个变更吗？

你：好，开始吧

AI：运行 /opsx:propose notification-center 来创建变更。
```

**场景二：技术方案有争议，想比较选项**

```
你：/opsx:explore 我们的搜索要不要上 Elasticsearch

AI：[读取代码库，了解现有数据量和查询模式]

    你们目前用 PostgreSQL 的 LIKE 查询，数据量 50 万条。
    
    直接对比：
    - 继续用 PG：加 GIN 索引支持全文搜索，零运维成本，
      够用到千万级
    - 上 ES：强大但引入新的基础设施，需要数据同步，
      运维复杂度翻倍
    
    50 万条数据量，PG 全文搜索完全够用。
    除非你们有多语言分词或复杂相关性排序的需求，
    否则不建议现在引入 ES。
```

**场景三：改动影响面不清楚，想先摸底**

```
你：/opsx:explore 如果我们把用户 ID 从自增整数改成 UUID
    会影响哪些地方

AI：[扫描代码库]

    影响面相当大，我找到了：
    - 23 个接口的路径参数用了 userId
    - 5 张关联表有外键
    - 前端有 3 处做了 userId 是数字的假设（用了 parseInt）
    - 埋点日志里 userId 是 number 类型
    
    这个改动建议拆成多个变更分批做，
    而不是一次性迁移。要我帮你规划拆分方案吗？
```

### 和 `/opsx:propose` 的区别

`/opsx:explore` 不创建任何文件，纯粹是对话和思考。想清楚之后，AI 会提示你运行 `/opsx:propose` 正式开始。

**简单来说：需求清晰就直接 `/opsx:propose`，有疑问或风险就先 `/opsx:explore`。**

---

## 六、修复问题应该用什么命令

修复问题同样走标准流程，用 `/opsx:propose` 开始就行，命名上带 `fix-` 前缀来区分：

```
/opsx:propose fix-login-timeout-bug
```

### 根据问题复杂度选择路径

**简单 bug（原因清楚，改动小）**

直接 propose，AI 会生成一个轻量的变更，proposal 和 tasks 为主，specs 可能很简单甚至不需要 delta：

```
/opsx:propose fix-avatar-not-showing-after-upload
```

**原因不明，需要先排查**

先用 explore 让 AI 帮你定位问题：

```
/opsx:explore
```

然后描述现象：

```
你：登录后偶发跳回登录页，复现概率大概 30%，
    不知道是 token 过期判断的问题还是并发请求的问题

AI：[扫描 auth 相关代码]

    找到了，你们的 token 刷新逻辑有竞态条件——
    多个请求同时发现 token 快过期时会并发触发刷新，
    导致其中一些请求拿到已失效的旧 token。
    
    建议用请求队列解决，要开始一个变更吗？

你：好

AI：运行 /opsx:propose fix-token-refresh-race-condition
```

**影响面大的问题（涉及多个模块）**

explore 摸清影响范围后，考虑拆成多个 fix 分批处理：

```
你：/opsx:explore 支付回调偶发丢失，想看看影响哪些地方

AI：[扫描代码]
    涉及：回调接收、幂等校验、订单状态机、通知触发
    建议拆成两个变更：
    1. fix-payment-callback-idempotency（核心修复）
    2. fix-payment-callback-retry（补偿机制）
```

### fix 类变更的 spec delta 怎么写

bug 修复对应的 delta 通常是 `MODIFIED`，描述行为从"错误的"变成"正确的"：

```markdown
# Delta for Auth

## MODIFIED Requirements

### Requirement: Token Refresh
The system SHALL refresh the access token without race conditions.
（Previously: 并发请求可能触发多次刷新导致部分请求失败）

#### Scenario: 并发请求时 token 刷新
- GIVEN 多个请求同时检测到 token 即将过期
- WHEN 第一个请求触发刷新
- THEN 其余请求等待刷新完成后复用新 token
- AND 不会出现使用已失效 token 的情况
```

**简单来说：修 bug 和加功能用的是同一套命令，`/opsx:propose` 走起，原因不明就先 `/opsx:explore` 排查，想清楚再 propose。**
