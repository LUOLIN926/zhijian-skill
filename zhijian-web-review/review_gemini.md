# 知见 (Zhijian-Web) 代码审查与安全评估报告

本报告对 `zhijian-web` 项目进行了全面、仔细的代码审查，涵盖后端（Node.js/TypeScript）与前端（Vite/React/TypeScript）部分的架构设计、逻辑正确性、安全防范及代码规范。

---

## 核心发现与总结

`zhijian-web` 整体设计结构清晰，前后端职责分工明确。前端通过沙箱 `<iframe>` 对用户创作的互动网页代码进行隔离，后端通过合理的事务管理和 Zod 参数校验，保证了基础业务的稳定运行。

但在深度代码审计中，发现以下 **2 项高风险安全漏洞** 与多项代码维护性、局部逻辑缺陷，需尽快修复：
1. **【高风险】已发布作品的编辑接口可绕过管理员审核机制**（直接上线恶意/敏感内容）
2. **【高风险】CORS 跨域配置反射请求源并携带凭证**（可导致跨站请求与敏感数据泄漏）
3. **【中风险】封禁/降级用户凭证延迟生效**（公共接口未走数据库状态校验，导致已封禁/降级 JWT 仍可访问部分限制性资源）
4. **【低风险/代码冗余】`createWorkVersion` 逻辑双重实现与遗留的无用服务文件**。

---

## 安全漏洞深度解析 (Security Vulnerabilities)

### 1. 【高风险】作品直接编辑绕过审核漏洞 (Admin Review Queue Bypass)
* **涉及文件**：
  * 后端控制器：[backend/src/handlers/works/index.ts](file:///Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/handlers/works/index.ts#L883-L1019)
  * 后端公共字段过滤：[backend/src/services/publicWorks.ts](file:///Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/services/publicWorks.ts#L20-L22)
* **漏洞描述**：
  在知见平台的设计中，为了防止用户发布不良或违规内容，作品的发布与变更需要通过管理员的审核（`pending_review` $\rightarrow$ `published`）。
  但在 `updateWork`（`PUT /api/v1/works/:id`）的实现中，后端仅在请求体包含 `status` 字段且值为 `'published'` 时，才会把作品状态置为待审核 `'pending_review'`：
  ```typescript
  let effectiveStatus: WorkStatus = existingWork.status
  if (existingWork.status === 'draft' || existingWork.status === 'published') {
    if (status) {
      const nextStatus = status === 'published' ? 'pending_review' : status
      updates.push('status = ?')
      values.push(nextStatus)
      effectiveStatus = nextStatus
      ...
    }
  }
  ```
  如果攻击者直接通过 API 发送 `PUT` 请求，更新 `html_content`、`title` 或 `brief_description`，但在请求体中**不传入 `status` 字段**（即 `status` 为 `undefined`），那么：
  * `updates` 列表不会添加 `status = ?` 这一项。
  * `effectiveStatus` 依然保持原来的 `'published'` 不变。
  * 数据库中的 `works` 表被直接更新为攻击者传入的 `html_content`（包含潜在的恶意脚本、钓鱼广告等）。
  * 由于状态依然是 `'published'`，前端和公共访问 API（例如 `getWorkHtml`）在查询时，其 `CASE WHEN` 逻辑：
    ```sql
    CASE WHEN w.status = 'pending_review' THEN pv.html_content ELSE w.html_content END
    ```
    会直接返回 `works` 表里的最新内容。**攻击者无需经过任何管理员审核，即可直接更新已发布作品的线上代码。**

* **防范与修复建议**：
  只要已发布或已上线的作品（`status === 'published'`）被修改了核心展示字段（如 `title`、`brief_description`、`html_content`），就应强制将作品状态回滚至 `'pending_review'`（待审核状态），或者采用“线上版与草稿版分离”的方案，将修改暂存，待审核通过后再合并覆盖到线上。

---

### 2. 【高风险】不安全的 CORS 跨域反射与凭证配置 (Insecure CORS Configuration)
* **涉及文件**：
  * 后端中间件：[backend/src/middleware/cors.ts](file:///Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/middleware/cors.ts#L7-L13)
* **漏洞描述**：
  在跨域资源共享（CORS）配置中，代码存在以下逻辑：
  ```typescript
  if (allowedOrigins.includes('*')) {
    res.setHeader('Access-Control-Allow-Origin', origin || '*')
  } else if (allowedOrigins.includes(origin)) {
    res.setHeader('Access-Control-Allow-Origin', origin)
  }

  res.setHeader('Access-Control-Allow-Credentials', 'true')
  ```
  当环境变量未配置 `ALLOWED_ORIGINS` 时，`allowedOrigins` 默认为 `['*']`。此时，系统会**动态读取客户端请求头中的 `Origin`，并直接将其原样反射写入到响应头 `Access-Control-Allow-Origin` 中**，同时设置 `Access-Control-Allow-Credentials: true`。
  
  这是一种典型的 CORS 配置不当漏洞：
  * 允许凭证（Credentials: true）时，浏览器禁止将 `Access-Control-Allow-Origin` 设置为 `*`，而本段代码通过动态反射请求源（Reflected Origin）绕过了此限制。
  * 任何恶意第三方网站（如 `http://malicious-site.com`）均可通过 AJAX/Fetch 请求知见 API 并读取响应，导致敏感数据泄露或被跨站劫持。例如，恶意的第三方站可以通过浏览器自动携带的 `refreshToken` Cookie，请求 `/api/v1/auth/refresh` 获取用户的最新 `accessToken` 并窃取。

* **防范与修复建议**：
  如果需要支持带凭证（`Credentials: true`）的跨域请求，**绝对不能**动态反射任意未经过白名单检验的请求源。应在生产环境配置明确的域名白名单，若请求源不在白名单中，则拒绝写入 `Access-Control-Allow-Origin`。

---

## 业务逻辑缺陷与改进点 (Business Logic Issues)

### 3. 【中风险】已封禁/已降级用户凭证延迟生效问题 (Stateless JWT Session Lag)
* **涉及文件**：
  * 后端路由层：[backend/src/router.ts](file:///Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/router.ts#L237-L283)
  * 后端公共接口助手：[backend/src/handlers/works/index.ts](file:///Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/handlers/works/index.ts#L93-L101) 和 [backend/src/handlers/comments/index.ts](file:///Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/handlers/comments/index.ts#L17-L25)
* **漏洞描述**：
  后端虽然在 `router.ts` 中针对**需要认证（`route.auth === true`）的接口**做了基于数据库的实时状态校验（带 1 分钟 LRU 缓存），拦截了已被封禁（`banned`）或禁用（`disabled`）的用户：
  ```typescript
  let currentUser = userCache.get(user.id)
  if (!currentUser) {
    const [userRows] = await db.query('SELECT id, role, status FROM users WHERE id = ? LIMIT 1', [user.id])
    ...
  }
  if (currentUser.status === 'banned') { ... return }
  ```
  但是，在部分**无需认证即可访问的公共接口**（如 `GET /api/v1/works/:id/html`、`GET /api/v1/works/:id/comments`）中，为了识别当前访问者是否为作者或管理员，使用的是手写的 `getViewerFromRequest` 方法：
  ```typescript
  function getViewerFromRequest(req: IncomingMessage): Viewer {
    const authHeader = req.headers.authorization
    ...
    const user = getUserFromToken(token)
    if (!user) return null
    return { id: user.id, role: user.role }
  }
  ```
  该方法**完全避开了数据库校验与缓存校验**，直接对 JWT 进行了解析。如果一个管理员或 Beta 创作者被取消权限或封禁，但其客户端的 JWT 依然处于有效期内（例如刚发出的 Token，或者有效期较长），他依然可以利用此 Token 的剩余有效期：
  * 绕过普通用户的评论列表可见性过滤，继续读取未通过审核的评论。
  * 绕过可见性检查，继续查看已被隐藏/封禁的私有作品。
* **防范与修复建议**：
  应统一提取一个全局的 `getAuthedViewer` 中间件/工具函数，对于携带 Token 的请求，一律使用 LRU 缓存或轻量级的 DB 状态查询，防止封禁/降级用户的 JWT 产生权限泄露“空窗期”。

---

## 代码质量与架构亮点 (Code Quality & Architecture Wins)

### 1. 完善的沙箱预览隔离（Security Win）
* **涉及文件**：
  * 编辑器预览：[frontend/src/components/editor/LivePreview.tsx](file:///Users/lin/Desktop/project/zhijian/zhijian-web/frontend/src/components/editor/LivePreview.tsx#L70)
  * 作品详情：[frontend/src/pages/WorkDetail.tsx](file:///Users/lin/Desktop/project/zhijian/zhijian-web/frontend/src/pages/WorkDetail.tsx#L160)
* **审查结果**：
  知见允许用户上传并运行任意的 HTML/CSS/JS 代码（包括 React UMD 片段）。为了防止 XSS 攻击劫持主站的登录状态（`localStorage`、Session 等），项目在渲染端做到了出色的防范：
  * 前端预览 `<iframe>` 统一配置了极其严密的沙箱属性 `sandbox="allow-scripts allow-modals"`。
  * **未配置 `allow-same-origin`**：使得 iframe 内部的页面与知见主站处于不同的源（Unique Origin），即使执行恶意脚本，也完全无法读取主站的 Token 或 Cookie。
  * 后端 HTML 渲染接口 `getWorkHtml` 配合下发了 `Content-Security-Policy` 的 `sandbox allow-scripts allow-modals` 响应头，实现了前后端双重保险。

### 2. 纯 JS/TS 内存 ZIP 压缩实现（Best Practice）
* **涉及文件**：
  * 压缩工具类：[backend/src/utils/zip.ts](file:///Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/utils/zip.ts)
* **审查结果**：
  在导出作品包的实现中，项目没有为了方便直接通过 `child_process` 运行系统的 `zip` 命令，也没有依赖复杂的第三方 C++ 二进制包，而是通过手写 CRC32 校验码与 Zip 头信息定义（`createStoredZip`），在 Node.js Buffer 内存中纯手工组装了一个标准的 ZIP 存档。
  这极大提高了平台的跨平台部署便捷性，消除了由于命令拼接带来的**系统命令注入攻击风险**，代码书写十分扎实。

---

## 代码冗余与清理建议 (Code Cleanup Recommendations)

### 1. `createWorkVersion` 逻辑重复实现
* **涉及文件**：
  * [backend/src/handlers/admin/index.ts#L101-L132](file:///Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/handlers/admin/index.ts#L101-L132)
  * [backend/src/handlers/works/index.ts#L534-L565](file:///Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/handlers/works/index.ts#L534-L565)
* **审查结果**：
  两处文件分别实现了一套完全相同的 `createWorkVersion` 方法（版本快照保存）。管理端版本的方法由于需要支持数据库事务，传入了 `executor: Queryable` 作为首个参数，而普通用户端的版本则直接使用了全局的 `db` 对象。
  这属于典型的代码复制粘贴（Code Duplication），建议将两个方法合并并提取为公共服务函数（例如定义在 `backend/src/services/workVersions.ts` 中），统一接收 `executor`（默认为全局 `db`）以简化结构。

### 2. 无效的遗留服务文件 `aiMetadata.ts`
* **涉及文件**：
  * [backend/src/services/aiMetadata.ts](file:///Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/services/aiMetadata.ts)
* **审查结果**：
  该文件定义了一套完整的基于 OpenAI API 的元数据生成方法，但是后端控制器的 AI 接口早已将数据逻辑迁移到了 [backend/src/services/aiGeneration.ts](file:///Users/lin/Desktop/project/zhijian/zhijian-web/backend/src/services/aiGeneration.ts) 中。`aiMetadata.ts` 文件目前在项目内无任何地方引用，属于历史遗留代码，应当直接予以删除，避免混淆。
