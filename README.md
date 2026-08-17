# kin-console

种核 KIN 管理台（单页 HTML）。

## 定位

- 对话 / 模型 / 用量：**只经虚拟机官方 Claude Code**
- sessionKey：仅用于导入与恢复；续期由 CLI 负责
- 探测：`claude auth status` + 官方 hop 的 `rate_limit_event`
- 模型列表：CLI 二进制目录，非 Anthropic HTTP

## 部署

静态托管（Vercel 等）。`API Base` 指向 gateway（本站可留空；异域默认 `https://kin.fkcodex.com`）。

登录：面板管理员账号 / 密码。
