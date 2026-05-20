# 知见组件样式模式

以下样式可直接复制使用。所有样式基于设计 tokens，确保风格一致。

---

## 按钮

### 基础按钮样式

```css
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--spacing-sm);
  font-weight: var(--font-weight-bold);
  border-radius: var(--radius-sm);
  transition: background-color var(--transition-fast),
              color var(--transition-fast),
              border-color var(--transition-fast),
              transform var(--transition-fast),
              box-shadow var(--transition-fast),
              opacity var(--transition-fast);
  white-space: nowrap;
  min-height: 40px;
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

### 按钮尺寸

```css
/* 小按钮 */
.btn-sm {
  padding: 6px 12px;
  font-size: 0.85em;
  min-height: 32px;
  font-weight: var(--font-weight-semibold);
}

/* 中按钮 */
.btn-md {
  padding: 10px 16px;
  font-size: 0.95em;
}

/* 大按钮 */
.btn-lg {
  padding: 12px 24px;
  font-size: 1.05em;
}
```

### 按钮变体

```css
/* 主要按钮 */
.btn-primary {
  background: var(--color-primary);
  color: white;
}
.btn-primary:hover:not(:disabled) {
  background: var(--color-primary-dark);
  box-shadow: var(--shadow-md);
}

/* 次级按钮 */
.btn-secondary {
  background: var(--color-cream-dark);
  color: var(--color-text);
  border: 1px solid var(--color-earth);
}
.btn-secondary:hover:not(:disabled) {
  background: var(--color-earth);
  color: white;
}

/* 轮廓按钮 */
.btn-outline {
  background: transparent;
  color: var(--color-primary);
  border: 1.5px solid var(--color-primary);
}
.btn-outline:hover:not(:disabled) {
  background: var(--color-primary);
  color: white;
}

/* 幽灵按钮 */
.btn-ghost {
  background: transparent;
  color: var(--color-text-light);
}
.btn-ghost:hover:not(:disabled) {
  background: var(--color-cream-dark);
  color: var(--color-text);
}

/* CTA 按钮（药丸形） */
.btn-cta {
  background: linear-gradient(135deg, var(--color-primary), var(--color-primary-dark));
  color: white;
  border-radius: var(--radius-full);
  padding: 12px 32px;
  font-size: 1.05em;
  box-shadow: var(--shadow-sm);
}
.btn-cta:hover:not(:disabled) {
  box-shadow: var(--shadow-lg);
  transform: translateY(-1px);
}

/* 危险按钮 */
.btn-danger {
  background: var(--color-danger);
  color: white;
}
.btn-danger:hover:not(:disabled) {
  background: color-mix(in srgb, var(--color-danger) 85%, black);
  box-shadow: var(--shadow-md);
}
```

### 焦点状态

```css
.btn:focus-visible {
  outline: none;
  box-shadow: 0 0 0 3px color-mix(in oklch, var(--color-primary) 15%, transparent);
}
```

---

## 卡片

### 基础卡片

```css
.card {
  background: white;
  border-radius: var(--radius-md);
  border: 1px solid var(--color-cream-dark);
  box-shadow: var(--shadow-sm);
  padding: var(--spacing-md);
  transition: box-shadow var(--transition-normal), transform var(--transition-normal);
}

.card-hoverable:hover {
  box-shadow: var(--shadow-md);
  transform: translateY(-2px);
  cursor: pointer;
}
```

---

## 输入框

### 输入组

```css
.input-group {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-xs);
}

.input-label {
  font-size: 0.85em;
  font-weight: var(--font-weight-semibold);
  color: var(--color-text-light);
}
```

### 输入框

```css
.input-field {
  padding: 10px 14px;
  border: 1px solid var(--color-earth);
  border-radius: var(--radius-sm);
  background: white;
  color: var(--color-text);
  font-size: 0.95em;
  transition: border-color var(--transition-fast), box-shadow var(--transition-fast);
}

.input-field::placeholder {
  color: var(--color-text-muted);
}

.input-field:focus-visible {
  outline: none;
  border-color: var(--color-primary);
  box-shadow: 0 0 0 3px color-mix(in oklch, var(--color-primary) 15%, transparent);
}
```

### 错误状态

```css
.input-field.input-error {
  border-color: var(--color-danger);
}

.input-field.input-error:focus-visible {
  box-shadow: 0 0 0 3px color-mix(in oklch, var(--color-danger) 15%, transparent);
}

.input-error-text {
  font-size: 0.8em;
  color: var(--color-danger);
}
```

### 文本区域

```css
textarea.input-field {
  resize: vertical;
  min-height: 80px;
}
```

---

## 徽章

```css
.badge {
  display: inline-flex;
  align-items: center;
  padding: 3px 10px;
  font-size: 0.75em;
  font-weight: var(--font-weight-semibold);
  border-radius: var(--radius-full);
  white-space: nowrap;
}

.badge-default {
  background: var(--color-cream-dark);
  color: var(--color-text-light);
}

.badge-primary {
  background: color-mix(in oklch, var(--color-primary) 12%, transparent);
  color: var(--color-primary);
}

.badge-success {
  background: color-mix(in oklch, var(--color-success) 12%, transparent);
  color: var(--color-success);
}

.badge-warning {
  background: color-mix(in oklch, var(--color-warning) 12%, transparent);
  color: var(--color-warning);
}

.badge-danger {
  background: color-mix(in oklch, var(--color-danger) 12%, transparent);
  color: var(--color-danger);
}
```

---

## 模态框

```css
.modal {
  border: none;
  background: transparent;
  padding: 0;
  max-width: 90vw;
  max-height: 90vh;
  margin: auto;
  position: fixed;
  inset: 0;
  width: fit-content;
  height: fit-content;
  overflow: visible;
}

.modal[open] {
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal::backdrop {
  background: rgba(61, 50, 41, 0.4);
}

.modal-content {
  background: white;
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-lg);
  min-width: 360px;
  max-width: 520px;
  animation: modal-in 0.2s ease-out;
}

@keyframes modal-in {
  from {
    opacity: 0;
    transform: scale(0.95) translateY(10px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: var(--spacing-md);
  border-bottom: 1px solid var(--color-cream-dark);
}

.modal-close {
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.4em;
  color: var(--color-text-muted);
  border: 1px solid transparent;
  border-radius: var(--radius-sm);
  background: transparent;
  cursor: pointer;
  transition: background-color var(--transition-fast),
              color var(--transition-fast),
              border-color var(--transition-fast);
  line-height: 1;
}

.modal-close:hover {
  background: var(--color-cream-dark);
  color: var(--color-text);
  border-color: var(--color-earth);
}

.modal-body {
  padding: var(--spacing-md);
}
```

---

## 布局区块

### Hero 区块

```css
.hero {
  position: relative;
  text-align: center;
  padding: var(--spacing-2xl) var(--spacing-lg);
  background: var(--color-cream);
  border-radius: var(--radius-xl);
  overflow: hidden;
  min-height: clamp(40vh, 80vh, calc(100vh - 120px));
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.hero-title {
  font-size: clamp(2rem, 4vw, 2.8rem);
  font-weight: var(--font-weight-bold);
  color: var(--color-primary-dark);
  line-height: 1.3;
  letter-spacing: -0.02em;
  margin-bottom: var(--spacing-lg);
}

.hero-intro {
  font-size: 1.1em;
  color: var(--color-text-light);
  max-width: 480px;
  margin: 0 auto var(--spacing-xl);
  line-height: 1.7;
}

.hero-actions {
  display: flex;
  gap: var(--spacing-md);
  justify-content: center;
  flex-wrap: wrap;
}
```

### 两栏布局

```css
.two-column {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: var(--spacing-xl);
  padding: var(--spacing-xl) var(--spacing-lg);
  border-radius: var(--radius-xl);
  align-items: center;
}

@media (max-width: 768px) {
  .two-column {
    grid-template-columns: 1fr;
    gap: var(--spacing-lg);
  }
}
```

### CTA 区块

```css
.cta-section {
  text-align: center;
  padding: var(--spacing-2xl);
  background: var(--color-cream-dark);
  border-radius: var(--radius-xl);
}

.cta-section h2 {
  margin-bottom: var(--spacing-sm);
}

.cta-section p {
  color: var(--color-text-light);
  margin-bottom: var(--spacing-xl);
}
```

---

## 响应式媒体查询模板

```css
/* 小屏手机 */
@media (max-width: 480px) {
  /* 调整字号、间距、布局 */
}

/* 平板 */
@media (max-width: 768px) {
  /* 切换单栏布局 */
}

/* 桌面 */
@media (min-width: 1024px) {
  /* 大屏优化 */
}

/* 减少动画模式 */
@media (prefers-reduced-motion: reduce) {
  * {
    animation: none !important;
    transition: none !important;
  }
}

/* 深色模式 */
@media (prefers-color-scheme: dark) {
  /* 使用 design-tokens.md 中的深色覆盖值 */
}
```