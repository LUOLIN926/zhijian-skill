# 知见作品模板

以下模板均为完整可运行的 HTML 文件，可直接复制使用。当前起始模板只保留两类：交互演示、动态书。

> 标题、简介、标签等元数据应放在 `metadata/work.json` 中，HTML 专注展示和交互。`tags[0]` 应为 `交互演示` 或 `动态书`。

---

## 交互演示

适合 Canvas/SVG 动画、数据可视化、概念模拟和单屏交互实验。平台会通过固定动态演示 window 展示。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>交互演示</title>
  <style>
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
      --color-sky: #5B8A8A;
      --color-lavender: #c2a2d2;
      --color-moss: #6B8E6B;
      --color-rust: #B8765A;
      --color-success: #5B8C5A;
      --color-warning: #C9844A;
      --color-danger: #A65D4E;
      --font-family: 'Noto Serif SC', 'Noto Serif', serif;
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
      --transition-fast: 0.15s ease-out;
      --transition-normal: 0.3s ease-out;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    body {
      min-height: 100vh;
      padding: var(--spacing-lg);
      font-family: var(--font-family);
      color: var(--color-text);
      background:
        linear-gradient(135deg, rgba(74, 103, 65, 0.12), transparent 42%),
        var(--color-cream);
    }

    button {
      font: inherit;
    }

    .demo-shell {
      width: min(1120px, 100%);
      min-height: calc(100vh - var(--spacing-lg) * 2);
      margin: 0 auto;
      display: grid;
      grid-template-columns: minmax(0, 1fr) 300px;
      gap: var(--spacing-lg);
      align-items: stretch;
    }

    .stage,
    .panel {
      background: white;
      border: 1px solid var(--color-cream-dark);
      border-radius: var(--radius-lg);
      box-shadow: var(--shadow-md);
    }

    .stage {
      display: grid;
      grid-template-rows: minmax(0, 1fr) auto;
      overflow: hidden;
    }

    .canvas-wrap {
      min-height: 420px;
      background: var(--color-cream);
      position: relative;
    }

    canvas {
      display: block;
      width: 100%;
      height: 100%;
    }

    .metrics {
      display: grid;
      grid-template-columns: repeat(3, minmax(0, 1fr));
      border-top: 1px solid var(--color-cream-dark);
    }

    .metric {
      padding: var(--spacing-md);
      border-right: 1px solid var(--color-cream-dark);
    }

    .metric:last-child {
      border-right: none;
    }

    .metric span {
      display: block;
      margin-bottom: 4px;
      color: var(--color-text-muted);
      font-size: 0.8rem;
      font-weight: var(--font-weight-semibold);
    }

    .metric strong {
      color: var(--color-primary-dark);
      font-size: 1.24rem;
    }

    .panel {
      padding: var(--spacing-lg);
      display: grid;
      align-content: start;
      gap: var(--spacing-md);
    }

    .panel h1 {
      color: var(--color-primary-dark);
      font-size: 1.6rem;
      line-height: 1.25;
    }

    .panel p {
      color: var(--color-text-light);
      line-height: 1.75;
    }

    .controls {
      display: grid;
      gap: var(--spacing-sm);
    }

    .btn {
      min-height: 42px;
      padding: 0 16px;
      border: 1px solid transparent;
      border-radius: var(--radius-sm);
      cursor: pointer;
      font-weight: var(--font-weight-bold);
      transition: background var(--transition-fast), border-color var(--transition-fast), transform var(--transition-fast);
    }

    .btn:hover {
      transform: translateY(-1px);
    }

    .btn-primary {
      background: var(--color-primary);
      color: white;
    }

    .btn-secondary {
      background: var(--color-cream);
      border-color: var(--color-earth);
      color: var(--color-text);
    }

    @media (max-width: 820px) {
      body {
        padding: var(--spacing-md);
      }

      .demo-shell {
        grid-template-columns: 1fr;
      }

      .metrics {
        grid-template-columns: 1fr;
      }

      .metric {
        border-right: none;
        border-bottom: 1px solid var(--color-cream-dark);
      }
    }
  </style>
</head>
<body>
  <main class="demo-shell">
    <section class="stage" aria-label="动态演示窗口">
      <div class="canvas-wrap">
        <canvas id="demoCanvas"></canvas>
      </div>
      <div class="metrics">
        <div class="metric"><span>状态</span><strong id="statusText">待开始</strong></div>
        <div class="metric"><span>能量</span><strong id="energyText">0%</strong></div>
        <div class="metric"><span>节点</span><strong id="countText">48</strong></div>
      </div>
    </section>

    <aside class="panel">
      <h1>交互演示标题</h1>
      <p>在这里写观察提示、参数含义和关键结论。交互演示应把注意力集中在动态窗口本身。</p>
      <div class="controls">
        <button class="btn btn-primary" id="startBtn" type="button">开始</button>
        <button class="btn btn-secondary" id="pauseBtn" type="button">暂停</button>
        <button class="btn btn-secondary" id="resetBtn" type="button">重置</button>
      </div>
    </aside>
  </main>

  <script>
    const canvas = document.getElementById('demoCanvas');
    const ctx = canvas.getContext('2d');
    const statusText = document.getElementById('statusText');
    const energyText = document.getElementById('energyText');
    const countText = document.getElementById('countText');
    const points = [];
    let running = false;
    let pointer = { x: 0, y: 0, active: false };

    function seed() {
      points.length = 0;
      const rect = canvas.getBoundingClientRect();
      for (let i = 0; i < 48; i++) {
        points.push({
          x: Math.random() * rect.width,
          y: Math.random() * rect.height,
          vx: (Math.random() - 0.5) * 0.8,
          vy: (Math.random() - 0.5) * 0.8,
          r: 3 + Math.random() * 4
        });
      }
      countText.textContent = String(points.length);
    }

    function fitCanvas() {
      const rect = canvas.getBoundingClientRect();
      const dpr = window.devicePixelRatio || 1;
      canvas.width = Math.round(rect.width * dpr);
      canvas.height = Math.round(rect.height * dpr);
      ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
      seed();
      draw();
    }

    function update() {
      const rect = canvas.getBoundingClientRect();
      let energy = 0;
      points.forEach((point) => {
        if (pointer.active) {
          const dx = pointer.x - point.x;
          const dy = pointer.y - point.y;
          const distance = Math.hypot(dx, dy) || 1;
          if (distance < 180) {
            const pull = (180 - distance) / 180;
            point.vx += (dx / distance) * pull * 0.06;
            point.vy += (dy / distance) * pull * 0.06;
            energy += pull;
          }
        }
        point.x += point.vx;
        point.y += point.vy;
        point.vx *= 0.985;
        point.vy *= 0.985;
        if (point.x < point.r || point.x > rect.width - point.r) point.vx *= -1;
        if (point.y < point.r || point.y > rect.height - point.r) point.vy *= -1;
      });
      energyText.textContent = Math.min(100, Math.round(energy * 8)) + '%';
    }

    function draw() {
      const rect = canvas.getBoundingClientRect();
      ctx.clearRect(0, 0, rect.width, rect.height);
      ctx.fillStyle = '#FAF8F5';
      ctx.fillRect(0, 0, rect.width, rect.height);
      points.forEach((point) => {
        ctx.fillStyle = '#4A6741';
        ctx.beginPath();
        ctx.arc(point.x, point.y, point.r, 0, Math.PI * 2);
        ctx.fill();
      });
    }

    function loop() {
      if (!running) return;
      update();
      draw();
      requestAnimationFrame(loop);
    }

    document.getElementById('startBtn').addEventListener('click', () => {
      if (running) return;
      running = true;
      statusText.textContent = '运行中';
      loop();
    });
    document.getElementById('pauseBtn').addEventListener('click', () => {
      running = false;
      statusText.textContent = '已暂停';
    });
    document.getElementById('resetBtn').addEventListener('click', () => {
      running = false;
      statusText.textContent = '待开始';
      energyText.textContent = '0%';
      seed();
      draw();
    });
    canvas.addEventListener('pointermove', (event) => {
      const rect = canvas.getBoundingClientRect();
      pointer = { x: event.clientX - rect.left, y: event.clientY - rect.top, active: true };
    });
    canvas.addEventListener('pointerleave', () => {
      pointer.active = false;
    });
    window.addEventListener('resize', fitCanvas);
    fitCanvas();
  </script>
</body>
</html>
```

---

## 动态书

适合长文教程、章节叙事、学习路径、实验记录和内嵌动态演示。平台会像网页一样通过上下滚动浏览。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>动态书</title>
  <style>
    :root {
      --color-primary: #4A6741;
      --color-primary-dark: #3A5334;
      --color-secondary: #A0522D;
      --color-earth: #C4A77D;
      --color-cream: #FAF8F5;
      --color-cream-dark: #F5F2ED;
      --color-text: #3D3229;
      --color-text-light: #6B5D4D;
      --color-text-muted: #766958;
      --font-family: 'Noto Serif SC', 'Noto Serif', serif;
      --font-weight-semibold: 600;
      --font-weight-bold: 800;
      --spacing-sm: 8px;
      --spacing-md: 16px;
      --spacing-lg: 24px;
      --spacing-xl: 32px;
      --spacing-2xl: 48px;
      --radius-sm: 8px;
      --radius-md: 12px;
      --shadow-sm: 0 2px 8px rgba(61, 50, 41, 0.08);
      --shadow-md: 0 4px 16px rgba(61, 50, 41, 0.12);
      --transition-fast: 0.15s ease-out;
    }

    * {
      box-sizing: border-box;
    }

    html {
      scroll-behavior: smooth;
    }

    body {
      margin: 0;
      min-height: 100vh;
      font-family: var(--font-family);
      color: var(--color-text);
      background:
        linear-gradient(180deg, rgba(196, 167, 125, 0.24), transparent 360px),
        var(--color-cream);
    }

    .book-shell {
      width: min(940px, 100%);
      margin: 0 auto;
      padding: var(--spacing-2xl) 20px 72px;
    }

    .book-stack {
      display: grid;
      gap: var(--spacing-xl);
    }

    .book-hero,
    .chapter,
    .demo-block,
    .toc {
      background: white;
      border: 1px solid var(--color-cream-dark);
      border-radius: var(--radius-md);
      box-shadow: var(--shadow-sm);
    }

    .book-hero {
      padding: clamp(36px, 7vw, 76px);
    }

    .eyebrow {
      margin: 0 0 12px;
      color: var(--color-primary);
      font-size: 0.82rem;
      font-weight: var(--font-weight-bold);
      letter-spacing: 0.12em;
      text-transform: uppercase;
    }

    h1,
    h2,
    h3,
    p {
      margin-top: 0;
    }

    h1 {
      max-width: 12em;
      margin-bottom: var(--spacing-md);
      font-size: clamp(2.4rem, 8vw, 4.8rem);
      line-height: 1.08;
    }

    h2 {
      color: var(--color-primary-dark);
      font-size: clamp(1.5rem, 4vw, 2.2rem);
    }

    p,
    li {
      color: var(--color-text-light);
      line-height: 1.85;
    }

    .toc {
      padding: var(--spacing-lg);
    }

    .toc nav {
      display: flex;
      flex-wrap: wrap;
      gap: var(--spacing-sm);
    }

    .toc a {
      padding: 8px 12px;
      border-radius: var(--radius-sm);
      background: var(--color-cream);
      color: var(--color-primary-dark);
      font-weight: var(--font-weight-semibold);
      text-decoration: none;
    }

    .chapter {
      padding: clamp(24px, 5vw, 46px);
      scroll-margin-top: var(--spacing-lg);
    }

    .chapter ul {
      display: grid;
      gap: var(--spacing-sm);
      padding-left: var(--spacing-lg);
    }

    .demo-block {
      display: grid;
      grid-template-columns: minmax(0, 1.1fr) minmax(260px, 0.9fr);
      gap: var(--spacing-lg);
      align-items: center;
      padding: var(--spacing-lg);
      scroll-margin-top: var(--spacing-lg);
    }

    .demo-window {
      min-height: 340px;
      border-radius: var(--radius-sm);
      background: linear-gradient(135deg, rgba(74, 103, 65, 0.12), rgba(160, 82, 45, 0.08));
      position: relative;
      overflow: hidden;
    }

    .marker {
      width: 72px;
      height: 72px;
      border-radius: 50%;
      background: var(--color-secondary);
      position: absolute;
      left: 22%;
      top: 42%;
      box-shadow: 0 18px 40px rgba(160, 82, 45, 0.24);
    }

    button {
      min-height: 42px;
      padding: 0 16px;
      border: 1px solid transparent;
      border-radius: var(--radius-sm);
      background: var(--color-primary);
      color: white;
      font: inherit;
      font-weight: var(--font-weight-bold);
      cursor: pointer;
      transition: background var(--transition-fast), transform var(--transition-fast);
    }

    button:hover {
      background: var(--color-primary-dark);
      transform: translateY(-1px);
    }

    @media (max-width: 760px) {
      .demo-block {
        grid-template-columns: 1fr;
      }
    }
  </style>
</head>
<body>
  <main class="book-shell">
    <div class="book-stack">
      <header class="book-hero">
        <p class="eyebrow">Dynamic Book</p>
        <h1>动态书标题</h1>
        <p>用连续页面承载文字、图表和交互演示，让读者像浏览网页一样向下阅读。</p>
      </header>

      <section class="toc" aria-label="目录">
        <nav>
          <a href="#chapter-question">提出问题</a>
          <a href="#chapter-demo">动态观察</a>
          <a href="#chapter-summary">收束结论</a>
        </nav>
      </section>

      <section class="chapter" id="chapter-question">
        <h2>第一章：提出问题</h2>
        <p>把背景、核心问题和阅读目标写在这里。动态书适合更长的叙事结构，也可以在章节中嵌入可交互演示。</p>
        <ul>
          <li>每一章只承载一个清晰的问题或观点。</li>
          <li>用短段落、列表和图示降低阅读负担。</li>
        </ul>
      </section>

      <section class="demo-block" id="chapter-demo" aria-label="内嵌动态演示">
        <div class="demo-window">
          <div class="marker" id="marker"></div>
        </div>
        <div>
          <h2>第二章：动态观察</h2>
          <p>这里可以放 Canvas、SVG 或 DOM 动画。读者不需要离开页面，就能一边阅读一边观察变化。</p>
          <button id="toggleBtn" type="button">暂停</button>
        </div>
      </section>

      <section class="chapter" id="chapter-summary">
        <h2>第三章：收束结论</h2>
        <p>把前面的观察整理成结论，并给出下一步阅读、练习或行动建议。</p>
      </section>
    </div>
  </main>

  <script>
    const marker = document.getElementById('marker');
    const toggleBtn = document.getElementById('toggleBtn');
    let running = true;
    let t = 0;

    function animate() {
      if (running) {
        t += 0.018;
        marker.style.transform = `translate(${Math.sin(t) * 140}px, ${Math.cos(t * 1.4) * 64}px)`;
      }
      requestAnimationFrame(animate);
    }

    toggleBtn.addEventListener('click', () => {
      running = !running;
      toggleBtn.textContent = running ? '暂停' : '继续';
    });

    animate();
  </script>
</body>
</html>
```
