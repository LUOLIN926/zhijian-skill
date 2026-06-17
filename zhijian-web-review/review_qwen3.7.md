Let我先了解一下 zhijian-web 的最近变更情况。


没有未提交的更改。我将对 zhijian-web 项目的最近提交进行全面代码审查。让我先获取最近的变更范围。


现在我将并行启动三个代码审查子代理，分别审查完整性、正确性和影响面。


三个审查子代理已完成。以下是合并后的审查报告：

---

## 严重问题 (MUST FIX)

### 1. `lru-cache` 版本与 API 不匹配，后端无法启动

[router.ts#L3](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/router.ts#L3), [router.ts#L18-L21](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/router.ts#L18-L21)

**问题**: 代码使用 `import { LRUCache } from 'lru-cache'`（v7+ 命名导出 API + `ttl` 选项），但 `package.json` 中安装的是 `^5.1.1`。v5 没有 `LRUCache` 命名导出，也没有 `ttl` 选项，后端启动时会直接崩溃：`TypeError: LRUCache is not a constructor`。

**修复**: 升级 lru-cache 到最新版：
```bash
cd backend && npm install lru-cache@latest
```

---

## 警告 (SHOULD FIX)

### 1. Token 刷新失败时，排队请求的 Promise 永远不会 reject

[client.ts#L68-L81](/Users/lin/Desktop/project/zhijian/zhijian-web/frontend/src/api/client.ts#L68-L81)

**问题**: 刷新失败时（catch 分支），代码清空 `refreshSubscribers` 数组但未 reject 等待中的 Promise。这些 Promise 只存了 `resolve`，没有 `reject`，将永远挂起直到 15 秒 axios 超时。

**修复**: subscriber 同时存储 resolve/reject，刷新失败时 reject 所有等待请求：
```typescript
let refreshSubscribers: { resolve: (t: string) => void; reject: (e: Error) => void }[] = []
// catch 分支中：
refreshSubscribers.forEach(sub => sub.reject(refreshError as Error))
```

### 2. CORS 默认配置允许任意 Origin 携带凭证

[cors.ts#L4-L13](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/middleware/cors.ts#L4-L13)

**问题**: `ALLOWED_ORIGINS` 未设置时默认为 `['*']`，代码将请求 Origin 原样反射回响应头，同时无条件设置 `Allow-Credentials: true`。等效于允许任意来源携带认证 Cookie 跨域请求，存在 CSRF 风险。

**修复**: 通配符模式下不应设置 credentials，或在 `deploy/.env.production.example` 中强制配置明确的 origin 列表。

### 3. 用户状态缓存未在管理操作时失效（封禁延迟最长 60 秒）

[router.ts#L18-L21](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/router.ts#L18-L21), [router.ts#L251-L261](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/router.ts#L251-L261)

**问题**: `userCache`（TTL 60s）缓存了用户 `status` 和 `role`，但管理员封禁/禁用用户时未清除对应缓存。被封禁用户在最长 60 秒内仍可使用有效 JWT 访问所有 API。

**修复**: 导出 `invalidateUserCache(userId)` 函数，在 admin handler 的封禁/禁用/角色变更操作中调用。

### 4. Refresh Token Cookie 缺少 `Secure` 标志

[auth/index.ts#L21-L24](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/handlers/auth/index.ts#L21-L24)

**问题**: Cookie 设置了 `HttpOnly` + `SameSite=Lax`，但缺少 `Secure` 标志，生产 HTTPS 环境中 cookie 仍可能通过 HTTP 明文连接被发送。

**修复**:
```typescript
const secure = process.env.NODE_ENV === 'production' ? '; Secure' : ''
res.setHeader('Set-Cookie', `refreshToken=${token}; HttpOnly; Path=/api/v1/auth; Max-Age=2592000; SameSite=Lax${secure}`)
```

### 5. 认证 API 破坏性变更：`refreshToken` 从响应体移除

[auth/index.ts#L73-L86](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/handlers/auth/index.ts#L73-L86)

**问题**: `register`/`login` 响应不再包含 `refreshToken` 字段，改为 HttpOnly Cookie。如有移动端或第三方客户端依赖响应体中的字段，将无法获取刷新令牌。

---

## 建议 (CONSIDER)

### 1. `ALLOWED_ORIGINS` 未在部署配置示例中声明

[deploy/.env.production.example](/Users/lin/Desktop/project/zhijian/zhijian-web/deploy/.env.production.example)

**建议**: 在部署配置示例中添加 `ALLOWED_ORIGINS` 变量说明，确保生产部署时不会遗漏 CORS 配置。

---

## 变更总结

- **路由系统重构**: 从手写正则匹配迁移至 `find-my-way` 路由库 + LRU 用户缓存（**但 lru-cache 版本不匹配导致无法启动**）
- **认证系统增强**: 实现完整的 refresh token 机制（HttpOnly Cookie）、登出接口，前端增加 401 自动刷新拦截器和请求队列
- **CORS 安全改进**: 引入基于环境变量的 origin 白名单机制，作品 HTML 预览新增 CSP sandbox 指令
- **插件系统外部化**: 移除 examclock-v2 本地代码（~7400 行），改为外部 URL + 新标签页打开
- **编辑器调优**: 防抖延迟从 300ms 调整为 800ms，移除插件页面构建者筛选功能