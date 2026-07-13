# 知见作品包规范

## 目录

- 目录结构
- 新建导入清单
- 平台导出快照
- 封面
- 上传与检查

## 目录结构

```text
作品名/
├── source/
│   └── index.html
├── metadata/
│   ├── work.json
│   └── usage.md
└── cover/
    └── cover.png | cover.jpg | cover.webp | cover.html
```

`source/index.html` 必须是完整、自包含的 HTML 文档。`metadata/work.json` 用于预填上传表单；`usage.md` 便于独立阅读，并应与 JSON 中的 `usage_markdown` 保持一致。

## 新建导入清单

为新作品只写平台会导入的字段：

```json
{
  "schema_version": 1,
  "title": "作品标题",
  "brief_description": "一句话说明学习价值",
  "usage_markdown": "## 如何使用\n...",
  "tags": ["交互演示", "主题标签"]
}
```

| 字段 | 规则 |
|---|---|
| `schema_version` | 必须为 `1` |
| `title` | 必填，去除首尾空白后不超过 200 字符 |
| `brief_description` | 可选，不超过 2000 字符 |
| `usage_markdown` | 可选，不超过 200000 字符 |
| `tags` | 最多 5 个、去重；首个类型标签为 `交互演示` 或 `动态书` |

不要为新作品伪造 `id`、`status`、`cover`、`created_at`、`updated_at` 或 `exported_at`。平台上传页会忽略这些导出字段，封面文件需要单独选择。

## 平台导出快照

从平台下载的作品包可能额外包含：

```json
{
  "id": 123,
  "status": "published",
  "cover": {
    "type": "image",
    "thumbnail_url": "/api/v1/uploads/covers/example.png",
    "has_html_cover": false
  },
  "created_at": "...",
  "updated_at": "...",
  "exported_at": "..."
}
```

这些字段是只读快照。优化下载包时保留以便追踪，但不要依赖它们控制重新上传后的 ID、状态或时间。

## 封面

- 图片封面：PNG、JPEG 或 WebP，≤ 5MB，最长边 ≤ 4096px；建议 16:10。
- HTML 封面：完整自包含文档，建议 1200×750，不使用外部脚本或资源。
- 一个最终作品包只放一套封面；没有封面时可以省略 `cover/`。
- GIF 可能出现在旧导出包中，但新生成封面优先使用 PNG/JPEG/WebP。

## 上传与检查

在知见 `/create` 选择作品类型，再分别提供：

1. `source/index.html`
2. `metadata/work.json`（可选但推荐）
3. 图片封面（可选；HTML 封面可在发布助手中生成或粘贴）

上传前检查：

- [ ] HTML 可独立打开且没有控制台错误
- [ ] 标题、简介和使用说明与作品内容一致
- [ ] `schema_version` 为 1
- [ ] 类型标签位于 `tags[0]`
- [ ] 标签不超过 5 个
- [ ] 图片封面大小与尺寸合法，或 HTML 封面自包含
- [ ] 新建清单不含伪造的平台只读字段
