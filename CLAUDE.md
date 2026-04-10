# CLAUDE.md — 项目编码规范

> 本文档是 Claude Code 的执行规范，所有代码生成、修改、重构行为必须严格遵守以下规则。

---

## 一、AI 执行原则

1. 生成代码前，先阅读当前文件所在模块的已有代码风格，保持一致
2. 修改已有代码时，不擅自改变无关部分的格式和结构
3. 新增文件时，严格遵循项目文件结构规范
4. 发现已有代码违反本规范时，在不影响功能的前提下，顺手修正并说明
5. 每次生成代码后，必须完成底部「自检清单」后再输出

---

## 二、代码规范（Code Style & Quality）

> 目标：保证代码可读、可维护、可复用。

### 2.1 命名规范

| 类型 | 规则 | 示例 |
|------|------|------|
| 变量、函数 | camelCase | `getUserInfo`、`isLoading` |
| 组件、Class、Interface、Type | PascalCase | `UserCard`、`UserProps` |
| 组件文件夹名 | PascalCase | `UserCard/`、`OrderList/` |
| CSS 文件名、非组件 TS 文件名 | kebab-case | `use-pagination.ts`、独立样式文件 `user-card.module.css` |
| 组件内样式文件 | `index.module.css` | 组件文件夹内固定命名，与 `index.tsx` 对应 |
| 全局常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 布尔变量 | is/has/can/should 开头 | `isVisible`、`hasError` |
| 事件回调 Props | on 开头 | `onClick`、`onChange`、`onSubmit` |
| 自定义 Hook | use 开头 | `useUser`、`usePagination` |
| 枚举值 | PascalCase | `OrderStatus.Pending` |

命名语义清晰，禁止使用无意义命名：

```typescript
// ❌ 禁止
const a = getUserInfo()
const data2 = []
function handle() {}

// ✅ 正确
const currentUser = getUserInfo()
const orderList = []
function handleSubmitOrder() {}
```

### 2.2 缩进、格式、分号

- 使用 **ESLint + Prettier** 强制统一风格，以项目配置文件为准
- 缩进：**2 个空格**
- 末尾分号：**必须加**
- 字符串：**单引号**（JSX 属性用双引号）
- 每行最大字符数：**120**
- 对象/数组尾随逗号：**必须加**

### 2.3 TypeScript 规范

- 变量、函数参数、返回值、Props 必须明确类型声明
- **禁止使用 `any`**，必要时使用 `unknown` 并做类型收窄，同时注释说明原因
- **禁止使用 `@ts-ignore`**，必要时使用 `@ts-expect-error` 并注释原因
- 优先使用 `interface` 定义对象类型，`type` 用于联合类型 / 工具类型
- 函数必须显式声明返回类型

```typescript
// ❌ 禁止
function getUser(id: any): any {}
const list: any[] = []

// ✅ 正确
function getUser(id: string): Promise<User> {}
const list: User[] = []
```

- 枚举优先使用联合类型替代普通 enum

```typescript
// ❌ 普通 enum（会生成额外运行时代码）
enum Status { Active, Inactive }

// ✅ 推荐
type Status = 'active' | 'inactive'

// ✅ 或使用 const enum（注意：Vite + SWC / esbuild 对跨文件 const enum 支持有限，建议仅在同文件内使用）
const enum Status { Active = 'active', Inactive = 'inactive' }
```

- 泛型命名语义化，单个泛型用 `T`，多个用语义化名称

```typescript
// ❌ 禁止
function merge<T, U, V>() {}

// ✅ 正确
function merge<TSource, TTarget>() {}
```

### 2.4 函数与逻辑设计

- 函数单一职责，**不超过 50 行**
- 避免超过 **3 层**的嵌套逻辑，使用提前返回（Early Return）

```typescript
// ❌ 禁止：深层嵌套
function processOrder(order: Order) {
  if (order) {
    if (order.status === 'paid') {
      if (order.items.length > 0) {
        // 处理逻辑
      }
    }
  }
}

// ✅ 正确：提前返回
function processOrder(order: Order) {
  if (!order) return;
  if (order.status !== 'paid') return;
  if (order.items.length === 0) return;
  // 处理逻辑
}
```

- 避免多重三元操作符，超过 2 层改用 `if/else` 或策略对象

```typescript
// ❌ 禁止
const label = status === 'a' ? 'A' : status === 'b' ? 'B' : status === 'c' ? 'C' : 'Unknown'

// ✅ 正确
const labelMap: Record<string, string> = { a: 'A', b: 'B', c: 'C' }
const label = labelMap[status] ?? 'Unknown'
```

- 禁止魔法值，所有硬编码数字 / 字符串必须定义为常量

```typescript
// ❌ 禁止
if (retryCount > 3) {}
if (type === 'admin') {}

// ✅ 正确
const MAX_RETRY_COUNT = 3
const USER_TYPE_ADMIN = 'admin'
if (retryCount > MAX_RETRY_COUNT) {}
```

### 2.5 异步与错误处理

- 所有异步函数必须处理异常，使用 `try/catch`
- **禁止空 catch**，必须处理或上报错误
- **禁止**组件内直接调用 `fetch` / `axios`，统一通过封装的 API 层调用
- Promise 不允许悬空（no floating promises）

```typescript
// ❌ 禁止：未捕获异常
async function loadUser() {
  const data = await fetchUser() // 异常会向上抛出，调用方无感知
  return data
}

// ❌ 禁止：空 catch
try {
  await fetchUser()
} catch (e) {}

// ✅ 正确
async function loadUser() {
  try {
    const data = await fetchUser()
    return data
  } catch (error) {
    console.error('[fetchUser] 请求失败:', error)
    // 或上报错误监控
    throw error
  }
}
```

### 2.6 注释规范

- 函数、复杂逻辑、接口调用必须添加注释
- **JSDoc 注释**用于公共函数 / 组件，面向调用方，描述「是什么、参数含义、返回值」

```typescript
/**
 * 根据用户 ID 获取用户信息
 * @param userId - 用户 ID
 * @param options - 可选配置项
 * @returns 用户信息对象
 */
async function getUserById(userId: string, options?: FetchOptions): Promise<User> {}
```

- **行内注释**用于函数内部，说明「为什么」而不是「是什么」

```typescript
// ❌ 无意义注释
// 将 count 加 1
count++

// ✅ 有价值注释
// 后端分页从 1 开始，需要 +1 转换
const page = currentIndex + 1
```

- 临时代码 / 待处理逻辑必须标注 TODO / FIXME

```typescript
// TODO: 等后端接口联调完成后，移除 mock 数据
// FIXME: 当列表为空时，此处会抛出异常，待修复
```

---

## 三、复用与重复代码规范（DRY Principle）

> 目标：避免相同逻辑分散在多处，降低维护成本。

### 3.1 封装判断标准

- 相同或高度相似的代码出现 **2 次及以上**，必须考虑封装
- 仅用一次的逻辑不强行封装，避免过度设计
- 封装前先判断类型，选择对应封装方式：

```
相同代码出现 2 次以上
        ↓
判断复用类型
  ├── 有状态逻辑 / 副作用   → 自定义 Hook
  ├── UI 结构相似           → 组件 + Props
  └── 无状态纯函数          → utils 工具函数
```

### 3.2 逻辑复用 → 自定义 Hook

适用：多个组件有相同的状态管理、副作用、数据请求逻辑

```typescript
// ❌ 禁止：多个组件各自重复相同请求逻辑
// ComponentA.tsx
const [user, setUser] = useState<User | null>(null)
useEffect(() => {
  fetchUser().then(setUser)
}, [])

// ComponentB.tsx —— 完全相同的逻辑再写一遍
const [user, setUser] = useState<User | null>(null)
useEffect(() => {
  fetchUser().then(setUser)
}, [])

// ✅ 正确：抽成 Hook
// hooks/useUser.ts
export function useUser() {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<Error | null>(null)

  useEffect(() => {
    setLoading(true)
    fetchUser()
      .then(setUser)
      .catch(setError)
      .finally(() => setLoading(false))
  }, [])

  return { user, loading, error }
}
```

### 3.3 UI 复用 → 组件 + Props

适用：多处渲染结构相似、只有数据或样式略有不同的 UI

```typescript
// ❌ 禁止：结构相同的组件各写一份
// SuccessCard.tsx / ErrorCard.tsx / WarningCard.tsx

// ✅ 正确：抽成通用组件，通过 Props 控制差异
// components/common/StatusCard/index.tsx
interface StatusCardProps {
  type: 'success' | 'error' | 'warning'
  title: string
  description?: string
  onAction?: () => void
  actionLabel?: string
}

export function StatusCard({
  type,
  title,
  description,
  onAction,
  actionLabel = '确认',
}: StatusCardProps) {
  // ...
}
```

- Props 数量超过 **7 个**时，考虑拆分组件或使用 Context

### 3.4 纯逻辑复用 → utils 工具函数

适用：格式化、计算、校验等无状态纯函数

```typescript
// utils/format.ts
export function formatPrice(price: number, currency = 'CNY'): string {
  return new Intl.NumberFormat('zh-CN', { style: 'currency', currency }).format(price)
}

// utils/validators.ts
export function isValidPhone(phone: string): boolean {
  return /^1[3-9]\d{9}$/.test(phone)
}
```

### 3.5 封装位置规范

```
src/
├── components/
│   ├── ui/          # 与业务无关的基础 UI 组件（Button、Modal、Input）
│   └── common/      # 跨模块复用的业务组件
├── hooks/           # 跨模块复用的自定义 Hook
├── utils/           # 工具函数
└── modules/
    └── order/       # 业务模块
        ├── components/  # 仅在 order 模块内复用的组件
        └── hooks/       # 仅在 order 模块内复用的 Hook
```
### 3.7 第三方库引入标准

引入新依赖前，必须评估以下维度：

| 维度 | 要求 |
|------|------|
| Bundle 大小 | 优先选择轻量替代方案；新增依赖导致 gzip 体积增加 > 10 KB 需说明理由 |
| 维护状态 | npm 最近 6 个月有更新，GitHub Issues 有响应 |
| License | MIT / Apache-2.0 / BSD，商业项目禁止引入 GPL 依赖 |
| TypeScript 支持 | 必须有官方 `@types` 或内置类型定义 |
| 按需引入 | 支持 Tree Shaking 或按需导入（见 §5.1） |

```bash
# 引入前用 bundlephobia 评估包体积
# https://bundlephobia.com/package/<package-name>
```

### 3.6 Props 传递规范

#### 核心原则
组件依赖某个数据对象的多个字段时，
直接传入整个对象，字段处理逻辑收口到组件内部，
避免字段名散落在所有调用处。

```tsx
// ❌ 禁止：字段处理散落在调用处
// 一旦字段改名，所有调用处都要修改
<Card title={item.name} desc={item.desc} img={item.imgUrl} />
<Card title={item.name} desc={item.desc} img={item.imgUrl} />
<Card title={item.name} desc={item.desc} img={item.imgUrl} />

// ✅ 正确：传整个对象，Card 内部统一处理字段
// 字段改名只需改 Card 内部一处
<Card item={item} />
<Card item={item} />
<Card item={item} />

// Card 组件内部
function Card({ item }: CardProps) {
  return (
    <div>
      <img src={item.imgUrl} />
      <h3>{item.name}</h3>
      <p>{item.desc}</p>
    </div>
  )
}
```

#### 例外情况

满足以下**任意一条**时，单独传 props，不传整个对象：
1. 组件是与业务无关的基础 UI 组件（Button、Avatar、Modal 等）
2. 组件仅依赖对象中的 **1～2 个字段**，且无业务映射逻辑（如字段改名、状态转换）

```tsx
// ✅ 基础 UI 组件，单独传 props（与业务数据结构解耦）
<Button label="提交" onClick={handleSubmit} />
<Avatar src={item.imgUrl} size="md" />

// ✅ 仅用 1 个字段，无业务逻辑
<Divider label={section.title} />

// ❌ 依赖 3 个以上字段，应传整个对象
<ProductCard name={product.name} price={product.price} img={product.imgUrl} stock={product.stock} />
// ✅ 改为
<ProductCard item={product} />
```

---

## 四、组件规范（Component & UI）

> 目标：保证组件可维护、可复用、风格一致。

### 4.1 文件结构

每个组件独立文件夹：

```
ComponentName/
├── index.tsx            # 组件主文件
├── index.module.css     # 局部样式
├── types.ts             # 类型定义（可选）
└── hooks.ts             # 组件专属 Hook（可选，仅在组件内部有复杂状态/副作用逻辑时创建）
```

组件文件内部代码顺序：

```typescript
// 1. Imports
import React, { useState, useCallback } from 'react'

// 2. Types / Interfaces
interface UserCardProps {}

// 3. 常量定义
const DEFAULT_AVATAR = '/images/default-avatar.png'

// 4. 组件函数
export function UserCard(props: UserCardProps) {
  // 4.1 Hooks
  // 4.2 派生状态 / 计算值
  // 4.3 事件处理函数
  // 4.4 副作用
  // 4.5 渲染
}

// 5. 子组件（如有）
// 6. export default
```

### 4.2 Props & State

- 所有 Props 必须定义 TypeScript 类型，明确必填 / 可选项
- Props 提供合理默认值，避免 `undefined` 错误
- **禁止**组件内部直接修改 props
- **禁止**直接操作 DOM，通过 `ref` 或状态管理处理

```typescript
// ❌ 禁止
function UserCard(props) {} // 无类型
props.name = 'new name'    // 直接修改 props

// ✅ 正确
interface UserCardProps {
  userId: string             // 必填
  size?: 'sm' | 'md' | 'lg' // 可选，有枚举约束
  onSelect?: (id: string) => void
}

function UserCard({ userId, size = 'md', onSelect }: UserCardProps) {}
```

### 4.3 渲染规范

- 组件拆分小组件，单个组件 JSX **不超过 100 行**
- 列表渲染必须加稳定的 `key`（禁止用 index 作为 key，数据有唯一 ID 时）

```typescript
// ❌ 禁止：用 index 作为 key（列表有增删时会出现 Bug）
{list.map((item, index) => <Item key={index} {...item} />)}

// ✅ 正确
{list.map(item => <Item key={item.id} {...item} />)}
```

- 避免在 render 中创建新对象 / 数组作为 props（每次渲染引用不同，导致子组件重渲染）

```typescript
// ❌ 禁止
<Chart config={{ color: 'red' }} />        // 每次渲染新对象
<List filter={['a', 'b']} />               // 每次渲染新数组

// ✅ 正确
const chartConfig = useMemo(() => ({ color: 'red' }), [])
<Chart config={chartConfig} />
```

### 4.4 性能优化

按需使用，不过度优化：

| Hook | 使用场景 |
|------|---------|
| `React.memo` | 组件 props 变化频率低 且 渲染成本高 |
| `useMemo` | 计算成本高的派生数据（排序、过滤大列表等） |
| `useCallback` | 回调函数作为 props 传入子组件时 |

### 4.5 样式规范

- 组件样式必须局部化，使用 **CSS Modules** / Styled Components / LESS Scoped
- **禁止**在组件内写全局 CSS
- 颜色、间距、字体大小统一使用设计变量（CSS Variables / Design Tokens）

```css
/* ❌ 禁止：全局污染 */
.button { color: #1890ff; }

/* ✅ 正确：CSS Modules 局部作用域 */
/* button.module.css */
.button { color: var(--color-primary); }
```

### 4.6 状态管理选型

按以下优先级选择状态管理方案，避免过度引入全局状态：

```
状态作用范围
  ├── 单个组件内部                → useState / useReducer
  ├── 父子组件间（层级 ≤ 2）      → Props 传递
  ├── 跨多层组件 / 同模块共享     → React Context + useReducer
  ├── 跨模块 / 全局异步数据       → React Query / SWR（服务端状态）
  └── 全局客户端状态（购物车、权限等）→ Zustand / Redux Toolkit
```

- **禁止**将服务端数据（接口返回值）存入 Redux/Zustand；使用 React Query 或 SWR 管理
- Context 仅用于低频更新数据（主题、语言、用户信息），高频更新会导致全树重渲染

---

## 五、性能规范（Performance）

> 目标：提高页面加载速度和渲染效率。

### 5.1 资源加载
- 第三方依赖按需引入，禁止全量引入大型库

```typescript
// ❌ 禁止：全量引入
import _ from 'lodash'
import * as Icons from '@ant-design/icons'

// ✅ 正确：按需引入
import debounce from 'lodash/debounce'
import { UserOutlined } from '@ant-design/icons'
```

### 5.2 渲染优化

- 减少无用 DOM 层级，避免深层嵌套
- 高频触发的事件（scroll、resize、input）必须使用**防抖 / 节流**

```typescript
// ✅ 搜索输入防抖
const handleSearch = useMemo(
  () => debounce((value: string) => fetchSearch(value), 300),
  []
)
```

- 减少不必要的状态更新，相关状态合并管理

### 5.3 网络优化

- 列表接口必须**分页**，禁止一次性请求全量数据
- 相同接口数据做缓存（内存缓存 / localStorage / SWR / React Query）
- 接口并发请求使用 `Promise.all`，避免串行等待

```typescript
// ❌ 禁止：串行请求（且缺少 try/catch）
const user = await fetchUser()
const orders = await fetchOrders()

// ✅ 正确：并发请求 + 异常处理
try {
  const [user, orders] = await Promise.all([fetchUser(), fetchOrders()])
} catch (error) {
  console.error('[loadPageData] 请求失败:', error)
  throw error
}
```

---

## 六、安全规范（Security）

> 目标：防范常见前端安全漏洞。

- **禁止**在前端代码中硬编码密钥、Token、密码，统一使用环境变量
- **禁止**直接使用 `dangerouslySetInnerHTML`，必须先做 XSS 过滤
- 用户输入必须做校验，不直接拼接到 URL / HTML 中
- 敏感信息（Token、用户隐私数据）不写入 `localStorage`，使用 `httpOnly Cookie`；临时会话数据可用 `sessionStorage`，但不存敏感凭据
- 环境变量统一命名前缀区分公私（如 `NEXT_PUBLIC_` / `VITE_`）
- CSRF 防护：接口使用 `SameSite=Strict/Lax` Cookie，避免用 GET 请求执行写操作；敏感操作可附加 CSRF Token

```typescript
// ❌ 禁止
const API_KEY = 'sk-xxxxxxxxxxxxxxxx'
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// ✅ 正确
const API_KEY = process.env.VITE_API_KEY
import DOMPurify from 'dompurify'
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />
```

---

## 七、无障碍访问规范（Accessibility）

> 目标：确保产品对所有用户（包括使用辅助技术的用户）可用，遵循 WCAG 2.1 AA 标准。

- 所有交互元素（按钮、链接、表单）必须有语义化标签或 `aria-label`
- 图片必须有 `alt` 属性；装饰性图片使用 `alt=""`
- 表单控件必须关联 `<label>`（`htmlFor` 或包裹式）
- 键盘可导航：所有功能必须可通过 Tab / Enter / Space / Esc 操作，禁止仅依赖鼠标事件
- 颜色对比度：正文文字对比度 ≥ 4.5:1，大号文字 ≥ 3:1

```tsx
// ❌ 禁止：无语义，屏幕阅读器无法识别
<div onClick={handleDelete}>删除</div>

// ✅ 正确：语义化 + 键盘支持
<button type="button" aria-label="删除订单" onClick={handleDelete}>
  删除
</button>

// ✅ 表单关联 label
<label htmlFor="email">邮箱</label>
<input id="email" type="email" />

// ✅ 图片 alt
<img src={product.imgUrl} alt={product.name} />
<img src={decorativeBanner} alt="" aria-hidden="true" />
```

---

## 八、工程化规范（Tooling & Engineering）

> 目标：保证团队协作和项目可维护。

### 7.1 代码检查

- ESLint + Prettier 强制统一风格，以项目根目录配置文件为准
- Git Hooks（husky + lint-staged）提交前自动校验
- 提交信息使用 **Conventional Commits** 规范

### 7.2 Git 提交规范

格式：`<type>(<scope>): <subject>`

| type | 说明 |
|------|------|
| `feat` | 新功能 |
| `fix` | Bug 修复 |
| `refactor` | 重构（不影响功能） |
| `style` | 代码格式调整 |
| `docs` | 文档更新 |
| `test` | 测试相关 |
| `chore` | 构建 / 工具链变更 |
| `perf` | 性能优化 |

```bash
# ✅ 示例
feat(auth): 新增手机号登录功能
fix(cart): 修复数量为 0 时仍可提交的问题
refactor(user): 将用户信息请求逻辑抽成 useUser Hook
perf(list): 对商品列表搜索添加防抖优化
```

### 7.3 错误监控

- 生产环境异常必须上报至错误监控平台（如 Sentry）
- **禁止**在生产环境使用 `console.error` 替代错误上报
- 上报时附带必要上下文（用户 ID、页面路径、操作链路）

```typescript
// ✅ 正确：封装统一上报函数，组件内调用
import { captureException } from '@/utils/monitor'

try {
  const data = await fetchOrder(orderId)
} catch (error) {
  // 开发环境打印，生产环境上报
  captureException(error, { context: 'fetchOrder', orderId })
  throw error
}
```

- 全局未捕获异常通过 `ErrorBoundary`（React）和 `window.onerror` 兜底捕获

---

## 九、协作与文档规范（Collaboration & Docs）

> 目标：保证团队成员高效协作、知识可传承。
- 公共组件需注释说明用途、Props 含义、使用示例
- 使用 `TODO` / `FIXME` 标注待处理问题，格式：`// TODO(@author): 说明`
- 新增公共 Hook / 工具函数，同步更新对应文档或 README

---

## 十、AI 自检清单

> 每次生成或修改代码后，必须逐项确认：

### 代码质量
- [ ] 是否有未明确类型的变量或函数（any / 缺少返回类型）
- [ ] 是否有魔法值（硬编码数字 / 字符串）未定义为常量
- [ ] 是否有超过 3 层的嵌套逻辑未使用提前返回优化
- [ ] 是否有超过 50 行的函数需要拆分

### 异步与错误
- [ ] 是否有未捕获的异步异常（缺少 try/catch）
- [ ] 是否有空 catch 块
- [ ] 是否有悬空 Promise（未 await 也未 .catch）

### 组件与复用
- [ ] 当前逻辑在项目中是否已有类似实现，可复用而非重写
- [ ] 列表渲染是否都有稳定的 key
- [ ] 是否有在 render 中创建新对象 / 数组作为 props
- [ ] 相同代码是否出现 2 次以上，是否需要封装为 Hook / 组件 / utils
- [ ] 同一个对象的多个字段是否散落在多处调用，
      若是则改为传整个对象，字段处理收口到组件内部
- [ ] 是否存在重复的 CSS 类定义，可提取为公共样式变量或 class

### 状态管理
- [ ] 状态作用范围是否与选型匹配（组件内 / Context / React Query / Zustand）
- [ ] 服务端数据是否错误存入了全局状态库（应使用 React Query / SWR）
- [ ] Context 是否用于高频更新数据（会导致全树重渲染）

### 无障碍
- [ ] 交互元素是否有语义化标签或 `aria-label`
- [ ] 图片是否有 `alt` 属性
- [ ] 表单控件是否关联了 `<label>`
- [ ] 功能是否可通过键盘操作（Tab / Enter / Esc）

### 第三方库
- [ ] 新增依赖是否评估了包体积、维护状态、License
- [ ] 是否有现有库可复用，避免重复引入功能相似的包

### 安全
- [ ] 是否有硬编码的密钥 / Token
- [ ] 是否有未过滤的用户输入直接渲染到 HTML
- [ ] 生产环境异常是否通过监控平台上报（非 console.error）

### 注释与可读性
- [ ] 复杂逻辑是否有注释说明
- [ ] 新增的公共函数 / 组件是否有 JSDoc 注释
- [ ] 是否有未处理的 TODO / FIXME 需要标注

---

*本文档持续维护，规范变更需团队评审后更新。*