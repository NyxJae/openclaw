# 04-web_fetch 支持代理需求

> 2026-04-29

## 背景

腾讯云服务器上部分境外站点不可直连，需走本地 HTTP 代理 `http://127.0.0.1:7890`。

`web_search`（DuckDuckGo provider）已通过 `withTrustedWebToolsEndpoint` → `useEnvProxy: true` 支持读取 `HTTP_PROXY`/`HTTPS_PROXY` 环境变量走代理。

`web_fetch` 调 `fetchWithWebToolsNetworkGuard` 时**未传** `useEnvProxy`，走 strict 模式，不走代理。导致被墙站点（如 Wikipedia）无法访问。

## 需求

让 `web_fetch` 支持代理，行为与 `web_search` 一致。

## 方案

`src/agents/tools/web-fetch.ts` 中 `fetchWithWebToolsNetworkGuard` 调用加 `useEnvProxy: true`：

```ts
const result = await fetchWithWebToolsNetworkGuard({
  url: params.url,
  maxRedirects: params.maxRedirects,
  timeoutSeconds: params.timeoutSeconds,
  useEnvProxy: true,  // ← 新增
  lookupFn: params.lookupFn,
  policy: ...,
  init: { ... },
});
```

## 代理来源

读取标准环境变量 `HTTP_PROXY` / `HTTPS_PROXY` / `NO_PROXY`，与 `web_search` 的 `useEnvProxy` 机制完全一致。当前已通过 systemd drop-in 注入到 Gateway 进程。

## 举例

### 正向例

- `web_fetch({ url: "https://en.wikipedia.org/wiki/..." })` → 通过 `http://127.0.0.1:7890` 代理访问 → 正常返回内容
- `web_fetch({ url: "https://github.com/..." })` → 通过代理访问 → 正常返回内容
- `web_fetch({ url: "http://127.0.0.1:8080/..." })` → `NO_PROXY` 含 `127.0.0.1` → 直连

### 反例

无（加 `useEnvProxy` 后 behavior 为 pure addition，不破坏现有功能）

## 涉及文件

| 文件 | 改动 |
|------|------|
| `src/agents/tools/web-fetch.ts` | `fetchWithWebToolsNetworkGuard` 调用加 `useEnvProxy: true` |
