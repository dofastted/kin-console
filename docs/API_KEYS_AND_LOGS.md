# API Key 与请求日志接口说明

面向 **kin-console / 管理台前端** 与客户端集成。  
网关基址示例：`https://kin.fkcodex.com`（本地可留空，相对路径）。

> 权限约定  
> - **管理接口**（`/api/panel/*`）：仅 **面板登录会话** 或 **Master `KIN_API_KEY`**  
> - **托管 API Key**（`sk-kin-…`）：仅可调用协议转发（`/v1/messages`、`/v1/chat/completions` 等），**不能**访问面板

---

## 1. 鉴权方式

### 1.1 面板登录（管理台）

```http
POST /api/panel/login
Content-Type: application/json

{ "username": "admin", "password": "…" }
```

成功响应：

```json
{
  "ok": true,
  "token": "kin-panel-…",
  "user": "admin",
  "expires_in": 604800
}
```

后续管理请求携带其一：

| 方式 | Header / Cookie |
|------|-----------------|
| Bearer | `Authorization: Bearer kin-panel-…` |
| 自定义头 | `X-Panel-Token: kin-panel-…` |
| Cookie | 登录接口下发的 session cookie |

### 1.2 Master API Key

环境变量 `KIN_API_KEY`。可调协议接口，也可当管理身份调 `/api/panel/*`。

```http
Authorization: Bearer <KIN_API_KEY>
```

### 1.3 托管 API Key（发给终端用户）

面板生成，前缀 `sk-kin-`。**仅协议接口**。

```http
Authorization: Bearer sk-kin-…
# 或
X-Api-Key: sk-kin-…
```

---

## 2. API Key 管理接口

基路径：`/api/panel/api-keys`  
**需要**：面板 token 或 Master key。

### 2.1 列表

```http
GET /api/panel/api-keys
```

```json
{
  "ok": true,
  "count": 1,
  "keys": [
    {
      "id": "key_a1b2c3d4e5f6",
      "name": "client-a",
      "key": "sk-kin-ab…90ab",
      "key_prefix": "sk-kin-ab",
      "status": "active",
      "max_concurrency": 2,
      "quota_requests": 1000,
      "quota_used": 12,
      "rpm": 60,
      "expires_at": null,
      "created_at": "2026-08-18T00:00:00.000Z",
      "updated_at": "2026-08-18T00:00:00.000Z",
      "last_used_at": null,
      "requests": 12,
      "tokens_in": 0,
      "tokens_out": 0,
      "inflight": 0
    }
  ]
}
```

说明：列表中的 `key` 为**脱敏**片段，无法还原完整密钥。

### 2.2 生成

```http
POST /api/panel/api-keys
Content-Type: application/json

{
  "name": "client-a",
  "max_concurrency": 2,
  "quota_requests": 1000,
  "rpm": 60,
  "expires_in_days": 30
}
```

| 字段 | 类型 | 默认 | 说明 |
|------|------|------|------|
| `name` | string | `default` | 展示名 |
| `max_concurrency` | number | `2` | 同时进行中的请求数；`0` = 不限制 |
| `quota_requests` | number | `0` | 累计请求额度；`0` = 不限制 |
| `rpm` | number | `0` | 滑动窗口每分钟请求数；`0` = 不限制 |
| `expires_in_days` | number | — | 从现在起 N 天后过期 |
| `expires_at` | ISO string | — | 显式过期时间（优先于 days） |
| `key` / `custom_key` | string | 自动生成 | 自定义明文 key（可选） |

成功 `201`：

```json
{
  "ok": true,
  "item": {
    "id": "key_…",
    "name": "client-a",
    "key": "sk-kin-<完整明文，仅此一次>",
    "status": "active",
    "max_concurrency": 2,
    "quota_requests": 1000,
    "quota_used": 0,
    "rpm": 60,
    "expires_at": "2026-09-17T00:00:00.000Z",
    "…"
  },
  "note": "key is shown only once; store it securely"
}
```

> **前端必须立刻展示并引导复制。** 之后列表只返回脱敏值。

冲突：`409`（`key_exists`）。参数错误：`400`。

### 2.3 更新

```http
PATCH /api/panel/api-keys/:id
Content-Type: application/json

{
  "name": "renamed",
  "status": "disabled",
  "max_concurrency": 4,
  "quota_requests": 5000,
  "rpm": 120,
  "expires_at": null
}
```

`status`：`active` | `disabled`。

### 2.4 清零额度

```http
POST /api/panel/api-keys/:id/reset-quota
```

将 `quota_used` 置 `0`。

### 2.5 删除

```http
DELETE /api/panel/api-keys/:id
```

立即失效，客户端再用该 key 会 `401`。

### 2.6 客户端调用示例（协议）

```http
POST /v1/messages
Authorization: Bearer sk-kin-…
Content-Type: application/json

{
  "model": "claude-haiku-4-5-20251001",
  "max_tokens": 1024,
  "messages": [{ "role": "user", "content": "hi" }]
}
```

超额 / 限流响应示例：

| HTTP | 含义 |
|------|------|
| `401` | key 无效或已删除 |
| `403` | 停用 / 过期 / 试图访问 panel |
| `429` | 并发打满 / 额度用尽 / RPM 超限 |

---

## 3. 请求日志接口

基路径：`/api/panel/request-logs`  
**需要**：面板 token 或 Master key。

服务端默认模式由环境变量控制：

| 环境变量 | 值 | 含义 |
|----------|-----|------|
| `KIN_REQUEST_LOG_MODE` | `normal`（默认） | 只写摘要 JSONL，不含 body |
| | `debug` | 额外写全量脱敏记录 |
| | `off` | 不写日志 |
| `KIN_REQUEST_LOG_RETAIN_DAYS` | `7` | 保留天数 |
| `KIN_REQUEST_LOG_DEBUG_CHARS` | `200000` | debug body 最大字符 |

单次请求覆盖（任意客户端均可带，影响该次落盘级别）：

```http
X-Kin-Debug: 1
# 或
X-Kin-Log: debug | normal | off
```

所有协议响应会带：

```http
X-Request-ID: <uuid 或入站原样>
```

### 3.1 列表 — 普通模式

```http
GET /api/panel/request-logs?mode=normal&limit=50
```

```json
{
  "ok": true,
  "mode": "normal",
  "config": {
    "mode": "normal",
    "retain_days": 7,
    "recent_normal": 12,
    "today_file": "…/request-logs/2026-08-18.jsonl"
  },
  "items": [
    {
      "id": "log_…",
      "request_id": "a1b2c3d4-…",
      "ts": "2026-08-18T04:00:00.000Z",
      "log_mode": "normal",
      "method": "POST",
      "path": "/v1/messages",
      "protocol": "anthropic.messages",
      "model": "claude-haiku-4-5-20251001",
      "stream": false,
      "status": 200,
      "duration_ms": 320,
      "api_key_kind": "managed",
      "api_key_id": "key_…",
      "vm_id": "vm-01",
      "account_id": "…",
      "workspace": "client",
      "input_tokens": null,
      "output_tokens": null,
      "error_code": null,
      "error_message": null,
      "user_agent": "…",
      "ip": "…",
      "has_tools": false
    }
  ]
}
```

普通模式**不含** `inbound_body` / headers 原文。

### 3.2 列表 — Debug 模式

```http
GET /api/panel/request-logs?mode=debug&limit=20
```

每条可能额外包含：

- `headers`（`authorization` / `x-api-key` 等已脱敏）
- `inbound_summary`（消息数、tools 数、roles…）
- `inbound_body`（脱敏后的请求体；过长会 `_truncated`）
- `hop_meta`
- `upstream_status`
- `outbound_summary` / `outbound_body`（若有）

### 3.3 单条 Debug 详情

```http
GET /api/panel/request-logs/:requestId
```

```json
{
  "ok": true,
  "item": { "request_id": "…", "inbound_body": { … }, "…" }
}
```

不存在：`404`。

### 3.4 前端展示建议

| 场景 | 建议 |
|------|------|
| 默认列表 | `mode=normal`，表格：时间 / 协议 / 模型 / 状态 / 耗时 / Key / VM |
| 排障 | 切 `mode=debug`，点「详情」拉 `:requestId` |
| 复制追踪 | 展示 `X-Request-ID` / `request_id` 前 8 位，悬停全文 |
| 错误行 | `status >= 400` 高亮，副文案显示 `error_code` |

### 3.5 前端请求示例（TypeScript）

```ts
const base = '' // 或 'https://kin.fkcodex.com'
const headers = {
  Authorization: `Bearer ${panelToken}`,
  'Content-Type': 'application/json',
}

// 密钥列表
const keys = await fetch(`${base}/api/panel/api-keys`, { headers }).then(r => r.json())

// 生成密钥 — 务必保存 r.item.key
const created = await fetch(`${base}/api/panel/api-keys`, {
  method: 'POST',
  headers,
  body: JSON.stringify({
    name: 'client-a',
    max_concurrency: 2,
    quota_requests: 1000,
    rpm: 60,
    expires_in_days: 30,
  }),
}).then(r => r.json())
console.log('one-time key:', created.item?.key)

// 普通日志
const logs = await fetch(`${base}/api/panel/request-logs?mode=normal&limit=50`, { headers })
  .then(r => r.json())

// Debug 详情
const detail = await fetch(`${base}/api/panel/request-logs/${requestId}`, { headers })
  .then(r => r.json())
```

---

## 4. 落盘路径（运维参考）

| 模式 | 路径 |
|------|------|
| 普通 | `data/request-logs/YYYY-MM-DD.jsonl` |
| Debug | `data/request-logs/debug/YYYY-MM-DD/<request_id>.json` |
| API Keys | `data/api-keys.json`（权限 `0600`） |

密钥与 Authorization 在 debug 记录中一律 `***REDACTED***`。

---

## 5. 错误码速查

| code | HTTP | 场景 |
|------|------|------|
| `missing_api_key` | 401 | 未带凭证 |
| `invalid_api_key` | 401 | key 错误/已删 |
| `api_key_disabled` | 403 | 停用 |
| `api_key_expired` | 403 | 过期 |
| `api_key_quota_exhausted` | 429 | 额度用尽 |
| `api_key_rate_limit` | 429 | RPM |
| `api_key_concurrency` | 429 | 并发打满 |
| `forbidden` | 403 | 托管 key 访问 panel |
