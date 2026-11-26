# Verboo Code Router for Magpie & OpenCode

[English](README.md)

这个供应商插件把 [Verboo Code](https://code.verboo.ai) 官方 Router（`https://code.verboo.ai/router/v1`）接入 [Magpie](https://usemagpie.ai) 与 OpenCode（1.x 与 2.x）。支持浏览器 PKCE OAuth 登录与 API Key 认证，按账号动态加载可用模型、思考档位与订阅用量，并将对话请求转发到 Verboo 的 Chat Completions 接口，通过 Magpie 返回普通 JSON 或 SSE 流式响应。

## 安装和使用

### 通过 GitHub 安装（Magpie）

```sh
magpie plugin add github:lizimu0/magpie-verboo
```

或在 Magpie 的 **插件 → 添加插件** 中填写仓库地址 / 选择本文件夹。

### 登录与测试

1. 在 Magpie 的 **供应商 → Verboo Code Router** 选择 **浏览器登录**（OAuth），或填写 [Verboo API Key](https://code.verboo.ai/en/api-keys)（`vbk_...`），也可以在终端运行：
   ```sh
   magpie plugin login verboo
   ```
2. 等模型列表加载后，在 Magpie 中选模型，或运行：
   ```sh
   magpie provider test verboo
   ```
   通过 Magpie 网关调用时，模型名为 `verboo/<上游模型 ID>`。
3. 查看账号的订阅计划、续费周期、Stamina 余量与试用额度：
   ```sh
   magpie quota
   ```

## 请求、认证与用量

- **浏览器登录与轮换令牌保护**：使用与桌面版 Verboo Code 相同的 OAuth 2.1 PKCE 流程——本机 `127.0.0.1` 临时端口接收授权回调，向 `https://code.verboo.ai/oauth/token` 换取令牌，并从 `/api/me` 读取账号邮箱与用户标识（`uid`）以支持多账号区分与去重。机器的安装标识保存在 `~/.magpie-verboo/installation-id`（`0600`）。插件已声明 `rotating: true`，并发与跨进程轮换时自动去重并从最新落盘凭据恢复，避免误标过期。
- **API Key 支持**：支持 `vbk_...` 密钥直连 `/router/v1/*`，用量检查仅轻量验证模型列表接口，不误调 OAuth 专属 `/api/me/*`。
- **模型目录与思考档位**：按账号独立发现上下文长度、最大输出、视觉能力与思考档位，将 Verboo 的 `off` 映射为 Magpie 规范的 `none`。后续模型列表刷新会自动同步到既有 `loader` 且多账号完全隔离。显式请求的档位按 DeepSeek 思考格式（`thinking` + `reasoning_effort`）发送并自动对齐到最近支持档位；未指定档位时保留模型默认行为，非思考模型不注入思考参数。
- **订阅与 Stamina 用量（`auth.usage`）**：读取 `/api/me`、`/api/me/subscriptions`、`/api/me/stamina` 与 `/api/me/trial-status`，展示订阅计划、到期时间、自动续费状态、Stamina 节流余量（标记 `aside: true`，仅展示不阻塞网关路由）与试用额度；旧接口返回 `404` 时自动降级。
- **安全边界**：凭据仅发送到 `https://code.verboo.ai/router/v1` 与 `https://code.verboo.ai` 认证/账号接口，并禁用重定向（`redirect: "error"`）。对话响应原样流式交回 Magpie。

> 在 Magpie 里退出账号只会删除本地保存的凭据；由于 Magpie 不会把退出事件回调给插件，服务器端的刷新令牌会等待其自然失效。

## 本地测试

需要 Node.js >= 20.3 或 Bun（零外部依赖）：

```sh
npm test
# 或
bun test
```

## 参考链接

- [Verboo API 快速入门](https://code.verboo.ai/en/docs/getting-started)
- [Verboo 认证与错误](https://code.verboo.ai/en/docs/api/authentication)
- [Magpie 插件文档](https://usemagpie.ai/docs/zh/plugins)
