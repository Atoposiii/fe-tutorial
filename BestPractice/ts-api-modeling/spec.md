## ADDED Requirements
### 定义 API 文件的导出结构
- **WHEN** 编写一个 API 封装文件
- **THEN** `RequestParams` / `RequestBody` 不加 `export`（调用方无需感知入参结构细节）
- **AND** `Response` interface 加 `export`（业务层需要引用响应类型做进一步处理）
- **AND** API 函数加 `export`（service 层通过 import 调用）

### Requirement: 为 API 函数定义请求和响应接口
每个 API 调用函数，SHALL 定义独立的 Request interface 和 Response interface，明确入参和出参类型。

重要程度：**核心**

```typescript
// APIs/GET_Le_insurance_merchant_insuranceurl_api.ts
interface RequestParams {
  type: number
  dpshopids?: string
  recordid?: number
}

export interface InsuranceUrlResponse {
  url: string
}

function MRN_GET_Le_insurance_merchant_insuranceurl_api(
  params: RequestParams
): Promise<InsuranceUrlResponse> {
  return mapi.get('/le/insurance/merchant/insuranceurl', params)
}
```

#### Scenario: 定义 GET 接口的类型
- **WHEN** 封装一个 GET 请求函数
- **THEN** 定义 RequestParams interface（query 参数）和 Response interface（响应数据），函数返回 `Promise<Response>`

---

### Requirement: 使用 Promise<T> 标注异步函数返回类型
所有返回 Promise 的异步函数，SHALL 显式标注 `Promise<T>` 返回类型，明确 resolve 值的类型。

重要程度：**核心**

```typescript
export function fetchSigningShops(
  params: FetchSigningShopsParams
): Promise<SigningShopResponse.t> {
  return MRN_GET_Le_insurance_merchant_signingshop_api(params)
}
```

---

### Requirement: GET 与 POST 入参接口命名规范
GET 请求的入参 interface SHALL 命名为 `RequestParams`（对应 query 参数）；POST 请求的入参 interface SHALL 命名为 `RequestBody`（对应 request body）。语义更清晰，便于区分请求方式。

重要程度：**核心**

```typescript
// ✅ GET 请求 — 入参命名为 RequestParams
interface RequestParams {
  type: number
  dpshopids?: string
}

function MRN_GET_Le_insurance_merchant_insuranceurl_api(
  params: RequestParams
): Promise<InsuranceUrlResponse> {
  return mapi.get('/le/insurance/merchant/insuranceurl', params)
}

// ✅ POST 请求 — 入参命名为 RequestBody
interface RequestBody {
  shopId: string
  insuranceType: number
}

function MRN_POST_Le_insurance_merchant_bind_api(
  body: RequestBody
): Promise<BindResponse> {
  return mapi.post('/le/insurance/merchant/bind', body)
}
```

#### Scenario: 封装 GET 或 POST 请求时命名入参接口
- **WHEN** 封装一个 GET 请求函数
- **THEN** 入参 interface 命名为 `RequestParams`
- **WHEN** 封装一个 POST 请求函数
- **THEN** 入参 interface 命名为 `RequestBody`

---

### Requirement: 可选参数与必填参数的精确区分
定义 Request interface 时，SHALL 用 `?` 精确标记真正可选的字段，必填字段不加 `?`。避免将所有字段都标为可选，导致调用方漏传必要参数时无法在编译期发现错误。

重要程度：**核心**

```typescript
// ❌ 错误：将必填字段也标为可选，编译器无法保护调用方
interface RequestParams {
  type?: number       // type 实际上是必填的
  dpshopids?: string
  recordid?: number
}

// ✅ 推荐：必填字段不加 ?，真正可选的字段才加 ?
interface RequestParams {
  type: number        // 必填，不加 ?
  dpshopids?: string  // 可选，加 ?
  recordid?: number   // 可选，加 ?
}
```

#### Scenario: 定义 API 入参接口时区分必填与可选字段
- **WHEN** 定义 RequestParams 或 RequestBody interface
- **THEN** 与后端接口文档对齐，必填字段不加 `?`，可选字段加 `?`，确保编译期能发现漏传必填参数的问题

---

### Requirement: Request/Response 类型与函数的导出规范
`RequestParams` / `RequestBody` 是 API 函数的实现细节，SHALL 不导出（file-private）；`Response` 类型供业务层（service / hook）使用，SHALL 导出；API 封装函数本身 SHALL 导出，供 service 层调用。

重要程度：**核心**

```typescript
// ✅ Request 类型只在当前文件使用，不导出
interface RequestParams {
  type: number
  dpshopids?: string
  recordid?: number
}

// ✅ Response 类型供业务层使用，应导出
export interface InsuranceUrlResponse {
  url: string
}

// ✅ 函数本身导出，供 service 层调用
export function MRN_GET_Le_insurance_merchant_insuranceurl_api(
  params: RequestParams
): Promise<InsuranceUrlResponse> {
  return mapi.get('/le/insurance/merchant/insuranceurl', params)
}
```