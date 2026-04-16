@@ -0,0 +1,570 @@
+# F2ELint 新项目接入指南
+
+> F2ELint 是《阿里巴巴前端规约》的配套 Lint 工具，封装了 ESLint、Stylelint、commitlint、markdownlint、Prettier 等工具的安装与配置，让新项目能够一键接入完整的代码规范体系。
+>
+> 本文档适用于**从零开始**的新前端项目。
+
+---
+
+## 目录
+
+- [环境要求](#环境要求)
+- [快速开始](#快速开始)
+- [初始化详解](#初始化详解)
+- [日常使用](#日常使用)
+- [配置文件说明](#配置文件说明)
+- [规则包说明](#规则包说明)
+- [Git 提交卡口](#git-提交卡口)
+- [IDE 集成](#ide-集成)
+- [常见问题](#常见问题)
+- [手动接入（不使用 CLI）](#手动接入不使用-cli)
+
+---
+
+## 环境要求
+
+| 工具 | 最低版本 | 推荐版本 |
+|------|---------|---------|
+| Node.js | 18.x | 20.x LTS |
+| npm | 8.x | 10.x |
+| Git | 2.x | 最新版 |
+
+---
+
+## 快速开始
+
+### 第一步：全局安装 F2ELint
+
+```bash
+npm install f2elint -g
+```
+
+安装完成后验证：
+
+```bash
+f2elint -h
+```
+
+### 第二步：在项目根目录初始化
+
+```bash
+cd your-project
+f2elint init
+```
+
+CLI 会交互式询问项目类型，根据你的选择自动安装依赖并生成配置文件。**整个过程约 1~2 分钟**，完成后即可使用。
+
+### 第三步：验证接入效果
+
+```bash
+# 扫描项目，查看规约问题
+f2elint scan
+
+# 自动修复可修复的问题
+f2elint fix
+```
+
+---
+
+## 初始化详解
+
+执行 `f2elint init` 后，CLI 会依次询问以下问题：
+
+```
+? 请选择项目语言类型
+  ❯ JavaScript
+    TypeScript
+
+? 请选择项目框架
+  ❯ React
+    Vue
+    无框架（Node.js / 纯 JS）
+
+? 是否使用 Prettier 格式化代码？
+  ❯ 是
+    否
+
+? 是否需要 Stylelint（CSS/Less/Sass 规范）？
+  ❯ 是
+    否
+```
+
+根据你的选择，F2ELint 会自动完成以下操作：
+
+**1. 安装依赖**
+
+```
+eslint + eslint-config-ali（及对应框架插件）
+stylelint + stylelint-config-ali
+commitlint + commitlint-config-ali
+markdownlint + markdownlint-config-ali
+prettier + prettier-config-ali
+husky（git hooks 管理）
+lint-staged（只扫暂存区文件）
+```
+
+**2. 生成配置文件**
+
+```
+your-project/
+├── .eslintrc.js          # ESLint 配置
+├── .eslintignore         # ESLint 忽略规则
+├── .stylelintrc.js       # Stylelint 配置
+├── .stylelintignore      # Stylelint 忽略规则
+├── commitlint.config.js  # commitlint 配置
+├── .markdownlint.json    # markdownlint 配置
+├── .markdownlintignore   # markdownlint 忽略规则
+├── .prettierrc.js        # Prettier 配置
+├── .editorconfig         # 编辑器基础配置
+├── f2elint.config.js     # F2ELint 自身配置
+└── .vscode/
+    ├── extensions.json   # 推荐安装的 VSCode 插件
+    └── settings.json     # 保存时自动 fix 配置
+```
+
+**3. 配置 Git 提交卡口**
+
+通过 husky 在 `pre-commit` 和 `commit-msg` 阶段自动注入检查脚本，有问题时阻断提交。
+
+---
+
+## 日常使用
+
+### 扫描项目
+
+```bash
+# 扫描整个项目
+f2elint scan
+
+# 只扫描指定目录
+f2elint scan --include src/
+
+# 只报告 error 级别问题（忽略 warning）
+f2elint scan --quiet
+
+# 扫描并输出报告文件（生成 f2elint-report.json）
+f2elint scan -o
+```
+
+### 自动修复
+
+```bash
+# 修复整个项目
+f2elint fix
+
+# 只修复指定目录
+f2elint fix --include src/
+```
+
+> ⚠️ `fix` 只能自动修复**格式类**问题（缩进、引号、分号等），逻辑类问题需要手动修复。执行前建议先 `git stash` 或确认代码已提交，方便对比修复前后的差异。
+
+### 升级规则包
+
+```bash
+# 升级 F2ELint 及所有规则包到最新版
+f2elint update
+```
+
+---
+
+## 配置文件说明
+
+### `.eslintrc.js`
+
+F2ELint 根据项目类型生成对应的 ESLint 配置，以 Vue + TypeScript 项目为例：
+
+```js
+module.exports = {
+  root: true,
+  extends: [
+    'eslint-config-ali/typescript',  // TS 基础规则
+    'eslint-config-ali/vue',         // Vue 规则
+  ],
+  rules: {
+    // 在这里覆盖或新增项目特有规则
+  },
+}
+```
+
+常见的项目类型对应的 extends：
+
+| 项目类型 | extends 配置 |
+|---------|-------------|
+| JS 项目 | `eslint-config-ali` |
+| TS 项目 | `eslint-config-ali/typescript` |
+| React JS | `eslint-config-ali/react` |
+| React TS | `eslint-config-ali/typescript`、`eslint-config-ali/react` |
+| Vue JS | `eslint-config-ali/vue` |
+| Vue TS | `eslint-config-ali/typescript`、`eslint-config-ali/vue` |
+| Node.js | `eslint-config-ali/node` |
+
+### `.stylelintrc.js`
+
+```js
+module.exports = {
+  extends: ['stylelint-config-ali'],
+  rules: {
+    // 在这里覆盖或新增项目特有规则
+  },
+}
+```
+
+### `commitlint.config.js`
+
+```js
+module.exports = {
+  extends: ['commitlint-config-ali'],
+}
+```
+
+commitlint-config-ali 要求 commit message 遵循以下格式：
+
+```
+<type>(<scope>): <subject>
+
+# 示例
+feat(login): 新增手机号登录方式
+fix(cart): 修复数量为 0 时仍可提交的问题
+refactor(api): 重构请求层，统一错误处理
+```
+
+支持的 type 类型：
+
+| type | 说明 |
+|------|------|
+| `feat` | 新功能 |
+| `fix` | Bug 修复 |
+| `refactor` | 重构（不涉及功能变更） |
+| `perf` | 性能优化 |
+| `style` | 代码格式调整（不影响逻辑） |
+| `test` | 测试相关 |
+| `docs` | 文档变更 |
+| `chore` | 构建/工具链变更 |
+| `revert` | 回滚提交 |
+| `ci` | CI/CD 配置变更 |
+
+### `f2elint.config.js`
+
+F2ELint 自身的配置文件，控制启用哪些检查器：
+
+```js
+module.exports = {
+  enableESLint: true,       // 启用 ESLint
+  enableStylelint: true,    // 启用 Stylelint
+  enableMarkdownlint: true, // 启用 markdownlint
+  enablePrettier: true,     // 启用 Prettier
+}
+```
+
+---
+
+## 规则包说明
+
+F2ELint 使用的规则包均基于《阿里巴巴前端规约》定制，核心覆盖范围如下：
+
+### eslint-config-ali 覆盖的检查维度
+
+| 维度 | 典型规则 | 严重级别 |
+|------|---------|---------|
+| **变量声明** | 禁止未使用变量、禁止重复声明 | error |
+| **类型安全** | 禁止 `any`、要求显式返回类型 | error / warn |
+| **异步处理** | 禁止空 catch 块、要求处理 Promise | error |
+| **React Hooks** | `rules-of-hooks`、`exhaustive-deps` | error / warn |
+| **安全** | 禁止 `eval()`、禁止 `dangerouslySetInnerHTML` 未净化 | error |
+| **代码风格** | 缩进、引号、分号、换行 | warn（Prettier 接管） |
+| **命名规范** | 变量 camelCase、组件 PascalCase | warn |
+| **导入规范** | 禁止循环依赖、禁止全量引入大型库 | warn |
+
+### stylelint-config-ali 覆盖的检查维度
+
+| 维度 | 典型规则 |
+|------|---------|
+| **属性书写顺序** | 按 position → display → box-model → typography 排列 |
+| **颜色规范** | 禁止无效颜色值、统一使用小写十六进制 |
+| **单位规范** | 禁止使用 `px` 以外的绝对单位（可配置） |
+| **选择器规范** | 禁止过深的选择器嵌套（默认 ≤ 3 层） |
+| **空值规范** | 禁止空规则块、禁止重复属性 |
+
+---
+
+## Git 提交卡口
+
+`f2elint init` 完成后，会自动配置两个 Git 钩子：
+
+### pre-commit 钩子
+
+在 `git commit` 时自动触发，只扫描**本次提交的变更文件**（通过 lint-staged 实现，不扫全量）：
+
+```bash
+f2elint commit-file-scan
+```
+
+默认只对 **error 级别**问题卡口，warning 不阻断提交。如需对 warning 也卡口，开启严格模式：
+
+```bash
+# 修改 .husky/pre-commit
+f2elint commit-file-scan --strict
+```
+
+### commit-msg 钩子
+
+在 `git commit` 时校验 commit message 格式：
+
+```bash
+f2elint commit-msg-scan
+```
+
+不符合 commitlint 规范的 message 会被拒绝，并给出提示：
+
+```
+✖ type must be one of [feat, fix, refactor, perf, style, test, docs, chore, revert, ci]
+```
+
+### 跳过卡口（紧急情况）
+
+```bash
+# 跳过 pre-commit 检查（不推荐，仅紧急情况使用）
+git commit --no-verify -m "fix: 紧急修复"
+```
+
+---
+
+## IDE 集成
+
+### VSCode（推荐）
+
+`f2elint init` 会自动生成 `.vscode/extensions.json`，用 VSCode 打开项目时会提示安装以下插件：
+
+| 插件 | 插件 ID | 作用 |
+|------|---------|------|
+| ESLint | `dbaeumer.vscode-eslint` | 实时显示 ESLint 错误 |
+| Stylelint | `stylelint.vscode-stylelint` | 实时显示 CSS 错误 |
+| Prettier | `esbenp.prettier-vscode` | 保存时自动格式化 |
+| markdownlint | `davidanson.vscode-markdownlint` | Markdown 规范检查 |
+| EditorConfig | `editorconfig.editorconfig` | 统一编辑器基础配置 |
+
+同时生成的 `.vscode/settings.json` 会配置**保存时自动 fix**：
+
+```json
+{
+  "editor.formatOnSave": true,
+  "editor.defaultFormatter": "esbenp.prettier-vscode",
+  "editor.codeActionsOnSave": {
+    "source.fixAll.eslint": true,
+    "source.fixAll.stylelint": true
+  }
+}
+```
+
+### WebStorm / IntelliJ IDEA
+
+在 `Preferences → Languages & Frameworks → JavaScript → Code Quality Tools → ESLint` 中：
+
+1. 选择 **Automatic ESLint configuration**
+2. 勾选 **Run eslint --fix on save**
+
+Stylelint 同理，在 `Preferences → Languages & Frameworks → Style Sheets → Stylelint` 中启用。
+
+---
+
+## 常见问题
+
+### Q：`f2elint init` 时提示权限错误
+
+```bash
+# macOS / Linux 配置 npm 全局目录到用户目录（推荐）
+npm config set prefix ~/.npm-global
+export PATH=~/.npm-global/bin:$PATH
+
+# 或者使用 sudo（不推荐）
+sudo npm install f2elint -g
+```
+
+### Q：初始化后 ESLint 报大量错误，项目无法正常开发
+
+这是正常现象，说明项目存在不符合规约的历史代码。建议分两步处理：
+
+第一步，先用 `f2elint fix` 自动修复格式类问题（通常能解决 60%~80% 的报错）：
+
+```bash
+f2elint fix --include src/
+```
+
+第二步，对于剩余的逻辑类问题，可以临时将部分规则从 `error` 降为 `warn`，分批修复，避免影响日常开发：
+
+```js
+// .eslintrc.js
+rules: {
+  'no-unused-vars': 'warn',  // 先降级，后续再修复
+}
+```
+
+### Q：monorepo 项目如何接入
+
+在每个子包目录分别执行 `f2elint init`，各子包可以有独立的配置。根目录放公共的 `.editorconfig` 和 `commitlint.config.js`：
+
+```
+monorepo/
+├── .editorconfig           # 根目录公共配置
+├── commitlint.config.js    # 根目录公共 commit 规范
+├── packages/
+│   ├── app-web/
+│   │   ├── .eslintrc.js    # 子包独立 ESLint 配置（Vue）
+│   │   └── .stylelintrc.js
+│   └── app-node/
+│       └── .eslintrc.js    # 子包独立 ESLint 配置（Node.js）
+```
+
+### Q：项目已有 ESLint 配置，想迁移到 F2ELint
+
+不建议直接覆盖，推荐渐进式迁移：
+
+1. 先用 `f2elint scan` 扫描，了解与阿里规约的差距
+2. 在现有 `.eslintrc.js` 中逐步添加 `eslint-config-ali` 的规则
+3. 确认无冲突后，再将 `extends` 切换到 `eslint-config-ali`
+
+### Q：`f2elint commit-file-scan` 在 CI 环境报错
+
+CI 环境没有 git 暂存区，不需要执行 `commit-file-scan`。在 CI 中直接用 `f2elint scan` 全量扫描即可：
+
+```yaml
+# GitHub Actions 示例
+- name: Lint Check
+  run: f2elint scan --quiet
+```
+
+### Q：Windows 环境下 `f2elint` 命令无法识别
+
+```bash
+# 改用以下命令
+f2elint.cmd init
+f2elint.cmd scan
+```
+
+---
+
+## 手动接入（不使用 CLI）
+
+如果你不想使用 `f2elint init`，也可以手动安装各工具并引用规则包，效果完全一致。
+
+### 第一步：安装依赖
+
+```bash
+# ESLint（以 Vue + TypeScript 项目为例）
+npm install -D eslint eslint-config-ali \
+  @typescript-eslint/parser @typescript-eslint/eslint-plugin \
+  eslint-plugin-vue
+
+# Stylelint
+npm install -D stylelint stylelint-config-ali
+
+# commitlint
+npm install -D @commitlint/cli commitlint-config-ali
+
+# Prettier
+npm install -D prettier prettier-config-ali
+
+# Git hooks
+npm install -D husky lint-staged
+```
+
+### 第二步：创建配置文件
+
+**.eslintrc.js**
+
+```js
+module.exports = {
+  root: true,
+  extends: [
+    'eslint-config-ali/typescript',
+    'eslint-config-ali/vue',
+  ],
+  parser: 'vue-eslint-parser',
+  parserOptions: {
+    parser: '@typescript-eslint/parser',
+    ecmaVersion: 2022,
+    sourceType: 'module',
+  },
+  rules: {},
+}
+```
+
+**.stylelintrc.js**
+
+```js
+module.exports = {
+  extends: ['stylelint-config-ali'],
+  rules: {},
+}
+```
+
+**commitlint.config.js**
+
+```js
+module.exports = {
+  extends: ['commitlint-config-ali'],
+}
+```
+
+**.prettierrc.js**
+
+```js
+module.exports = require('prettier-config-ali')
+```
+
+**.editorconfig**
+
+```ini
+root = true
+
+[*]
+indent_style = space
+indent_size = 2
+end_of_line = lf
+charset = utf-8
+trim_trailing_whitespace = true
+insert_final_newline = true
+
+[*.md]
+trim_trailing_whitespace = false
+```
+
+### 第三步：配置 husky + lint-staged
+
+```bash
+# 初始化 husky
+npx husky install
+
+# 添加 pre-commit 钩子
+npx husky add .husky/pre-commit "npx lint-staged"
+
+# 添加 commit-msg 钩子
+npx husky add .husky/commit-msg "npx commitlint --edit $1"
+```
+
+在 `package.json` 中添加 lint-staged 配置和 prepare 脚本：
+
+```json
+{
+  "scripts": {
+    "prepare": "husky install"
+  },
+  "lint-staged": {
+    "*.{js,jsx,ts,tsx,vue}": ["eslint --fix"],
+    "*.{css,scss,less,vue}": ["stylelint --fix"]
+  }
+}
+```
+
+> `prepare` 脚本确保其他开发者执行 `npm install` 后自动初始化 husky，无需手动执行。
+
+---
+
+## 参考资料
+
+- [F2ELint GitHub 仓库](https://github.com/alibaba/f2e-spec/tree/main/packages/f2elint)
+- [eslint-config-ali 规则详情](https://www.npmjs.com/package/eslint-config-ali)
+- [stylelint-config-ali 规则详情](https://www.npmjs.com/package/stylelint-config-ali)
+- [阿里巴巴前端规约文档](https://alibaba.github.io/f2e-spec/)
+- [commitlint 官方文档](https://commitlint.js.org/)
+- [husky 官方文档](https://typicode.github.io/husky/)
