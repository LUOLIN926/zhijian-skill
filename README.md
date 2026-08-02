# 知见创作助手

面向 Codex 与 Claude Code 的知识可视化创作 Skill。它会先通过访谈锁定教学目标和作品方案，再生成可上传到知见平台的交互演示、动态书、元数据与封面。

## 核心能力

- 自适应多轮访谈与创作简报确认
- 交互演示、动态书和可直接上传的完整合集创作
- 知见设计语言、响应式和无障碍规范
- 从零创作或 minimal/full 两级优化
- Codex 默认 ImageGen 图片封面与六种 HTML 封面回退
- 标准单作品/合集文件夹和上传兼容元数据

## 在 Codex 中安装

Codex 会从仓库级和用户级 `.agents/skills` 目录发现 Skill。

项目级安装：

```text
your-project/
└── .agents/
    └── skills/
        └── zhijian/
            ├── SKILL.md
            ├── agents/openai.yaml
            └── references/
```

将本仓库复制或克隆到 `.agents/skills/zhijian`。如果希望在所有项目使用，安装到 `~/.agents/skills/zhijian`。

在 Codex CLI/IDE 中输入 `$` 选择 `zhijian`，或直接描述任务让 Codex 根据 Skill 描述自动匹配：

```text
$zhijian 帮我做一个面向高中生的引力波交互演示。
```

如果新安装的 Skill 没有出现，重启 Codex。详细机制见 [Codex Skills 官方文档](https://developers.openai.com/codex/skills/)。

## 在 Claude Code 中安装

将本仓库复制或克隆到项目的 `.claude/skills/zhijian`：

```text
your-project/
└── .claude/
    └── skills/
        └── zhijian/
            ├── SKILL.md
            └── references/
```

在 Claude Code 中输入 `/zhijian`，或直接要求使用知见技能创建/优化作品。

## 使用示例

```text
$zhijian 创建一个让初学者理解 MACD 的交互演示。
$zhijian 把这份量子计算课程改造成动态书。
$zhijian 创建一套包含交互演示和动态书的股票入门合集，并输出可直接上传的完整文件夹。
$zhijian improve:minimal 检查这个 HTML 是否兼容知见。
$zhijian improve:full 优化这个作品并重新制作封面。
```

创建流程固定为：访谈 → 创作简报 → 用户确认 → 制作 → 封面 → 打包 → 验证。未确认简报前，Skill 不会直接生成成品。

## 输出结构

```text
作品名/
├── source/index.html
├── metadata/work.json
├── metadata/usage.md（仅交互演示，可选）
└── cover/cover.png | cover.jpg | cover.webp | cover.html
```

在 Codex 中，确认创作简报后会默认调用 ImageGen 生成图片封面，无需再次选择封面路线；默认不把标题写进图片。用户明确要求确定性排版、ImageGen 不可用或图片验证失败时，回退到文雅纸本、结构网格、清透留白、杂志头版、数据大字报或流程蓝图 HTML 封面。

合集输出为一个根目录，其中 `metadata/collection.json` 描述章节和作品顺序，`metadata/introduction.md` 保存完整介绍，`cover/` 保存合集封面，`works/` 下保存全部标准作品包。用户在知见“直接上传”中选择整个根目录即可。

## 许可证

MIT License
