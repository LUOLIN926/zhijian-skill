# 知见设计系统 — 完整 Tokens

## 品牌哲学

**Natural, Grounded, Timeless**

- **温暖有机** — 大地色调而非冷峻科技色
- **书卷气息** — Noto Serif SC 衬线字体传递学者气质
- **克制装饰** — 装饰色点缀而非大面积使用
- **层级清晰** — 字号、字重、间距配合表达层次

---

## CSS 变量完整清单

以下所有变量可在作品中直接引用。使用时需先定义（复制到 `<style>` 块）。

### 品牌色

```css
--color-primary: #4A6741;        /* 苔绿 — 核心按钮、重点强调 */
--color-primary-dark: #3A5334;   /* 苔绿深 — hover 状态 */
--color-primary-light: #6B8E63;  /* 苔绿浅 — 背景点缀 */
```

### 次色与强调

```css
--color-secondary: #A0522D;      /* 赭石 — 次级强调 */
--color-accent: #8B7355;         /* 暖棕 — 边框、装饰 */
--color-earth: #C4A77D;          /* 大地 — 边框、分隔 */
```

### 背景色

```css
--color-cream: #FAF8F5;          /* 奶油 — 页面背景 */
--color-cream-dark: #F5F2ED;     /* 奶油深 — 卡片背景、hover */
```

### 文字色

```css
--color-text: #3D3229;           /* 深棕 — 正文文字 */
--color-text-light: #6B5D4D;     /* 中棕 — 次级文字 */
--color-text-muted: #766958;     /* 浅棕 — 提示文字 */
```

### 装饰色

```css
--color-sky: #5B8A8A;            /* 天蓝 — 点缀色 */
--color-sky-dark: #4A7575;       /* 天蓝深 */
--color-lavender: #c2a2d2;       /* 淡紫 — 点缀色 */
--color-lavender-dark: #b283b6;  /* 淡紫深 */
--color-autumn: #9B8568;         /* 秋棕 */
--color-autumn-dark: #8B7355;    /* 秋棕深 */
--color-moss: #6B8E6B;           /* 苔绿点缀 */
--color-rust: #B8765A;           /* 铁锈橙 */
```

### 状态色

```css
--color-success: #5B8C5A;        /* 成功绿 */
--color-warning: #C9844A;        /* 警告橙 */
--color-danger: #A65D4E;         /* 错误红 */
```

### 字体

```css
--font-family: 'Noto Serif SC', 'Noto Serif', serif;
--font-weight-regular: 400;      /* 正文 */
--font-weight-medium: 500;       /* 次级强调 */
--font-weight-semibold: 600;     /* 小标题 */
--font-weight-bold: 800;         /* 大标题 */
```

### 间距

```css
--spacing-xs: 4px;               /* 最小间隙 */
--spacing-sm: 8px;               /* 小间隙 */
--spacing-md: 16px;              /* 标准间隙 */
--spacing-lg: 24px;              /* 大间隙 */
--spacing-xl: 32px;              /* 区块间隙 */
--spacing-2xl: 48px;             /* 大区块间隙 */
```

### 圆角

```css
--radius-sm: 8px;                /* 按钮、输入框 */
--radius-md: 12px;               /* 卡片 */
--radius-lg: 16px;               /* 大型展示 */
--radius-xl: 24px;               /* 区块容器 */
--radius-full: 9999px;           /* 药丸形 */
```

### 阴影

```css
--shadow-sm: 0 2px 8px rgba(61, 50, 41, 0.08);   /* 卡片默认 */
--shadow-md: 0 4px 16px rgba(61, 50, 41, 0.12);  /* hover 状态 */
--shadow-lg: 0 8px 32px rgba(61, 50, 41, 0.16);  /* 弹窗 */
```

### 过渡

```css
--transition-fast: 0.15s ease-out;   /* 快速反馈 */
--transition-normal: 0.3s ease-out;  /* 标准过渡 */
--transition-slow: 0.5s ease-out;    /* 大动作 */
```

### 缓动曲线

```css
--ease-out-quart: cubic-bezier(0.25, 1, 0.5, 1);
--ease-out-quint: cubic-bezier(0.22, 1, 0.36, 1);
```

### 响应式断点

```css
--bp-sm: 480px;   /* 小屏手机 */
--bp-md: 768px;   /* 平板 */
--bp-lg: 1024px;  /* 桌面 */
```

### 安全区域（刘海屏）

```css
--safe-top: env(safe-area-inset-top, 0px);
--safe-bottom: env(safe-area-inset-bottom, 0px);
--safe-left: env(safe-area-inset-left, 0px);
--safe-right: env(safe-area-inset-right, 0px);
```

### 触摸目标

```css
--touch-target-min: 44px;        /* 最小触摸区域 */
```

---

## 深色模式覆盖

当 `prefers-color-scheme: dark` 时，以下变量自动调整：

```css
@media (prefers-color-scheme: dark) {
  :root {
    /* 背景色反转 */
    --color-cream: #1e1e1e;
    --color-cream-dark: #2d2d2d;
    --color-earth: #333;

    /* 文字色反转 */
    --color-text: #d6d6d6;
    --color-text-light: #b2b2b2;
    --color-text-muted: #8f96a2;

    /* 品牌色调整对比度 */
    --color-primary: #6B8E63;
    --color-primary-dark: #4A6741;
    --color-primary-light: #8AAF82;
    --color-secondary: #C4784A;
    --color-accent: #A89070;

    /* 装饰色调整 */
    --color-sky: #7AA8A8;
    --color-lavender: #D4B8E4;
    --color-moss: #8AAF8A;
    --color-rust: #D08A6E;

    /* 阴影加深 */
    --shadow-sm: 0 2px 8px rgba(0, 0, 0, 0.3);
    --shadow-md: 0 4px 16px rgba(0, 0, 0, 0.4);
    --shadow-lg: 0 8px 32px rgba(0, 0, 0, 0.5);

    /* 状态色调整 */
    --color-success: #7AB87A;
    --color-warning: #E0A060;
    --color-danger: #C47A6A;
  }
}
```

---

## 字体层级规范

| 元素 | 字号 | 字重 | 颜色变量 |
|------|------|------|----------|
| H1 大标题 | `clamp(2rem, 4vw, 2.8rem)` | `--font-weight-bold` | `--color-primary-dark` |
| H2 标题 | `clamp(1.5rem, 3vw, 2rem)` | `--font-weight-bold` | `--color-primary-dark` |
| H3 小标题 | `clamp(1.2rem, 2vw, 1.5rem)` | `--font-weight-semibold` | `--color-text` |
| 正文 | `1rem` (16px) | `--font-weight-regular` | `--color-text` |
| 次级文字 | `0.9rem` | `--font-weight-regular` | `--color-text-light` |
| 提示文字 | `0.85rem` | `--font-weight-regular` | `--color-text-muted` |
| 代码 | `0.9rem` | `--font-weight-regular` | `monospace` |

---

## 间距应用指南

| 场景 | 推荐间距 |
|------|----------|
| 文字行间距 | `line-height: 1.6` |
| 段落间距 | `margin-bottom: --spacing-md` |
| 列表项间距 | `gap: --spacing-sm` |
| 卡片内边距 | `padding: --spacing-md` |
| 区块间距 | `margin: --spacing-xl 0` |
| 页面边距 | `padding: --spacing-lg` |

---

## 圆角应用指南

| 元素 | 推荐圆角 |
|------|----------|
| 按钮 | `--radius-sm` (8px) |
| 输入框 | `--radius-sm` (8px) |
| 卡片 | `--radius-md` (12px) |
| 大型展示容器 | `--radius-lg` (16px) |
| 页面区块 | `--radius-xl` (24px) |
| 药丸形按钮 | `--radius-full` |

---

## 阴影应用指南

| 场景 | 推荐阴影 |
|------|----------|
| 卡片默认 | `--shadow-sm` |
| 卡片 hover | `--shadow-md` |
| 弹窗/模态框 | `--shadow-lg` |

---

## 完整 CSS 变量定义代码

复制以下代码到作品的 `<style>` 块即可使用所有变量：

```css
:root {
  /* 品牌色 */
  --color-primary: #4A6741;
  --color-primary-dark: #3A5334;
  --color-primary-light: #6B8E63;

  /* 次色与强调 */
  --color-secondary: #A0522D;
  --color-accent: #8B7355;
  --color-earth: #C4A77D;

  /* 背景色 */
  --color-cream: #FAF8F5;
  --color-cream-dark: #F5F2ED;

  /* 文字色 */
  --color-text: #3D3229;
  --color-text-light: #6B5D4D;
  --color-text-muted: #766958;

  /* 装饰色 */
  --color-sky: #5B8A8A;
  --color-sky-dark: #4A7575;
  --color-lavender: #c2a2d2;
  --color-lavender-dark: #b283b6;
  --color-autumn: #9B8568;
  --color-moss: #6B8E6B;
  --color-rust: #B8765A;

  /* 状态色 */
  --color-success: #5B8C5A;
  --color-warning: #C9844A;
  --color-danger: #A65D4E;

  /* 字体 */
  --font-family: 'Noto Serif SC', 'Noto Serif', serif;
  --font-weight-regular: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 800;

  /* 间距 */
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  --spacing-xl: 32px;
  --spacing-2xl: 48px;

  /* 圆角 */
  --radius-sm: 8px;
  --radius-md: 12px;
  --radius-lg: 16px;
  --radius-xl: 24px;
  --radius-full: 9999px;

  /* 阴影 */
  --shadow-sm: 0 2px 8px rgba(61, 50, 41, 0.08);
  --shadow-md: 0 4px 16px rgba(61, 50, 41, 0.12);
  --shadow-lg: 0 8px 32px rgba(61, 50, 41, 0.16);

  /* 过渡 */
  --transition-fast: 0.15s ease-out;
  --transition-normal: 0.3s ease-out;
  --transition-slow: 0.5s ease-out;

  /* 缓动 */
  --ease-out-quart: cubic-bezier(0.25, 1, 0.5, 1);
  --ease-out-quint: cubic-bezier(0.22, 1, 0.36, 1);
}
```