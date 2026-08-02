# 知见组件模式

## 目录

- 通用规则
- 按钮
- 卡片
- 输入
- 标识与状态
- 弹窗与提示
- 作品专用组件

## 通用规则

- 组件使用 `references/design-tokens.md` 的语义变量。
- 交互状态至少覆盖默认、hover、focus-visible、disabled；需要时覆盖 loading 和 error。
- hover 只做增强，核心信息和操作不能依赖 hover。
- 图标按钮提供 `aria-label`；表单错误通过 `aria-invalid`、`aria-describedby` 关联。
- 减少动画偏好下关闭位移、循环和装饰动画。

## 按钮

```css
.btn {
  display: inline-flex;
  min-height: 40px;
  align-items: center;
  justify-content: center;
  gap: var(--spacing-sm);
  padding: 10px 16px;
  border: 1px solid transparent;
  border-radius: var(--radius-sm);
  font: var(--font-weight-bold) 0.95rem/1 var(--font-family);
  cursor: pointer;
  transition: background var(--transition-fast), color var(--transition-fast),
    border-color var(--transition-fast), box-shadow var(--transition-fast),
    transform var(--transition-fast);
}

.btn-primary { background: var(--color-action); color: var(--color-on-solid); }
.btn-primary:hover:not(:disabled) { background: var(--color-action-hover); box-shadow: var(--shadow-md); }
.btn-secondary { background: var(--color-cream-dark); color: var(--color-text); border-color: var(--color-border); }
.btn-secondary:hover:not(:disabled) { background: var(--color-action); color: var(--color-on-solid); }
.btn-outline { background: transparent; color: var(--color-primary); border-color: var(--color-primary); }
.btn-outline:hover:not(:disabled) { background: var(--color-action); color: var(--color-on-solid); }
.btn-ghost { background: transparent; color: var(--color-text-light); }
.btn-ghost:hover:not(:disabled) { background: var(--color-cream-dark); color: var(--color-text); }
.btn-danger { background: var(--color-danger); color: var(--color-on-solid); }
.btn:disabled { opacity: .5; cursor: not-allowed; }
.btn:focus-visible { outline: none; box-shadow: 0 0 0 3px color-mix(in oklch, var(--color-primary) 15%, transparent); }
```

每个视图只设置一个主要行动。次级行动使用 secondary/outline，文字级行动使用 ghost，破坏性操作使用 danger。

## 卡片

```css
.card {
  padding: var(--spacing-md);
  border: 1px solid var(--color-cream-dark);
  border-radius: var(--radius-md);
  background: var(--color-surface-raised);
  box-shadow: var(--shadow-sm);
  transition: box-shadow var(--transition-normal), transform var(--transition-normal);
}

.card-hoverable:hover {
  transform: translateY(-2px);
  box-shadow: var(--shadow-md);
}
```

整卡可点击时使用链接或按钮语义，并提供 focus-visible；不要只给 `div` 添加点击事件。

## 输入

```css
.input-group { display: flex; flex-direction: column; gap: var(--spacing-xs); }
.input-label { color: var(--color-text-light); font-size: .85rem; font-weight: var(--font-weight-semibold); }
.input-field {
  min-height: 40px;
  padding: 10px 14px;
  border: 1px solid var(--color-border);
  border-radius: var(--radius-sm);
  background: var(--color-control-bg);
  color: var(--color-text);
  font: inherit;
  transition: border-color var(--transition-fast), box-shadow var(--transition-fast);
}
.input-field::placeholder { color: var(--color-text-muted); }
.input-field:focus-visible { outline: none; border-color: var(--color-primary); box-shadow: 0 0 0 3px color-mix(in oklch, var(--color-primary) 15%, transparent); }
.input-field[aria-invalid='true'] { border-color: var(--color-danger); }
.input-error-text { color: var(--color-danger-text); font-size: .8rem; }
.input-hint-text { color: var(--color-text-muted); font-size: .8rem; }
```

标签保持可见，不用 placeholder 代替 label。滑块和参数控制同时显示名称、当前值和合理范围。

## 标识与状态

```css
.badge {
  display: inline-flex;
  align-items: center;
  padding: 3px 10px;
  border-radius: var(--radius-full);
  font-size: .75rem;
  font-weight: var(--font-weight-semibold);
}
.badge-default { background: var(--color-cream-dark); color: var(--color-text-light); }
.badge-primary { background: color-mix(in oklch, var(--color-primary) 12%, transparent); color: var(--color-primary); }
.badge-success { background: color-mix(in oklch, var(--color-success) 12%, transparent); color: var(--color-success-text); }
.badge-warning { background: color-mix(in oklch, var(--color-warning) 12%, transparent); color: var(--color-warning-text); }
.badge-danger { background: color-mix(in oklch, var(--color-danger) 12%, transparent); color: var(--color-danger-text); }
.badge-work-type {
  border: 1px solid color-mix(in oklch, var(--color-secondary) 42%, var(--color-surface));
  background: color-mix(in oklch, var(--color-secondary) 13%, var(--color-surface));
  color: var(--color-secondary);
}
```

状态文字使用专用 `*-text` 变量，避免亮色状态值在浅色背景上对比不足。

## 弹窗与提示

弹窗使用原生 `<dialog>` 或实现等价的焦点管理、Escape 关闭和背景点击策略。

```css
.modal::backdrop { background: var(--color-overlay); }
.modal-content {
  width: min(520px, calc(100vw - 32px));
  border-radius: var(--radius-md);
  background: var(--color-surface-raised);
  box-shadow: var(--shadow-lg);
}

.toast {
  display: flex;
  align-items: center;
  gap: var(--spacing-sm);
  padding: 12px 20px;
  border-radius: var(--radius-sm);
  color: var(--color-on-solid);
  box-shadow: var(--shadow-md);
}
.toast-success { background: var(--color-success); }
.toast-error { background: var(--color-danger); }
.toast-info { background: var(--color-primary); }
```

错误提示使用 `role="alert"`，普通状态使用 `role="status"`。自动消失提示需要允许鼠标或键盘聚焦时暂停。

## 作品专用组件

### 参数控制台

- 每个控制项包含名称、当前值、单位和范围。
- 参数变化后立即更新图形与结论，避免“点应用”造成割裂。
- 提供重置；复杂动画提供播放/暂停和当前阶段。

### 指标卡

- 指标名称使用 text-muted，数值使用 heading 或主题强调色。
- 变化值同时显示方向图标和文字，不只使用红绿。
- 不用超过 4 张等权卡片挤占演示区域。

### 注释与结论

- 把“观察什么”放在交互附近，把“为什么”放在解释区。
- 结论随状态更新时使用 `aria-live="polite"`。
- 公式、图例和单位与视觉编码保持一致。

### 动态书章节

- 动态书采用电子书或新闻长文结构：连续正文、清晰标题层级、舒适行长和充分留白。不要复用交互演示的固定 1280×720 舞台、左右控制台、指标卡矩阵或装饰卡片墙。
- 每个可导航章节使用 `<section data-zhijian-chapter id="稳定且唯一的英文或拼音标识">`，章节内第一个 `h2` 是平台目录标题；章节标签、正文段落和演示中的 `h3` 不会成为新章节。
- 每章只有一个主问题，先给情境，再给解释、证据或演示，最后给小结。正文只保留文字、必要图片与图注，以及确实服务于观察、比较、推演或验证的 HTML 交互。
- 不在正文创建章节目录，包括 `nav[aria-label="章节导航"]`、`.chapter-nav` 或等价锚点列表；平台会从章节分节生成唯一导航。也不要复制平台作品标题、简介、作者、标签或互动操作。
- 图片必须承担解释、证据或示意作用并提供准确 `alt`/图注；纯装饰图片省略。内嵌演示使用 `h3` 或更低层级标题，提供替代说明、可恢复状态和键盘操作。
- 章节容器设置合理 `scroll-margin-top`；窄屏不横向溢出，正文建议保持约 680–780px 的阅读宽度。
