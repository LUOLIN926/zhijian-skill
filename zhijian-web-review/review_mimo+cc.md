# zhijian-web 代码审查报告

**审查日期:** 2026-06-14
**审查工具:** MiMo + Claude Code
**项目:** 知见 (zhijian) —— "让知识看得见"，知识可视化创作与分享平台

---

## 项目架构概述

| 维度 | 详情 |
|------|------|
| **Monorepo** | npm workspaces (`frontend`, `backend`) |
| **前端** | React 18 + TypeScript + Vite 6, react-router-dom 6, TanStack Query 5, Zustand 5, CodeMirror 6, react-markdown, dnd-kit |
| **后端** | Node.js 20 裸 `http.createServer` (无框架), find-my-way 路由, MySQL2, JWT 认证, bcryptjs, Zod 校验, LRU 缓存 |
| **数据库** | MySQL |
| **开发端口** | 前端: 3000, 后端: 9000 (Vite 代理) |
| **构建** | 前端: `tsc -b && vite build`; 后端: esbuild 打包为单 ESM 文件 |
| **部署** | PM2 (ecosystem.config.js), 阿里云 OSS |
| **源文件数** | 前端: ~100 文件 (tsx/css/ts), 后端: ~27 文件 (ts) |

---

## 🔴 严重问题 (CRITICAL)

### 1. CORS 配置允许任意来源携带凭证

**文件:** `backend/src/middleware/cors.ts`

当 `ALLOWED_ORIGINS` 未设置时，默认 `['*']`，代码会将请求的 `Origin` 头直接反射到 `Access-Control-Allow-Origin`，同时设置了 `Access-Control-Allow-Credentials: true`。这是经典的 CORS 绕过漏洞——任何恶意网站都可以向 API 发起带凭证的跨域请求并读取响应。

```ts
if (allowedOrigins.includes('*')) {
  res.setHeader('Access-Control-Allow-Origin', origin || '*')  // 反射任意 Origin
}
// 同时设置了 Access-Control-Allow-Credentials: true
```

**修复建议:** 生产环境必须设置明确的白名单，且永远不要将 `*` 与 `credentials: true` 组合使用。

---

### 2. Access Token 与 Refresh Token 共用同一密钥

**文件:** `backend/src/services/jwt.ts`

`signAccessToken` 和 `signRefreshToken` 使用相同的 secret 和 payload 结构，唯一区别是过期时间。泄露的 refresh token 可直接当 access token 使用（包含 `id`, `username`, `role`，同一密钥签名）。且 refresh token 有效期 30 天且不可撤销。

**修复建议:** 使用独立的 refresh token 密钥，考虑将 refresh token 存储在数据库中以支持撤销，缩短有效期。

---

### 3. Token 刷新竞态条件与 Promise 永久挂起

**文件:** `frontend/src/api/client.ts`

多个并发 401 请求时，只有第一个触发刷新，其余通过 `refreshSubscribers` 排队等待。但存在以下问题：

- 刷新失败时，排队的 Promise 永远不会 reject，导致请求永久挂起
- `isRefreshing` 标志没有超时机制，刷新请求无响应时所有后续 401 处理永久排队
- 非空断言 `useAuthStore.getState().user!` 在 user 为 null 时会传递 null 给 `login()`

**修复建议:** 为排队 Promise 添加 reject 路径和超时机制；为 isRefreshing 添加超时重置。

---

### 4. 前端预览无 HTML 净化 (XSS)

**文件:** `frontend/src/utils/previewHtml.ts`

`normalizePreviewHtml()` 将用户 HTML 直接注入 iframe，无 DOMPurify 等净化。虽然 sandbox 属性 (`allow-scripts allow-modals`) 阻止了部分能力，但 `allow-scripts` 仍允许任意 JS 执行：

- 通过 `fetch()` 或 `navigator.sendBeacon()` 外传数据
- 渲染钓鱼登录覆盖层
- 挖矿、DDoS 等计算密集型攻击
- 通过已建立的 `postMessage` 通道发送伪造消息

**文件:** `frontend/src/pages/WorkDetail.tsx` 中 `<iframe srcDoc={previewHtml}>` 来自服务端存储的 HTML，存在存储型 XSS 风险。

**修复建议:** 添加 DOMPurify 净化，或在 iframe 内添加 CSP meta 标签限制 `connect-src` 等。

---

### 5. 无请求体大小限制 (DoS)

**文件:** `backend/src/router.ts`

`readBody()` 函数无 body 大小限制，攻击者可发送超大请求耗尽服务器内存。且 `req.on('error')` 事件未处理，Promise 永不 reject。

```ts
async function readBody(req: IncomingMessage): Promise<any> {
  return new Promise((resolve) => {
    const chunks: Buffer[] = []
    req.on('data', (chunk) => chunks.push(chunk))  // 无大小限制
    req.on('end', () => { ... })
    // 缺少 error 处理
  })
}
```

**修复建议:** 在 `data` 回调中累加字节数，超过阈值（如 1MB）时 destroy 请求；添加 `error` 事件处理 reject Promise。

---

## 🟠 高危问题 (HIGH)

### 6. 前端无路由级权限守卫

**文件:** `frontend/src/App.tsx`

`/admin/*`、`/settings`、`/create`、`/works/manage` 等受保护路由无任何认证/授权检查。未登录用户可直接访问，非管理员可访问管理后台。无 404 路由兜底。

**修复建议:** 创建 `ProtectedRoute` 和 `AdminRoute` 组件包装受保护路由；添加 `<Route path="*" element={<NotFound />} />`。

---

### 7. 点赞/收藏操作存在竞态条件

**文件:** `backend/src/handlers/interactions/index.ts`, `backend/src/handlers/collections/index.ts`

SELECT → INSERT/DELETE → UPDATE count 的模式在并发下会导致：
- 两个并发请求都看到 "not found"，都执行 INSERT
- 导致重复键崩溃或 `like_count`/`bookmark_count` 双倍计数

**修复建议:** 使用 `INSERT ... ON DUPLICATE KEY UPDATE` 或事务 + `SELECT ... FOR UPDATE` 锁。

---

### 8. `banUser` 可封禁管理员

**文件:** `backend/src/handlers/admin/index.ts`

`banUser` 未检查目标是否为管理员（`disableUser` 有此检查），也未验证用户是否存在。管理员可封禁其他管理员甚至自己。

```ts
// banUser — 无管理员检查
export async function banUser(req, res, params) {
  if (!ensureAdmin(req, res)) return
  const id = Number(params.id)
  await db.query("UPDATE users SET status = 'banned' WHERE id = ?", [id])
  jsonResponse(res, { id })
}

// disableUser — 有管理员检查 ✓
if (targetUser.role === 'admin') return errorResponse(res, 400, '不可禁用管理员账号')
```

**修复建议:** 添加与 `disableUser` 相同的管理员检查和用户存在性验证。

---

### 9. `html_content` 无长度限制

**文件:** `backend/src/handlers/works/index.ts`

所有其他大文本字段（`usage_markdown`, `html_cover`, `brief_description`）都有 `.max()`，唯独 `html_content` 没有。可发送任意大字符串，耗尽服务器内存或超出数据库列限制。

**修复建议:** 添加 `.max(5_000_000)` 或匹配数据库列类型的最大值。

---

### 10. Editor.tsx 性能问题

**文件:** `frontend/src/pages/Editor.tsx`

- **`getCombinedHtml()` 在每次渲染时执行** — 包含 HTML 拼接和正则解析，在每次按键时同步运行。应移至 `useMemo` 或事件处理器。
- **800ms 防抖 timer 依赖不稳定的函数引用** — `getCombinedHtml` 是 store 方法，每次渲染引用可能变化，导致 effect 频繁重新触发。

**修复建议:** 用 `useMemo` 缓存 `getCombinedHtml` 结果；将函数引用通过 `useRef` 稳定化。

---

### 11. 全局无错误边界

**文件:** `frontend/src/main.tsx`

无 `<ErrorBoundary>` 包裹 `<App />`，未捕获的渲染错误会导致白屏且无法恢复。React Query 也无全局 `onError` 处理。

**修复建议:** 添加 ErrorBoundary 组件包裹 App，提供降级 UI 和恢复路径。

---

## 🟡 中等问题 (MEDIUM)

### 后端

| # | 文件 | 问题 | 修复建议 |
|---|------|------|----------|
| 12 | `router.ts` | 用户缓存 TTL 1分钟，封禁后最多60秒内仍可访问 | 缩短 TTL 至 10-15 秒，或在管理操作时主动失效缓存 |
| 13 | `router.ts` | `versionId` 错误映射为 `workId`/`chapterId` | 移除映射，handler 直接使用 `params.versionId` |
| 14 | `auth/index.ts` | Refresh cookie 缺少 `Secure` 标志 | 生产环境添加 `Secure` 标志 |
| 15 | `auth/index.ts` | 手写 cookie 解析器不健壮 | 使用 `cookie` npm 包 |
| 16 | `comments/index.ts` | `parent_id` 接受负数/浮点数 | 改为 `z.number().int().positive().optional()` |
| 17 | `works/index.ts` | `syncWorkTags` 未在事务中，崩溃导致标签丢失 | 用 `withTransaction` 包裹 |
| 18 | 多个 handler | 搜索参数无长度限制 | 添加 `.slice(0, 200)` 或 schema 级校验 |

### 前端

| # | 文件 | 问题 | 修复建议 |
|---|------|------|----------|
| 19 | `api/client.ts` | 硬编码中文字符串匹配账户状态 | 使用状态码而非消息文本 |
| 20 | `api/client.ts` | 响应拦截器就地修改 `res.data`，`data: null` 时崩溃 | 返回新对象而非修改原对象 |
| 21 | 多个 API 文件 | 所有返回类型使用 `as` 强制转换，零运行时类型安全 | 考虑 zod 校验关键响应 |
| 22 | `editorStore.ts` | 正则解析 HTML 在多 `<style>`/`<script>` 块时失败 | 使用 DOMParser 替代正则 |
| 23 | `useMediaQuery.ts` | `query` prop 变更时状态不更新；SSR 不安全 | effect 中同步更新状态；添加 `typeof window` 检查 |
| 24 | `useDocumentTitle.ts` | 卸载时未恢复标题 | cleanup 函数中恢复 `previousTitle` |
| 25 | `bookmarkFolders.ts` | 递归函数无环检测，脏数据导致无限递归 | 添加 `visited` Set 守卫 |
| 26 | `Home.tsx` | `handleMouseMove` 每像素移动触发全组件重渲染 | 使用 `useRef` + `requestAnimationFrame` |
| 27 | `previewHtml.ts` | `postMessage` 使用 `'*'` 目标源 | 使用具体 origin |
| 28 | `previewHtml.ts` | 开发版 React CDN 脚本用于生产环境 | 使用 production.min.js |
| 29 | `Profile.tsx` | 标签页缺少 ARIA `role="tab"`/`aria-selected` | 参照 Editor.tsx 的实现 |
| 30 | `PublishDialog.tsx` | 标签删除按钮无 `aria-label` | 添加 `aria-label={移除标签 ${tag}}` |
| 31 | `WorkDetail.tsx` | 评论删除无确认对话框 | 添加确认弹窗 |
| 32 | `Editor.tsx` | 工作加载失败无反馈 | 添加 `onError` 处理 |
| 33 | `Profile.tsx` | 加载失败显示 "用户不存在" 而非实际错误 | 区分网络错误和用户不存在 |

---

## 🟢 低危问题 (LOW)

| # | 文件 | 问题 |
|---|------|------|
| 34 | `db.ts` | 环境变量缺失时静默回退到 `root`/空密码，应抛出错误 |
| 35 | `index.ts` | 无优雅关闭 (SIGTERM/SIGINT)，连接池未排空 |
| 36 | `response.ts` | 错误码 `status * 100` 与 router 内联响应不一致 |
| 37 | `admin/index.ts` | 本地重复定义 `withTransaction`，应从 services/db 导入 |
| 38 | `admin/index.ts` | `status` 过滤器无白名单校验，拼写错误静默返回空结果 |
| 39 | `upload/index.ts` | 手写 multipart 解析器，建议用 `busboy` |
| 40 | `auth.ts` | `refreshTokenApi` 是死代码，误用会导致无限循环 |
| 41 | `admin.ts` | `maxUses` 用 falsy 检查，`0` 值被跳过 |
| 42 | `admin.ts` | `Record<string, any>` 应使用具体类型 |
| 43 | `App.tsx` | 除 Editor 外所有页面同步导入，增大初始包体积 |
| 44 | `App.tsx` | 无 404 路由兜底 |
| 45 | `comments.ts` | `getComments`/`createComment`/`deleteComment` 返回 `any`，`Comment` 类型未使用 |
| 46 | `bookmarks.ts` | `BookmarkFolderFilter` 混合 string literals 和 number 类型 |
| 47 | `users/index.ts` | `display_name`/`bio` 接受纯空白字符串 |
| 48 | `htmlCoverTemplates.ts` | switch 无 default/fallback，类型扩展时静默返回 undefined |
| 49 | `assetUrl.ts` | `getAbsoluteApiOrigin()` 每次调用重新解析 URL |
| 50 | `Home.tsx` | IntersectionObserver 查询整个 DOM，耦合类名 |
| 51 | `Profile.tsx` | `navigate` 在渲染期间调用，Strict Mode 下双重导航 |
| 52 | `CodeEditor.tsx` | 外部值变更时全量替换文档，破坏撤销历史 |
| 53 | `admin.ts` | `is_active` 类型为 `number` 而非 `boolean` |
| 54 | `plugins.ts` | `is_active` 类型为 `number` 而非 `boolean` |
| 55 | `bookmarkFolders.ts` | 递归函数中 O(n) 线性搜索，整体 O(n²) 复杂度 |

---

## ✅ 做得好的地方

- **SQL 参数化查询全覆盖** —— 未发现 SQL 注入风险
- **Zod 输入校验** —— 大部分 handler 使用 schema 验证
- **React Markdown 默认安全** —— 不渲染原始 HTML（无 `rehype-raw`）
- **Header 汉堡按钮** —— 正确使用 `aria-label` 和 `aria-expanded`
- **Editor 标签页** —— 正确实现 ARIA tab 模式
- **Mutation 错误处理** —— 大部分 mutation 有 `onError` toast 提示
- **CodeMirror 编辑器** —— 语法高亮和主题支持完善
- **sandbox 属性** —— iframe 已移除 `allow-same-origin`，降低部分 XSS 风险
- **JWT_SECRET** —— 环境变量缺失时正确抛出错误

---

## 优先修复建议

### P0 — 立即修复（安全漏洞）

1. **CORS 配置** (#1) — 设置明确的 Origin 白名单
2. **请求体大小限制** (#5) — 添加 body size limit
3. **JWT 密钥分离** (#2) — refresh token 使用独立密钥
4. **Token 刷新机制** (#3) — 添加 reject 路径和超时
5. **HTML 净化** (#4) — 添加 DOMPurify

### P1 — 尽快修复（功能缺陷）

6. 前端路由守卫 (#6)
7. 点赞竞态条件 (#7)
8. banUser 权限检查 (#8)
9. html_content 长度限制 (#9)

### P2 — 版本迭代（质量提升）

10. Editor 性能优化 (#10)
11. 错误边界 (#11)
12. 类型安全 (#21)
13. 无障碍 (#29, #30)
14. 中等问题批量修复 (#12-#33)

### P3 — 长期改进

15. 低危问题清理 (#34-#55)
16. 测试覆盖
17. ESLint/Prettier 统一代码风格
