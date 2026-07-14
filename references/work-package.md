# 知见作品与合集包规范

## 单作品目录

```text
作品名/
├── source/index.html
├── metadata/work.json
├── metadata/usage.md
└── cover/cover.png | cover.jpg | cover.webp | cover.html
```

`source/index.html` 必须是完整、自包含的 HTML 文档。`work.json` 是平台元数据主来源；其 `usage_markdown` 为空时使用 `usage.md`。

```json
{
  "schema_version": 1,
  "title": "作品标题",
  "brief_description": "一句话说明学习价值",
  "usage_markdown": "## 如何使用\n...",
  "tags": ["交互演示", "主题标签"]
}
```

- 标题必填且不超过 200 字符；简介不超过 2000 字符；使用说明不超过 200000 字符。
- 标签去重后最多 5 个；唯一类型标签必须是 `交互演示` 或 `动态书`，并放在首位。
- HTML 不超过 4MB；图片封面限 PNG/JPEG/WebP、5MB、最长边 4096px；HTML 封面不超过 32KB。

## 完整合集目录

```text
合集名/
├── metadata/
│   ├── collection.json
│   └── introduction.md
├── cover/
│   └── cover.png | cover.jpg | cover.webp | cover.html
└── works/
    ├── 01-作品目录/
    │   ├── source/index.html
    │   ├── metadata/work.json
    │   ├── metadata/usage.md
    │   └── cover/cover.png | cover.jpg | cover.webp | cover.html
    └── 02-作品目录/
        └── ...
```

每个 `works/` 直接子目录都是完整、可独立运行和审核的标准作品包。目录名是合集清单中的稳定作品键，不在目录名中使用 `/` 或 `\`。

### collection.json

```json
{
  "schema_version": 1,
  "name": "合集名称",
  "brief_description": "用于卡片和搜索的短简介",
  "introduction_markdown": "# 合集完整介绍\n...",
  "tags": ["主题标签"],
  "chapters": [
    {
      "title": "第一章",
      "works": ["01-作品目录"],
      "children": [
        {
          "title": "第一节",
          "works": ["02-作品目录"],
          "children": []
        }
      ]
    }
  ]
}
```

- `schema_version` 必须为 `1`；名称不超过 200 字符，短简介不超过 2000 字符，完整介绍不超过 200000 字符。
- `introduction_markdown` 非空时优先使用，否则读取必需的 `metadata/introduction.md`。
- 合集标签最多 10 个，每个不超过 50 字符；合集本身不使用作品类型标签。
- `chapters` 是有序树，最多 100 个章节和 3 层；每个节点同时允许包含作品与子章节。
- `works` 只能引用 `works/` 下的直接子目录名；数组顺序就是章节内作品顺序。
- 合集至少包含 1 个、最多 50 个作品。每个作品必须且只能被引用一次，不允许重复、遗漏或未知引用。
- 合集封面可省略，但 Skill 默认在确认后生成一套；图片合集封面不使用 GIF。

## 平台导出只读字段

平台下载包可能包含 `id`、`status`、`cover`、`created_at`、`updated_at` 或 `exported_at`。这些字段只用于追踪导出来源，重新导入时不会控制平台 ID、状态、封面或时间。新建包不要伪造这些字段。

## 验证与上传

交付前同时执行单作品验证和合集交叉验证：

- [ ] 所有 HTML 可独立打开，无控制台错误且满足大小限制
- [ ] 每个作品只有一种有效类型，类型标签已规范到 `tags[0]`
- [ ] 每个作品和合集最多保留一套最终封面
- [ ] `collection.json` 引用与 `works/` 目录一一对应
- [ ] 章节深度、章节数、作品数和标签数合法
- [ ] JSON 元数据与 Markdown 独立文件内容一致；不一致时明确 JSON 为主
- [ ] 隐藏文件和无关附件不参与结构

在知见创作中心选择“直接上传”，选择单作品或合集的整个根目录。浏览器会自动识别结构并进入确认页；不要逐项选择文件，也不要把多个散装作品的上级目录当作合集上传。第一版不支持 ZIP。
