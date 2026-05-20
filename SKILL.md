---
name: zhijian
description: 知见平台一体化创作技能。zhijian-create 像老师一样规划设计并生成新作品，zhijian-improve 以两个级别优化已有作品。当用户提到"知见"、"zhijian"、"知识可视化"、"优化作品"、"创建作品"时使用。
user-invocable: true
argument-hint: "[create|improve|improve:minimal|improve:full]"
---

# Zhijian Skill

## 这个 Skill 做什么

帮助你为知见平台创作和优化知识可视化作品。两个核心功能：

- **zhijian-create** — 像老师一样，从需求分析到方案规划，再到实施创作，生成完整作品
- **zhijian-improve** — 以两个级别优化已有作品：最低限度（仅技术合规）或全面优化（设计系统 + 元数据）

---

## 平台介绍

### 知见是什么

知见是一个知识可视化创作平台，口号"让知识看得见"。核心理念：用代码创作交互式作品，将复杂概念转化为直观体验，分享给所有人。

### 目标用户

- **知识创作者**：教师、教程作者、开发者分享交互式演示
- **学习者**：浏览和发现有趣的可视化内容
- **社区参与者**：点赞、收藏、评论互动

### 核心功能

| 功能 | 说明 |
|------|------|
| 在线编辑器 | 分栏视图（左代码右预览），支持 HTML/CSS/JS 实时预览 |
| 安全沙箱 | 作品在隔离环境运行，无法访问敏感数据 |
| 社区分享 | 发布作品，收获点赞与评论，与他人交流 |
| AI 辅助 | 自动生成标题、描述、封面图片 |
| 合集功能 | 将多个作品组织成主题合集 |
| 收藏夹 | 分类收藏喜欢的作品 |
| 作品包下载 | 下载作品的完整包（HTML + 元数据 + 封面），支持离线编辑和备份 |

### 作品状态

| 状态 | 说明 | 可见范围 |
|------|------|----------|
| draft | 草稿 | 仅作者 |
| pending_review | 待审核 | 仅作者和管理员 |
| published | 已发布 | 所有人 |
| locked | 已锁定 | 仅管理员 |

### 作品类型

每个作品必须有一个类型标签（第一个标签）：

| 类型 | 说明 | 适合 |
|------|------|------|
| 交互演示 | Canvas/SVG 动画、数据可视化、概念模拟 | 动态演示、交互实验 |
| 互动图片视频 | 图片/视频讲解、章节跳转、热点标注 | 图像分析、视频教程 |
| 流程图 | 流程清单、步骤导航、进度追踪 | 操作指南、学习路径 |

### 安全沙箱

作品在隔离的 iframe 中运行，无法访问 Cookie、localStorage 或跨域资源。所有资源应内联或使用 CDN 链接。

---

## 设计系统速查

### 调色板

| 类别 | CSS 变量 | 色值 | 用途 |
|------|----------|------|------|
| 主色 | `--color-primary` | #4A6741 | 核心 CTA 按钮、重点强调 |
| 主色深 | `--color-primary-dark` | #3A5334 | hover 状态 |
| 主色浅 | `--color-primary-light` | #6B8E63 | 背景点缀 |
| 次色 | `--color-secondary` | #A0522D | 次级强调 |
| 强调 | `--color-accent` | #8B7355 | 边框、装饰 |
| 大地 | `--color-earth` | #C4A77D | 边框、分隔 |
| 背景 | `--color-cream` | #FAF8F5 | 页面背景 |
| 背景深 | `--color-cream-dark` | #F5F2ED | 卡片背景、hover |
| 文字 | `--color-text` | #3D3229 | 正文文字 |
| 文字浅 | `--color-text-light` | #6B5D4D | 次级文字 |
| 文字淡 | `--color-text-muted` | #766958 | 提示文字 |

### 装饰色

| 变量 | 色值 | 用途 |
|------|------|------|
| `--color-sky` | #5B8A8A | 天蓝色点缀 |
| `--color-lavender` | #c2a2d2 | 淡紫点缀 |
| `--color-autumn` | #9B8568 | 秋意棕 |
| `--color-moss` | #6B8E6B | 苔绿点缀 |
| `--color-rust` | #B8765A | 铁锈橙 |

### 状态色

| 变量 | 色值 | 用途 |
|------|------|------|
| `--color-success` | #5B8C5A | 成功状态 |
| `--color-warning` | #C9844A | 警告状态 |
| `--color-danger` | #A65D4E | 错误/危险 |

### 字体

```css
font-family: 'Noto Serif SC', 'Noto Serif', serif;
```

| 字重 | CSS 变量 | 值 | 用途 |
|------|----------|-----|------|
| 常规 | `--font-weight-regular` | 400 | 正文 |
| 中等 | `--font-weight-medium` | 500 | 次级强调 |
| 半粗 | `--font-weight-semibold` | 600 | 小标题 |
| 粗体 | `--font-weight-bold` | 800 | 大标题 |

### 间距

| 变量 | 值 | 用途 |
|------|-----|------|
| `--spacing-xs` | 4px | 最小间隙 |
| `--spacing-sm` | 8px | 小间隙 |
| `--spacing-md` | 16px | 标准间隙 |
| `--spacing-lg` | 24px | 大间隙 |
| `--spacing-xl` | 32px | 区块间隙 |
| `--spacing-2xl` | 48px | 大区块间隙 |

### 圆角

| 变量 | 值 | 用途 |
|------|-----|------|
| `--radius-sm` | 8px | 按钮、输入框 |
| `--radius-md` | 12px | 卡片 |
| `--radius-lg` | 16px | 大型展示 |
| `--radius-xl` | 24px | 区块容器 |
| `--radius-full` | 9999px | 药丸形 |

### 阴影

| 变量 | 值 | 用途 |
|------|-----|------|
| `--shadow-sm` | `0 2px 8px rgba(61, 50, 41, 0.08)` | 卡片默认 |
| `--shadow-md` | `0 4px 16px rgba(61, 50, 41, 0.12)` | hover 状态 |
| `--shadow-lg` | `0 8px 32px rgba(61, 50, 41, 0.16)` | 弹窗 |

### 过渡

| 变量 | 值 | 用途 |
|------|-----|------|
| `--transition-fast` | 0.15s ease-out | 快速反馈 |
| `--transition-normal` | 0.3s ease-out | 标准过渡 |
| `--transition-slow` | 0.5s ease-out | 大动作 |

### 缓动曲线

```css
--ease-out-quart: cubic-bezier(0.25, 1, 0.5, 1);
--ease-out-quint: cubic-bezier(0.22, 1, 0.36, 1);
```

---

## 工作流

### zhijian-create — 从零创作

#### Step 1 · 需求访谈

像老师一样，通过多轮对话了解用户需求：

1. **主题**：要可视化什么知识/概念？
2. **深度**：入门概览 / 核心要点 / 深度解析？
3. **受众**：学生 / 专业人士 / 大众？
4. **交互偏好**：纯展示 / 轻交互 / 深度交互？

根据回答追问细节，确保理解用户的真实意图。

#### Step 2 · 方案规划

根据访谈结果，规划可视化方案并以对话方式与用户讨论：

- **是否需要合集**：内容是否适合拆分为多个作品组成系列？如果主题有明确的子模块或递进关系，推荐合集。
- **作品类型选择**：根据内容特性选择最合适的作品类型：
  - 动态过程、数据关系 → 交互演示
  - 图像解读、空间导览 → 互动图片视频
  - 步骤流程、操作清单 → 流程图
- **内容模块**：每个作品需要包含哪些章节/模块
- **呈现逻辑**：推荐的叙事结构和信息组织方式

#### Step 3 · 生成方案文档

将讨论确定的方案写成 Markdown 文档，包含：

```markdown
# 可视化方案：{主题}

## 方案概述
- 主题：
- 目标受众：
- 内容深度：

## 作品结构
- 类型：单作品 / 合集（N 个作品）
- 每个作品的标题、类型、简介

## 内容大纲
### 作品 1：{标题}
- 章节 1：...
- 章节 2：...

## 技术选型
- 渲染方式：Canvas / SVG / DOM
- 外部依赖：无 / CDN 资源

## 视觉方向
- 主色调选择
- 布局风格
```

用户确认方案后进入实施。

#### Step 4 · 实施创作

根据批准的方案，逐一生成作品：

- 应用设计系统（参考 `reference/design-tokens.md`）
- 选择合适模板（参考 `reference/work-templates.md`）
- 生成完整 HTML 代码
- 每个作品完成后展示给用户预览

#### Step 5 · 打包输出

为每个作品生成完整的作品包：

- 生成 `metadata/work.json`（参考 `reference/work-package.md`）
- 组织标准目录结构：
  ```
  作品名/
  ├── source/
  │   └── index.html
  ├── metadata/
  │   ├── work.json
  │   └── usage.md
  └── cover/
  ```
- 如果是合集，为每个作品单独生成作品包

#### Step 6 · 自检清单

- [ ] 颜色使用 CSS 变量而非硬编码
- [ ] 字体符合层级规范（标题粗体、正文常规）
- [ ] 间距使用标准阶进
- [ ] 圆角使用标准值
- [ ] 阴影使用标准值
- [ ] work.json 的 schema_version 为 1
- [ ] title 非空且 ≤ 200 字符
- [ ] tags 不超过 5 个，第一个为作品类型标签
- [ ] index.html 是完整 HTML 文档
- [ ] 无 Cookie / localStorage / 跨域请求
- [ ] 所有资源为内联或 CDN 链接

---

### zhijian-improve — 优化已有作品

调用方式：

- `/zhijian improve` — 询问用户选择级别
- `/zhijian improve:minimal` — 直接执行最低限度优化
- `/zhijian improve:full` — 直接执行全面优化

#### 级别一：最低限度优化（minimal）

**目标**：仅确保作品能在知见平台正常运行，不更改任何内容及风格样式。

**适用场景**：用户有自己的设计风格，只想确保技术兼容性。

**Step 1 · 技术合规检查**

用户提供现有作品的 HTML 代码，检查以下问题：

- 是否为完整 HTML 文档（DOCTYPE、html、head、body）
- 是否使用了 Cookie / localStorage / 跨域请求（沙箱限制）
- 外部资源是否为 CDN 链接（平台不托管外部文件）

**Step 2 · 最小化修正**

- 补全缺失的 HTML 文档结构
- 移除或替换不兼容的 API 调用（Cookie → 无，localStorage → 内存变量，fetch 跨域 → CDN 资源）
- 将本地资源引用替换为 CDN 链接或内联
- **不修改**任何颜色、字体、间距、布局或交互逻辑

**Step 3 · 生成作品包**

- 创建标准目录结构
- 生成 `metadata/work.json`（从内容提取或由用户提供标题、简介、标签）
- 确保第一个标签为作品类型（交互演示 / 互动图片视频 / 流程图）

**Step 4 · 自检清单**

- [ ] index.html 是完整 HTML 文档
- [ ] 无 Cookie / localStorage / 跨域请求
- [ ] 所有资源可访问
- [ ] work.json 格式正确
- [ ] tags 第一个为作品类型

---

#### 级别二：全面优化（full）

**目标**：全面优化内容风格以符合知见设计系统，同时完善作品包结构和元数据。

**适用场景**：用户希望作品完全融入知见平台的视觉风格。

**Step 1 · 分析现有作品**

读取用户提供的 HTML/CSS/JS 代码，识别与知见风格不符的部分：

- 硬编码颜色值 → 替换为 CSS 变量
- 字体不符合层级 → 调整为 Noto Serif SC
- 间距不一致 → 标准化为 4/8/16/24/32/48px
- 圆角/阴影不符合 → 修正为标准值
- 按钮/卡片等组件样式 → 替换为知见组件模式（参考 `reference/component-patterns.md`）

**Step 2 · 生成改进方案**

- 列出需要修改的具体位置
- 提供修改前后的代码对比
- 解释为什么这样改符合知见风格
- 向用户确认后执行

**Step 3 · 应用设计系统**

- 替换颜色为 CSS 变量引用（参考 `reference/design-tokens.md`）
- 应用字体层级规范
- 应用间距、圆角、阴影标准
- 应用组件样式模式（参考 `reference/component-patterns.md`）
- 保持原有功能和交互不变

**Step 4 · 技术合规修正**

同「最低限度优化」Step 1-2 的技术检查和修正。

**Step 5 · 完善元数据**

生成或更新 `metadata/work.json`：

- `title`：从内容提取或优化现有标题（≤ 200 字符）
- `brief_description`：生成简洁准确的简介（≤ 2000 字符）
- `tags`：第一个为类型标签，补充相关标签（≤ 5 个）
- `usage_markdown`：生成使用说明

组织完整作品包结构（参考 `reference/work-package.md`）。

**Step 6 · 自检清单**

- [ ] 颜色使用 CSS 变量而非硬编码
- [ ] 字体符合层级规范
- [ ] 间距使用标准阶进
- [ ] 圆角/阴影使用标准值
- [ ] index.html 是完整 HTML 文档
- [ ] 无 Cookie / localStorage / 跨域请求
- [ ] work.json 格式正确且字段完整
- [ ] tags 第一个为作品类型

---

## 资源文件导览

```
zhijian-design/
├── SKILL.md              ← 你正在读
└── reference/
    ├── design-tokens.md      ← 完整设计系统
    ├── component-patterns.md ← UI 组件样式
    ├── work-templates.md     ← 作品起始模板
    ├── work-package.md       ← 作品包结构和编辑指南
    └── platform-guide.md     ← 平台使用指南
```

---

## 核心设计原则

1. **温暖有机** — 大地色调而非冷峻科技色，像一张用旧的书桌
2. **书卷气息** — Noto Serif SC 衬线字体传递学者气质
3. **克制装饰** — 装饰色点缀而非大面积使用
4. **层级清晰** — 字号、字重、间距配合表达层次
5. **细腻过渡** — ease-out 曲线让动画自然减速
