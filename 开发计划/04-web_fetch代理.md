# 04-web_fetch 支持代理 · 开发计划

> 关联需求：`需求文档/04-web_fetch支持代理需求.md`
> 日期：2026-04-29

## 改动范围

| 文件 | 改动 | 行数 |
|------|------|------|
| `src/agents/tools/web-fetch.ts` | `fetchWithWebToolsNetworkGuard` 调用加 `useEnvProxy: true` | +1 |
| `src/agents/tools/web-fetch.test.ts` | 新增：有/无 proxy env 时的 dispatcher 行为测试 | ~20 |

## 实施步骤

1. 修改 `web-fetch.ts`：在 `fetchWithWebToolsNetworkGuard` 参数中加 `useEnvProxy: true`
2. 编译：`pnpm build`
3. 验证：调用 `web_fetch` 访问被墙站点（如 Wikipedia），确认通过代理成功返回
4. 回归：确认 GitHub 等可直连站点仍正常
5. 全局安装：`npm install -g .`
6. 重启 Gateway：`sudo systemctl restart openclaw-gateway`

## 风险

- 极低：仅加一个参数，不改变现有逻辑路径
- 代理不可用时退化为直连（`EnvHttpProxyAgent` 在无 env var 时等价于直连）
