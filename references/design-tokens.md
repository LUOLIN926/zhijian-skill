# 知见设计语言

## 目录

- 三层设计语言
- 品牌基础色
- 语义 token
- 暗色主题
- 字体、间距与形状
- 动效与无障碍
- 作品使用方式

## 三层设计语言

### 1. 品牌基础层

保持 **自然、温暖、克制、可信**：苔绿表达生长与知识，赭石表达人文温度，大地色建立边界，奶油色提供纸张感。

### 2. 网站 UI 语义层

使用 `surface`、`heading`、`action`、`border` 等语义变量，而不是在组件里直接使用白色或某个品牌色。这样亮色与暗色主题能共享组件规则。

### 3. 作品创意表达层

作品不必复制网站后台界面。可以根据主题扩展图形、色彩和排版，但应保留清晰层级、自然质感、克制装饰和可读交互。每个作品最多选择 1 个主强调色和 1 个辅助强调色。

## 品牌基础色

```css
:root {
  --color-primary: #4A6741;
  --color-primary-dark: #3A5334;
  --color-primary-light: #6B8E63;
  --color-secondary: #A0522D;
  --color-accent: #8B7355;
  --color-earth: #C4A77D;
  --color-cream: #FAF8F5;
  --color-cream-dark: #F5F2ED;
  --color-text: #3D3229;
  --color-text-light: #6B5D4D;
  --color-text-muted: #766958;
  --color-success: #5B8C5A;
  --color-warning: #C9844A;
  --color-danger: #A65D4E;
  --color-sky: #5B8A8A;
  --color-sky-dark: #4A7575;
  --color-autumn: #9B8568;
  --color-autumn-dark: #8B7355;
  --color-lavender: #C2A2D2;
  --color-lavender-dark: #B283B6;
  --color-moss: #6B8E6B;
  --color-rust: #B8765A;
}
```

状态色只表达状态；装饰色只用于分区、数据系列和主题点缀，不承担主要操作。

## 语义 token

```css
:root {
  --color-surface: #FFFFFF;
  --color-surface-raised: #FFFFFF;
  --color-control-bg: #FFFFFF;
  --color-border: #C4A77D;
  --color-heading: #3A5334;
  --color-action: #4A6741;
  --color-action-hover: #3A5334;
  --color-on-solid: #FFFFFF;
  --color-preview-bg: #FFFFFF;
  --color-overlay: rgba(61, 50, 41, 0.4);
  --color-success-text: #3F6F43;
  --color-warning-text: #8A4F1D;
  --color-danger-text: #8B4136;
}
```

| 角色 | 用途 |
|---|---|
| `--color-surface` | 页面内普通表面 |
| `--color-surface-raised` | 卡片、弹窗、悬浮表面 |
| `--color-control-bg` | 输入框和控件背景 |
| `--color-border` | 语义边框；品牌大地色的主题适配值 |
| `--color-heading` | 标题和品牌文字 |
| `--color-action` / `--color-action-hover` | 主要操作及悬停 |
| `--color-on-solid` | 实色按钮和状态块上的文字 |
| `--color-preview-bg` | 需要固定浅色的作品预览 |
| `--color-overlay` | 模态遮罩 |

组件优先使用语义 token；只有图表、装饰和品牌展示直接使用基础色。

## 暗色主题

网站通过 `data-theme="dark"` 设置主题。独立作品不继承父页面属性；如作品需要暗色主题，在自己的文档根元素上设置相同属性或实现独立切换。

```css
:root[data-theme='dark'] {
  --color-cream: #1E1E1E;
  --color-cream-dark: #2D2D2D;
  --color-earth: #5E594F;
  --color-text: #D6D6D6;
  --color-text-light: #C2BCB4;
  --color-text-muted: #A59D92;
  --color-surface: #292929;
  --color-surface-raised: #303030;
  --color-control-bg: #242424;
  --color-border: #5E594F;
  --color-heading: #9DBB94;
  --color-action: #4A6741;
  --color-action-hover: #5B7A53;
  --color-on-solid: #FFFFFF;
  --color-overlay: rgba(0, 0, 0, 0.62);
  --color-primary: #6B8E63;
  --color-primary-dark: #4A6741;
  --color-primary-light: #8AAF82;
  --color-secondary: #C4784A;
  --color-accent: #A89070;
  --color-sky: #7AA8A8;
  --color-sky-dark: #5B8A8A;
  --color-lavender: #D4B8E4;
  --color-lavender-dark: #C2A2D2;
  --color-moss: #8AAF8A;
  --color-rust: #D08A6E;
  --color-success: #7AB87A;
  --color-warning: #E0A060;
  --color-danger: #C47A6A;
  --color-success-text: #9CCB9C;
  --color-warning-text: #F0B778;
  --color-danger-text: #E19B8F;
}
```

不要把 `white` 当作普通表面或按钮文字；分别使用 `surface` 和 `on-solid`。只有固定浅色预览可以使用 `preview-bg`。

## 字体、间距与形状

```css
:root {
  --font-family: 'Noto Serif SC', 'Noto Serif', serif;
  --font-mono: 'JetBrains Mono', 'Fira Code', 'Courier New', monospace;
  --font-weight-regular: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 800;

  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  --spacing-xl: 32px;
  --spacing-2xl: 48px;

  --radius-sm: 8px;
  --radius-md: 12px;
  --radius-lg: 16px;
  --radius-xl: 24px;
  --radius-full: 9999px;

  --shadow-sm: 0 2px 8px rgba(61, 50, 41, 0.08);
  --shadow-md: 0 4px 16px rgba(61, 50, 41, 0.12);
  --shadow-lg: 0 8px 32px rgba(61, 50, 41, 0.16);
}
```

| 元素 | 建议 |
|---|---|
| H1 | `clamp(1.8rem, 4vw, 2.8rem)` / 800 / heading |
| H2 | `clamp(1.4rem, 3vw, 2rem)` / 800 / heading |
| H3 | `clamp(1.1rem, 2vw, 1.4rem)` / 600 / text |
| 正文 | 16px / 行高 1.6–1.8 / text |
| 次级文字 | 14–15px / text-light |
| 提示文字 | 13–14px / text-muted |

## 动效与无障碍

```css
:root {
  --transition-fast: 0.15s ease-out;
  --transition-normal: 0.3s ease-out;
  --transition-slow: 0.5s ease-out;
  --ease-out-quart: cubic-bezier(0.25, 1, 0.5, 1);
  --ease-out-quint: cubic-bezier(0.22, 1, 0.36, 1);
  --touch-target-min: 44px;
}

:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    scroll-behavior: auto !important;
    transition-duration: 0.01ms !important;
  }
}
```

- 不只用颜色传递状态；同时提供文字、图标、形状或位置。
- 可点击区域在触屏上至少 44×44px。
- 动画必须能暂停、重置或在减少动画偏好下变为静态。
- 装饰图形使用 `aria-hidden="true"`；交互控件有可理解名称。

## 作品使用方式

将所需 token 完整定义在作品自己的 `:root`。不要假设 iframe 能继承知见网站变量。若作品只支持亮色，仍使用语义 token，以便之后扩展；若支持暗色，提供作品内切换并同步根元素 `data-theme`。
