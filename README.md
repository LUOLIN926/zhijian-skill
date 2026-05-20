# 知见创作助手

为知见平台创作和优化知识可视化作品的 Claude Code 技能。

## 安装

1. 下载或克隆本仓库
2. 将 `zhijian-design` 文件夹放入你项目的 `.claude/skills/` 目录下
3. 确保目录结构如下：

```
your-project/
├── .claude/
│   └── skills/
│       └── zhijian-design/
│           ├── SKILL.md
│           └── reference/
│               ├── design-tokens.md
│               ├── component-patterns.md
│               ├── platform-guide.md
│               ├── work-templates.md
│               └── work-package.md
└── ...
```

4. 打开 Claude Code，输入 `/zhijian` 即可开始使用

## 使用方式

在 Claude Code 中输入斜杠命令调用技能：

| 命令 | 说明 |
|------|------|
| `/zhijian create` | 像老师一样规划设计并生成新作品 |
| `/zhijian improve` | 优化已有作品（询问选择级别） |
| `/zhijian improve:minimal` | 最低限度优化：仅技术合规，不改风格 |
| `/zhijian improve:full` | 全面优化：设计系统 + 元数据 |

### zhijian-create

从零创作新作品的完整流程：

1. **需求访谈** — 像老师一样了解你要可视化什么、给谁看、要多深
2. **方案规划** — 规划作品结构、类型选择、内容模块
3. **生成方案文档** — 输出包含作品结构和内容大纲的 Markdown 文档
4. **用户确认** — 你确认方案后才开始实施
5. **实施创作** — 逐一生成作品的完整 HTML 代码
6. **打包输出** — 生成标准作品包（source/ + metadata/ + cover/）

### zhijian-improve

两个优化级别：

**最低限度优化（minimal）**
- 补全 HTML 文档结构
- 移除沙箱不兼容的 API（Cookie、localStorage、跨域请求）
- 将本地资源替换为 CDN 链接
- 不修改任何颜色、字体、间距或布局

**全面优化（full）**
- 替换硬编码颜色为 CSS 变量
- 应用 Noto Serif SC 字体层级
- 标准化间距、圆角、阴影
- 替换组件样式为知见模式
- 生成完整的 work.json 元数据

## 功能

- 应用知见大地色调设计系统（CSS 变量、配色方案）
- 像老师一样规划可视化方案（需求访谈 → 方案文档 → 用户确认）
- 从零创作交互式可视化作品（Canvas/SVG 动画、数据可视化、媒体讲解、流程清单）
- 以两个级别优化已有作品（技术合规 / 全面设计优化）
- 生成标准作品包（source/index.html + metadata/work.json）
- 支持单作品和合集（多个作品组成系列）

## 设计系统概览

### 主色调

| 名称 | 色值 | 用途 |
|------|------|------|
| 苔绿 | `#4A6741` | 主色，核心 CTA 按钮 |
| 赭石 | `#A0522D` | 次色，次级强调 |
| 大地 | `#C4A77D` | 边框、分隔 |
| 米白 | `#FAF8F5` | 页面背景 |

### 装饰色

天蓝 `#5B8A8A`、淡紫 `#c2a2d2`、苔绿 `#6B8E6B`、铁锈 `#B8765A`

### 字体

Noto Serif SC 衬线字体，传递书卷气质。

## 前提条件

- 已安装 [Claude Code](https://claude.ai/download)（CLI 或桌面应用均可）
- 在项目目录下操作

## 知见平台

知见是一个知识可视化创作平台，口号"让知识看得见"。用代码创作交互式作品，将复杂概念转化为直观体验，分享给所有人。

## 许可证

MIT License
