# kin-console

种核管理台（单页 `index.html`）。静态托管，协议和面板 API 都反代到 kin-gateway。

线上：`https://ccmax20.cc`

## 现在做什么

- 登录后管槽位、凭证、密钥、日志、代理、备份
- 卡片上的凭证状态：有凭证 · **可用（绿）/ 限制（黄）/ 不可用（红）**
- 「额度探测」走虚拟机 UID（5h / 7d / fable），不是宿主机
- 设置里可开关请求日志：关闭 / 普通 / Debug（热更新网关）

## 鉴权

- 面板：`POST /api/panel/login`
- 发给用户的 `sk-kin-…` 只能打 `/v1/*`，不能进面板

网关契约见 [kin-gateway PANEL_API](https://github.com/dofastted/kin-gateway/blob/main/gateway/PANEL_API.md)。
