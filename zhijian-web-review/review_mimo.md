
# zhijian-web 代码审查报告

## Critical Issues (MUST FIX)

### 1. LRU 缓存配置错误：`ttl` 选项无效，用户状态缓存永不过期
[backend/src/router.ts#L85-L88](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/router.ts)
**问题**：引入的 `lru-cache` 版本为 **v5.1.1**，该版本的过期配置选项名称是 `maxAge`，而不是 `ttl`（`ttl` 是 v7+ 才引入的）。当前代码使用了 `ttl: 1000 * 60`，该选项会被 v5 静默忽略，导致缓存条目**永远不会过期**。管理员封禁/禁用用户、调整用户角色后，变更最长需要等到 1000 个不同用户登录后才会因 LRU 淘汰生效（而非预期的 1 分钟）。
**修复**：将 `ttl` 改为 `maxAge`：
```typescript
const userCache = new LRUCache<number, { id: number; role: string; status: string }>({
  max: 1000,
  maxAge: 1000 * 60 // 1 minute
})
```

### 2. CORS 配置安全漏洞：任意来源反射 + 允许凭据 = 跨站认证请求
[backend/src/middleware/cors.ts#L7-L8](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/middleware/cors.ts)
**问题**：当环境变量 `ALLOWED_ORIGINS` 未设置时（当前开发和 `.env` 中均未配置），默认 `allowedOrigins = ['*']`。代码将浏览器请求的 `Origin` 头直接反射回 `Access-Control-Allow-Origin`，同时始终设置 `Access-Control-Allow-Credentials: true`。这形成了经典的 CORS 反射漏洞：**任何网站**都能对本 API 发起跨域认证请求并读取响应。
**修复**：当 `allowedOrigins` 为 `['*']` 时，不应反射 `origin`，也不应设置 `Credentials: true`：
```typescript
export function cors(req: IncomingMessage, res: ServerResponse) {
  const allowedOrigins = process.env.ALLOWED_ORIGINS ? process.env.ALLOWED_ORIGINS.split(',') : ['*']
  const origin = req.headers.origin || ''

  if (allowedOrigins.includes('*')) {
    res.setHeader('Access-Control-Allow-Origin', '*')
  } else if (allowedOrigins.includes(origin)) {
    res.setHeader('Access-Control-Allow-Origin', origin)
    res.setHeader('Access-Control-Allow-Credentials', 'true')
  }
  // ... 其他头部设置
}
```

### 3. 刷新令牌端点不验证用户状态，被封禁/禁用用户可持续获取新令牌
[backend/src/handlers/auth/index.ts#L143-L169](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/handlers/auth/index.ts)
**问题**：`refreshToken` 端点（路由标记为 `auth: false`）仅通过 JWT 签名验证 refresh token 的有效性，但**不查询数据库检查用户当前状态**。即使管理员已封禁或禁用某用户，该用户仍可通过 refresh token 持续获取新的 access token，绕过封禁。
**修复**：在验证 JWT 后添加数据库查询：
```typescript
export async function refreshToken(req: IncomingMessage, res: ServerResponse) {
  // ... 现有 cookie 解析代码 ...
  const user = getUserFromToken(token)
  if (!user) return errorResponse(res, 401, 'Invalid refresh token')

  // 新增：查询数据库验证用户当前状态
  const [rows] = await db.query(
    'SELECT id, role, status FROM users WHERE id = ? LIMIT 1',
    [user.id]
  )
  const currentUser = (rows as any[])[0]
  if (!currentUser) return errorResponse(res, 401, '用户不存在')
  if (currentUser.status === 'banned' || currentUser.status === 'disabled') {
    return errorResponse(res, 403, '该账号已被限制')
  }
  // ... 继续生成新 token ...
}
```

### 4. `banUser` 缺少对目标用户的验证，管理员可封禁其他管理员
[backend/src/handlers/admin/index.ts#L320-L325](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/handlers/admin/index.ts)
**问题**：`banUser` 函数直接执行 `UPDATE users SET status = 'banned' WHERE id = ?`，没有检查：目标用户是否存在、是否已被封禁、**是否是管理员**（管理员可以封禁其他管理员！）、是否是操作自己。对比同文件中的 `disableUser`，后者完整检查了所有这些条件。
**修复**：添加完整的验证逻辑：
```typescript
export async function banUser(req: IncomingMessage, res: ServerResponse, params: Record<string, string>) {
  if (!ensureAdmin(req, res)) return
  const id = Number(params.id)
  const adminUser = (req as any).user

  if (id === adminUser.id) return errorResponse(res, 400, '不可封禁自己')

  const [rows] = await db.query('SELECT id, role, status FROM users WHERE id = ? LIMIT 1', [id])
  const targetUser = (rows as any[])[0]
  if (!targetUser) return errorResponse(res, 404, '用户不存在')
  if (targetUser.role === 'admin') return errorResponse(res, 400, '不可封禁管理员')
  if (targetUser.status === 'banned') return errorResponse(res, 400, '该用户已被封禁')

  await db.query("UPDATE users SET status = 'banned' WHERE id = ?", [id])
  jsonResponse(res, { id })
}
```

## Warnings (SHOULD FIX)

### 1. 刷新令牌 Cookie 缺少 `Secure` 标志
[backend/src/handlers/auth/index.ts#L23](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/handlers/auth/index.ts)
**问题**：`setRefreshTokenCookie` 设置的 Cookie 没有 `Secure` 标志。Refresh Token 有效期为 30 天。缺少 `Secure` 标志意味着该 Cookie 会在 HTTP 明文连接中传输，存在被中间人截获的风险。
**修复**：根据环境动态设置 `Secure`：
```typescript
function setRefreshTokenCookie(res: ServerResponse, token: string) {
  const secure = process.env.NODE_ENV === 'production' ? '; Secure' : ''
  res.setHeader('Set-Cookie', `refreshToken=${token}; HttpOnly; Path=/api/v1/auth; Max-Age=2592000; SameSite=Lax${secure}`)
}
```

### 2. 用户状态缓存在封禁/禁用后最长 1 分钟内不生效
[backend/src/router.ts#L85-L88](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/router.ts)
**问题**：即使修复了 `ttl` 配置问题，`userCache` 的 TTL 仍为 1 分钟。当管理员封禁或禁用某用户后，该用户在缓存过期前（最长 60 秒）仍可正常访问所有认证接口。
**修复**：在修改用户状态的管理操作后，主动清除该用户的缓存条目：
```typescript
// 在 router.ts 中导出缓存清除方法
export function invalidateUserCache(userId: number) {
  userCache.delete(userId)
}

// 在 admin handler 中调用
import { invalidateUserCache } from '../../router.js'
// ... 在状态变更后 ...
invalidateUserCache(id)
```

### 3. SubmitWork 页面 iframe 使用 `sandbox=""` 完全移除沙箱限制
[frontend/src/pages/SubmitWork.tsx#L688-L689](/Users/lin/Desktop/project/zhijian/zhijian-web/frontend/src/pages/SubmitWork.tsx)
**问题**：两处 `ScaledPreviewFrame` 使用了 `sandbox=""`（空字符串），这意味着 iframe **没有任何沙箱限制**。虽然 HTML 封面内容来自服务端生成，但如果未来封面内容来源变更或服务端生成逻辑被绕过，这将成为严重的安全漏洞。
**修复**：添加适当的沙箱限制：
```tsx
<ScaledPreviewFrame
  srcDoc={htmlCover}
  sandbox="allow-scripts"  // 仅允许脚本执行
  title="HTML 封面预览"
  ...
/>
```

### 4. `readBody` 函数的 Promise 永不 reject，请求体解析错误被静默吞掉
[backend/src/router.ts#L200-L213](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/router.ts)
**问题**：`readBody` 函数创建的 Promise 只会 resolve（出错时 resolve 为 `null`），永远不会 reject。如果请求流发生错误（如网络中断），错误会被静默忽略，handler 会收到 `null` body 并继续执行。对于 POST/PUT 请求，这可能导致意外的空数据写入。
**修复**：添加错误事件监听：
```typescript
async function readBody(req: IncomingMessage): Promise<any> {
  return new Promise((resolve, reject) => {
    const chunks: Buffer[] = []
    req.on('data', (chunk) => chunks.push(chunk))
    req.on('error', (err) => reject(err))  // 新增：传播流错误
    req.on('end', () => {
      if (chunks.length === 0) return resolve(null)
      try {
        resolve(JSON.parse(Buffer.concat(chunks).toString()))
      } catch {
        resolve(null)
      }
    })
  })
}
```

### 5. JWT Access Token 和 Refresh Token 使用相同密钥
[backend/src/services/jwt.ts#L11-L27](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/services/jwt.ts)
**问题**：Access Token（15 分钟有效期）和 Refresh Token（30 天有效期）使用同一个 `JWT_SECRET` 签发。如果 Access Token 泄露，攻击者理论上可以构造自定义 payload 签发新的 Refresh Token，从而获得长期访问权限。
**修复**：使用不同的密钥：
```typescript
const JWT_ACCESS_SECRET = requireEnv('JWT_SECRET')
const JWT_REFRESH_SECRET = process.env.JWT_REFRESH_SECRET || requireEnv('JWT_SECRET') + ':refresh'

export function signRefreshToken(payload: TokenPayload): string {
  return jwt.sign(payload, JWT_REFRESH_SECRET, { expiresIn: REFRESH_TOKEN_EXPIRY })
}
```

### 6. 令牌刷新时 `user!` 非空断言可能导致认证状态不一致
[frontend/src/api/client.ts#L60](/Users/lin/Desktop/project/zhijian/zhijian-web/frontend/src/api/client.ts)
**问题**：自动刷新令牌的拦截器中使用了 `useAuthStore.getState().user!`（TypeScript 非空断言）。如果在刷新请求进行期间，用户从另一个标签页/窗口执行了登出操作，则 `user` 为 `null`。此时 `login(null, newToken)` 会将 store 设置为 `{ user: null, accessToken: newToken, isAuthenticated: true }` —— 一个**不一致状态**。
**修复**：添加空值检查：
```typescript
const currentUser = useAuthStore.getState().user
if (!currentUser) {
  throw new Error('User logged out during token refresh')
}
useAuthStore.getState().login(currentUser, newToken)
```

### 7. 点赞/收藏操作未使用事务，存在计数不一致风险
[backend/src/handlers/interactions/index.ts#L22-L57](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/handlers/interactions/index.ts)
**问题**：`toggleLike`、`toggleBookmark` 等操作中，对互动记录的增删和计数字段的更新是分别执行的两条独立 SQL 语句，**没有包裹在数据库事务中**。如果在两条语句之间发生服务崩溃或数据库异常，会导致 like_count / bookmark 状态与实际记录不一致。
**修复方向**：将所有 toggle 操作中的删除/插入和计数更新包裹在 `withTransaction` 中。

### 8. 管理员下架作品和驳回作品未清除 userCache
[backend/src/handlers/admin/index.ts#L247-L289](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/handlers/admin/index.ts)
**问题**：当管理员封禁用户、禁用用户、修改角色等操作修改了用户的 `role` 或 `status` 字段后，`router.ts` 中的 LRU 用户缓存仍然持有旧数据。在缓存过期之前，被封禁/禁用的用户仍可以正常访问需要认证的接口。
**修复方向**：在修改用户状态的管理操作完成后，主动清除该用户的缓存条目。

## Suggestions (CONSIDER)

### 1. 前端管理后台路由缺少客户端权限守卫
[frontend/src/App.tsx#L68-L74](/Users/lin/Desktop/project/zhijian/zhijian-web/frontend/src/App.tsx)
**问题**：`/admin/*` 路由没有客户端路由守卫。非管理员用户可以直接在浏览器地址栏输入 `/admin` 访问管理后台页面。虽然后续 API 调用会因权限不足而失败，但页面框架仍会渲染，用户体验不佳且可能泄露管理后台的 UI 结构。
**修复**：创建一个 `AdminRoute` 包装组件，在路由层面拦截非管理员用户。

### 2. `syncWorkTags` 存在 N+1 查询问题
[backend/src/handlers/works/index.ts#L172-L189](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/handlers/works/index.ts)
**问题**：对每个 tag 执行一次 SELECT + 一次 INSERT（或 SELECT + INSERT），如果有 N 个 tag 则需要 2N-3N 次数据库查询。当作品标签较多时会产生显著的性能开销。
**修复方向**：批量查询所有已存在的 tag，然后一次性插入新 tag，可以将 2N 次查询减少为 2-3 次。

### 3. `aiMetadata.ts` 整个文件是未使用的死代码
[backend/src/services/aiMetadata.ts](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/services/aiMetadata.ts)
**问题**：`aiMetadata.ts`（199 行）定义了 `generateWorkMetadata` 函数，但该文件**从未被任何其他模块导入**。实际使用的元数据生成功能在 `aiGeneration.ts` 中。这个文件增加了代码库的维护负担和混淆度。
**修复**：删除 `aiMetadata.ts` 文件。

### 4. `getWorkVersionDiff` 导出但未注册为路由
[backend/src/handlers/admin/index.ts#L839-L853](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/handlers/admin/index.ts)
**问题**：`getWorkVersionDiff` 函数已实现并导出，但在 `router.ts` 的路由表中没有对应的路由注册，也未在 admin handler 的导入中被使用。这是一个被遗忘的功能或未完成的集成。
**修复**：如果需要此功能，在 router 中添加路由注册；否则删除此死代码。

### 5. 前端 `selects.length === 0` 分支为不可达代码
[backend/src/handlers/users/index.ts#L322-L323](/Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/handlers/users/index.ts)
**问题**：在 `getMyBookmarks` 中，`itemType` 的合法值为 `'all'`、`'work'`、`'collection'`。当为 `'all'` 时两个 select 都添加，为 `'work'` 时添加第一个，为 `'collection'` 时添加第二个。因此 `selects.length === 0` 的分支永远不会执行。虽为防御性代码，但增加了理解成本。
**修复**：可以移除该分支或添加注释说明其防御性目的。

## Summary of Changes

- **认证系统**：实现了完整的 JWT 双令牌认证流程（注册/登录/登出/刷新令牌），使用 bcryptjs 哈希密码，HttpOnly cookie 存储 refresh token，支持内测码（beta code）机制
- **作品管理**：支持作品的 CRUD、草稿/发布/审核/锁定/隐藏状态流转、版本历史记录与恢复、ZIP 包导出下载
- **合集系统**：实现了多层章节结构的合集管理，支持作品在多个合集间灵活关联、章节树的 CRUD 和排序
- **社交互动**：实现了作品/合集的点赞、收藏、评论功能，支持收藏夹文件夹分层管理（最多 3 级），普通用户评论需要审核
- **管理后台**：完整的管理员功能，包括作品审核（通过/驳回/下架）、用户管理（封禁/禁用/恢复/删除/角色变更）、评论审核、插件管理、内测码管理
- **AI 功能**：集成了 AI 元数据生成、AI 封面生成和 HTML 模板封面生成，支持多种封面风格，有每日用量限制（beta 用户 5 倍额度）
- **前端架构**：React + TypeScript + Vite + Zustand + React Query + CodeMirror 编辑器，支持代码/可视化分屏和全屏编辑模式，完整的管理后台前端页面
## 代码审查完成

我已经完成了对 `zhijian-web` 代码库的全面审查，从完整性、正确性和影响三个角度进行了分析。以下是审查结果的总结：

### 关键发现

**严重问题（必须修复）：**
1. **LRU缓存配置错误** - `lru-cache` v5.1.1 使用了错误的 `ttl` 选项，导致用户状态缓存永不过期
2. **CORS配置安全漏洞** - 允许任意来源反射 + 凭据，构成跨站认证请求风险
3. **刷新令牌端点不验证用户状态** - 被封禁/禁用用户可持续获取新令牌
4. **`banUser`缺少验证** - 管理员可封禁其他管理员，缺少必要的边界检查

**警告（应该修复）：**
- 刷新令牌Cookie缺少`Secure`标志
- 用户状态缓存存在1分钟窗口期
- iframe沙箱限制过宽
- 请求体解析错误处理不完善
- JWT令牌密钥混用风险
- 前端状态管理边界条件问题
- 数据库事务缺失风险

**建议（可考虑）：**
- 前端管理后台缺少权限守卫
- 存在N+1查询性能问题
- 部分死代码未清理

### 整体评估

代码库整体架构清晰，功能完整，但存在几个关键的安全问题需要立即修复，特别是CORS配置和缓存机制的问题。建议优先处理Critical级别的问题，然后逐步解决Warnings中的项目。

完整的审查报告已按照标准格式生成，包含了具体的问题位置、详细描述和修复方案。