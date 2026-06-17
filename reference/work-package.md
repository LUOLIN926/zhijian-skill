# 知见作品包参考文档

## 什么是作品包

作品包是从知见平台下载的 ZIP 压缩包，包含一个作品的全部文件：源码、元数据和封面。可用于备份、离线编辑、版本存档或迁移到其他平台。

---

## 作品包结构

```
作品名-{id}.zip
├── source/
│   └── index.html              ← 作品 HTML 源码（必含）
├── metadata/
│   ├── work.json               ← 结构化元数据（必含）
│   └── usage.md                ← 使用说明 Markdown（可为空）
└── cover/
    ├── cover.{ext}             ← 图片封面（jpg/png/webp/gif，可选）
    └── cover.html              ← HTML 封面（与图片封面二选一，可选）
```

---

## work.json 字段规范

```json
{
  "schema_version": 1,
  "id": 123,
  "title": "作品标题",
  "brief_description": "一句话简介",
  "usage_markdown": "## 使用说明\n...",
  "tags": ["交互演示", "标签1", "标签2"],
  "status": "published",
  "cover": {
    "type": "image",
    "thumbnail_url": "/api/v1/uploads/covers/abc.png",
    "has_html_cover": false
  },
  "created_at": "2026-01-01T00:00:00.000Z",
  "updated_at": "2026-05-01T00:00:00.000Z",
  "exported_at": "2026-05-11T12:00:00.000Z"
}
```

### 字段约束

| 字段 | 类型 | 必填 | 约束 |
|------|------|------|------|
| `schema_version` | number | 是 | 必须为 `1` |
| `id` | number | 是 | 平台作品 ID（仅作参考，上传时忽略） |
| `title` | string | 是 | 非空，去除首尾空格后 ≤ 200 字符 |
| `brief_description` | string | 否 | ≤ 2000 字符 |
| `usage_markdown` | string | 否 | Markdown 格式，≤ 200000 字符 |
| `tags` | string[] | 否 | 每个标签 ≤ 50 字符，最多 5 个，自动去重；第一个类型标签应为 `交互演示` 或 `动态书` |
| `status` | string | 是 | `draft` / `published` / `hidden` / `locked` / `pending_review` |
| `cover` | object | 否 | 封面信息，含 `type`（`image` / `html` / `none`） |
| `created_at` | string | 是 | ISO 8601 时间戳 |
| `updated_at` | string | 是 | ISO 8601 时间戳 |
| `exported_at` | string | 是 | 导出时间 |

### cover 对象

| 字段 | 类型 | 说明 |
|------|------|------|
| `type` | string | `image`（图片封面）、`html`（HTML 封面）、`none`（无封面） |
| `thumbnail_url` | string | 图片封面的 API 路径（仅 type=image 时存在） |
| `has_html_cover` | boolean | 是否有 HTML 封面 |

---

## 编辑规范

### 编辑 source/index.html

- 必须是**完整的 HTML 文档**，包含 `<!DOCTYPE html>`、`<html>`、`<head>`、`<body>`
- 建议使用知见设计系统的 CSS 变量（参考 `design-tokens.md`）
- 作品在沙箱 iframe 中运行，无法访问 Cookie、localStorage 或跨域资源
- 所有资源应内联或使用 CDN 链接（平台不托管外部资源文件）

### 编辑 metadata/work.json

- `schema_version` 修改后必须保持为 `1`
- `title` 是发布时的必填字段
- `tags` 上传时会重新验证：去重、去空、截断超长标签；旧类型 `互动图片视频`、`流程图` 会归并为 `交互演示`
- `id`、`status`、`created_at`、`updated_at`、`exported_at` 在上传时会被忽略或覆盖，修改无意义

### 编辑 metadata/usage.md

- 使用 Markdown 格式
- 通常包含：作品简介、使用方法、技术说明
- 内容会在作品详情页的"使用说明"区域展示

### 编辑 cover/

- 图片封面：替换 `cover/cover.{ext}` 文件，支持 jpg/png/webp/gif
- HTML 封面：修改 `cover/cover.html`，应为自包含的 HTML 片段
- 两种封面在上传时需要分别处理（见下方上传指引）

---

## 常见编辑场景

### 场景 1：修改作品内容

1. 解压作品包
2. 编辑 `source/index.html` 中的 HTML/CSS/JS
3. 在浏览器中打开 `index.html` 预览效果
4. 按上传指引重新上传

### 场景 2：更新标题和描述

1. 解压作品包
2. 编辑 `metadata/work.json` 中的 `title` 和 `brief_description`
3. 同步更新 `metadata/usage.md`（如有变化）
4. 按上传指引重新上传

### 场景 3：更换封面

1. 解压作品包
2. 替换 `cover/cover.{ext}`（图片封面）或修改 `cover/cover.html`（HTML 封面）
3. 更新 `work.json` 中 `cover.type` 字段
4. 按上传指引重新上传

### 场景 4：添加或修改标签

1. 解压作品包
2. 编辑 `metadata/work.json` 中的 `tags` 数组
3. 按上传指引重新上传

---

## 上传指引

编辑完成后，通过知见平台的"上传作品"功能逐文件上传：

### Step 1 · 进入上传页面

访问 `/create`，选择"上传作品"。

### Step 2 · 上传文件

| 文件 | 对应上传项 | 是否必填 |
|------|-----------|----------|
| `source/index.html` | HTML 文件 | 必填 |
| `metadata/work.json` | 元数据 JSON 文件 | 可选 |
| `cover/cover.{ext}` | 封面图片 | 可选 |

### Step 3 · 确认元数据

系统会解析 JSON 文件并预填表单：
- 标题、简介、标签、使用说明会自动填入
- 若 JSON 未提供作品类型，平台会使用创建入口选择的类型或默认 `交互演示`
- 可在表单中手动修改任何字段

### Step 4 · 提交

确认无误后提交，作品会进入草稿或审核流程。

### 注意事项

- 上传 HTML 文件时，系统会尝试从 `<title>` 标签提取标题（如果未提供 JSON）
- 封面图片需要单独上传，不随 JSON 一起解析
- `work.json` 中的 `id` 和 `status` 字段会被忽略
- 如果 JSON 校验失败（如 schema_version 不为 1），系统会跳过元数据导入

---

## 下载作品包

在 `/works/manage` 页面，每个作品卡片上有"下载作品包"按钮。

### 下载限制

- 仅作品创作者可下载
- 管理员不可下载创作者的作品包
- 隐藏状态的作品不可下载

### 文件名格式

下载的文件名为 `{作品标题}-{作品ID}.zip`，标题中的特殊字符会被替换为下划线。
