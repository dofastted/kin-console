# kin-console

种核管理台（单页 `index.html`）。静态托管，协议和面板 API 都反代到 kin-gateway。

线上：`https://ccmax20.cc`

网关数据面：每槽一个长驻 **Go worker**，经该槽绑定的 **SOCKS5** 访问 Anthropic。控制台只走 `/api/panel`。

## 现在做什么

- 登录后管槽位、凭证、密钥、日志、代理、备份
- 卡片凭证状态：`无凭证` / `可用` / `5h 限制` / `7d 限制` / `Fable 限制` / `不可用`
- 「额度探测」走对应槽 Go worker + SOCKS5（5h / 7d / fable），不是宿主机，也不是 Claude CLI
- 「刷新凭证」调用 worker 的 OAuth refresh（worker 是唯一 refresh owner）
- 设置：粘性、账号池、failover、流式交付（realtime / verified）、请求日志 off/normal/debug
- 日志对齐 Sub2API usage：缓存 token、requested/upstream 模型、首 token、stop_reason、attempts

## 鉴权

- 面板：`POST /api/panel/login`
- 发给用户的 `sk-kin-…` 只能打 `/v1/*`，不能进面板
- 托管密钥落库为 HMAC；面板只显示前缀/后缀

## 契约

- 每个要转发的槽必须绑定 SOCKS5，缺代理 fail closed
- 生产推理只有 Go HTTP worker；`x-kin-forward: cli` / `x-kin-workspace: vm` 已失效
- 默认流式 `realtime`；`x-kin-delivery: verified` 缓冲到 `message_stop`

网关契约：[PANEL_API](https://github.com/dofastted/kin-gateway/blob/main/docs/PANEL_API.md) · [OAUTH](https://github.com/dofastted/kin-gateway/blob/main/docs/OAUTH.md) · [README](https://github.com/dofastted/kin-gateway/blob/main/README.md)
