# 知见作品模板

## 目录

- 使用原则
- 交互演示模板
- 动态书模板

## 使用原则

- 模板只是结构起点；根据创作简报替换内容、状态和图形，不要交付占位示例。
- 标题、简介、标签放在 `metadata/work.json`。作品内部只保留理解内容所需的章节标题和观察提示，不创建重复的平台标题横幅。
- 作品在 iframe 中运行，必须完整定义自己的 token 和样式。
- 交互演示以 1280×720 为设计视口；动态书以响应式滚动页面为主。

## 交互演示模板

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>交互演示</title>
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
      --color-surface: #FFFFFF;
      --color-surface-raised: #FFFFFF;
      --color-control-bg: #FFFFFF;
      --color-border: #C4A77D;
      --color-heading: #3A5334;
      --color-action: #4A6741;
      --color-action-hover: #3A5334;
      --color-on-solid: #FFFFFF;
      --font-family: 'Noto Serif SC', 'Noto Serif', serif;
      --spacing-xs: 4px;
      --spacing-sm: 8px;
      --spacing-md: 16px;
      --spacing-lg: 24px;
      --radius-sm: 8px;
      --radius-md: 12px;
      --radius-lg: 16px;
      --shadow-sm: 0 2px 8px rgba(61, 50, 41, .08);
      --shadow-md: 0 4px 16px rgba(61, 50, 41, .12);
      --transition-fast: .15s ease-out;
    }

    *, *::before, *::after { box-sizing: border-box; }
    html, body { width: 100%; min-height: 100%; margin: 0; }
    body {
      padding: var(--spacing-lg);
      color: var(--color-text);
      background: var(--color-cream);
      font-family: var(--font-family);
    }
    button, input { font: inherit; }
    button:focus-visible, input:focus-visible {
      outline: 2px solid var(--color-primary);
      outline-offset: 2px;
    }

    .demo-shell {
      width: min(1232px, 100%);
      min-height: calc(100vh - 48px);
      margin: 0 auto;
      display: grid;
      grid-template-columns: minmax(0, 1fr) 300px;
      gap: var(--spacing-lg);
    }
    .stage, .panel {
      border: 1px solid var(--color-cream-dark);
      border-radius: var(--radius-lg);
      background: var(--color-surface-raised);
      box-shadow: var(--shadow-sm);
    }
    .stage {
      min-height: 0;
      display: grid;
      grid-template-rows: minmax(420px, 1fr) auto;
      overflow: hidden;
    }
    .visual {
      position: relative;
      min-height: 420px;
      overflow: hidden;
      background:
        radial-gradient(circle at 25% 25%, rgba(74, 103, 65, .15), transparent 36%),
        var(--color-cream);
    }
    canvas { display: block; width: 100%; height: 100%; }
    .metrics { display: grid; grid-template-columns: repeat(3, 1fr); border-top: 1px solid var(--color-cream-dark); }
    .metric { padding: var(--spacing-md); border-right: 1px solid var(--color-cream-dark); }
    .metric:last-child { border-right: 0; }
    .metric span { display: block; color: var(--color-text-muted); font-size: .8rem; }
    .metric strong { color: var(--color-heading); font-size: 1.25rem; }
    .panel { padding: var(--spacing-lg); align-content: start; display: grid; gap: var(--spacing-md); }
    .panel h2 { margin: 0; color: var(--color-heading); font-size: 1.35rem; }
    .panel p { margin: 0; color: var(--color-text-light); line-height: 1.7; }
    .controls { display: grid; gap: var(--spacing-sm); }
    .control-label { display: flex; justify-content: space-between; gap: var(--spacing-sm); color: var(--color-text-light); font-size: .9rem; }
    input[type='range'] { width: 100%; accent-color: var(--color-primary); }
    .actions { display: grid; grid-template-columns: 1fr 1fr; gap: var(--spacing-sm); }
    .btn {
      min-height: 44px;
      border: 1px solid var(--color-border);
      border-radius: var(--radius-sm);
      background: var(--color-cream-dark);
      color: var(--color-text);
      cursor: pointer;
      font-weight: 700;
      transition: background var(--transition-fast), transform var(--transition-fast);
    }
    .btn-primary { border-color: transparent; background: var(--color-action); color: var(--color-on-solid); }
    .btn:hover { transform: translateY(-1px); }
    .btn-primary:hover { background: var(--color-action-hover); }
    .observation { padding: var(--spacing-md); border-left: 3px solid var(--color-secondary); background: var(--color-cream-dark); line-height: 1.65; }

    @media (max-width: 820px) {
      body { padding: var(--spacing-md); }
      .demo-shell { grid-template-columns: 1fr; }
      .stage { grid-template-rows: minmax(360px, 56vh) auto; }
    }
    @media (max-width: 520px) {
      .metrics { grid-template-columns: 1fr; }
      .metric { border-right: 0; border-bottom: 1px solid var(--color-cream-dark); }
      .actions { grid-template-columns: 1fr; }
    }
    @media (prefers-reduced-motion: reduce) {
      *, *::before, *::after { animation-duration: .01ms !important; transition-duration: .01ms !important; }
    }
  </style>
</head>
<body>
  <main class="demo-shell">
    <section class="stage" aria-label="主题动态演示">
      <div class="visual"><canvas id="canvas" aria-label="主题关系图"></canvas></div>
      <div class="metrics" aria-live="polite">
        <div class="metric"><span>参数</span><strong id="metricA">0.50</strong></div>
        <div class="metric"><span>结果</span><strong id="metricB">50%</strong></div>
        <div class="metric"><span>状态</span><strong id="status">观察中</strong></div>
      </div>
    </section>
    <aside class="panel" aria-label="观察控制台">
      <h2>观察提示</h2>
      <p>说明用户要改变什么、观察什么，以及变化代表的知识含义。</p>
      <div class="controls">
        <label class="control-label" for="parameter"><span>核心参数</span><output id="parameterValue">0.50</output></label>
        <input id="parameter" type="range" min="0" max="100" value="50" aria-describedby="observation">
      </div>
      <div class="actions">
        <button class="btn btn-primary" id="play" type="button">播放</button>
        <button class="btn" id="reset" type="button">重置</button>
      </div>
      <div class="observation" id="observation" aria-live="polite">拖动参数，比较图形与结果如何同步变化。</div>
    </aside>
  </main>
  <script>
    const canvas = document.querySelector('#canvas');
    const context = canvas.getContext('2d');
    const parameter = document.querySelector('#parameter');
    const parameterValue = document.querySelector('#parameterValue');
    const metricA = document.querySelector('#metricA');
    const metricB = document.querySelector('#metricB');
    const status = document.querySelector('#status');
    const reduceMotion = matchMedia('(prefers-reduced-motion: reduce)').matches;
    let playing = false;
    let frame = 0;

    function fit() {
      const rect = canvas.getBoundingClientRect();
      const dpr = Math.min(devicePixelRatio || 1, 2);
      canvas.width = Math.round(rect.width * dpr);
      canvas.height = Math.round(rect.height * dpr);
      context.setTransform(dpr, 0, 0, dpr, 0, 0);
      draw();
    }
    function draw() {
      const rect = canvas.getBoundingClientRect();
      const value = Number(parameter.value) / 100;
      context.clearRect(0, 0, rect.width, rect.height);
      context.fillStyle = '#FAF8F5';
      context.fillRect(0, 0, rect.width, rect.height);
      context.fillStyle = '#4A6741';
      context.beginPath();
      context.arc(rect.width * (.25 + value * .5), rect.height / 2, 26 + value * 42, 0, Math.PI * 2);
      context.fill();
      parameterValue.value = value.toFixed(2);
      metricA.textContent = value.toFixed(2);
      metricB.textContent = Math.round(value * 100) + '%';
    }
    function loop() {
      if (!playing || reduceMotion) return;
      frame += 1;
      parameter.value = String((Math.sin(frame / 45) * .5 + .5) * 100);
      draw();
      requestAnimationFrame(loop);
    }
    parameter.addEventListener('input', draw);
    document.querySelector('#play').addEventListener('click', () => {
      playing = !playing;
      status.textContent = playing ? '播放中' : '已暂停';
      document.querySelector('#play').textContent = playing ? '暂停' : '播放';
      if (playing) loop();
    });
    document.querySelector('#reset').addEventListener('click', () => {
      playing = false;
      parameter.value = '50';
      status.textContent = '观察中';
      document.querySelector('#play').textContent = '播放';
      draw();
    });
    new ResizeObserver(fit).observe(canvas);
  </script>
</body>
</html>
```

## 动态书模板

动态书正文不生成目录。平台按 DOM 顺序读取 `<section data-zhijian-chapter id="...">`，并取该分节内第一个 `h2` 作为目录标题；每个 `id` 必须唯一。演示、图片说明和小节标题使用 `h3` 或更低层级，避免被当作章节。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
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
      --color-surface: #FFFFFF;
      --color-surface-raised: #FFFFFF;
      --color-border: #C4A77D;
      --color-heading: #3A5334;
      --color-action: #4A6741;
      --color-on-solid: #FFFFFF;
      --font-family: 'Noto Serif SC', 'Noto Serif', serif;
      --spacing-sm: 8px;
      --spacing-md: 16px;
      --spacing-lg: 24px;
      --spacing-xl: 32px;
      --spacing-2xl: 48px;
      --radius-sm: 8px;
      --radius-md: 12px;
      --radius-lg: 16px;
      --shadow-sm: 0 2px 8px rgba(61, 50, 41, .08);
    }
    *, *::before, *::after { box-sizing: border-box; }
    html { scroll-behavior: smooth; }
    body { margin: 0; background: var(--color-surface); color: var(--color-text); font: 16px/1.8 var(--font-family); }
    a { color: var(--color-primary); }
    a:focus-visible, button:focus-visible { outline: 2px solid var(--color-primary); outline-offset: 3px; }
    .book { width: min(760px, calc(100% - 32px)); margin: 0 auto; padding: clamp(24px, 5vw, 56px) 0 96px; }
    .chapter { scroll-margin-top: 24px; padding: clamp(36px, 7vw, 64px) 0; border-bottom: 1px solid var(--color-cream-dark); }
    .chapter:first-of-type { padding-top: 0; }
    .chapter:last-of-type { border: 0; }
    .chapter-label { margin: 0 0 var(--spacing-sm); color: var(--color-secondary); font-size: .8rem; font-weight: 700; letter-spacing: .12em; text-transform: uppercase; }
    h2 { margin: 0 0 var(--spacing-md); color: var(--color-heading); font-size: clamp(1.55rem, 4vw, 2.4rem); line-height: 1.25; }
    p { margin: 0 0 var(--spacing-md); }
    .lead { color: var(--color-text-light); font-size: 1.08rem; }
    .insight { margin: var(--spacing-lg) 0; padding: var(--spacing-md) var(--spacing-lg); border-left: 4px solid var(--color-primary); background: var(--color-cream-dark); }
    h3 { margin: var(--spacing-xl) 0 var(--spacing-sm); color: var(--color-text); font-size: 1.2rem; }
    figure { margin: var(--spacing-xl) 0; }
    figure img { display: block; width: 100%; height: auto; border-radius: var(--radius-sm); }
    figcaption { margin-top: var(--spacing-sm); color: var(--color-text-muted); font-size: .85rem; line-height: 1.6; }
    .demo { min-height: 320px; margin: var(--spacing-md) 0 var(--spacing-xl); display: grid; place-items: center; border-top: 1px solid var(--color-border); border-bottom: 1px solid var(--color-border); background: linear-gradient(135deg, rgba(74, 103, 65, .08), transparent); }
    .demo button { min-height: 44px; padding: 10px 18px; border: 0; border-radius: var(--radius-sm); background: var(--color-action); color: var(--color-on-solid); font: 700 1rem var(--font-family); cursor: pointer; }
    @media (max-width: 560px) { .book { width: min(100% - 24px, 760px); padding-top: 20px; } .chapter { padding: 40px 0; } }
    @media (prefers-reduced-motion: reduce) { html { scroll-behavior: auto; } *, *::before, *::after { animation-duration: .01ms !important; transition-duration: .01ms !important; } }
  </style>
</head>
<body>
  <main class="book">
    <section class="chapter" id="question" data-zhijian-chapter>
      <p class="chapter-label">01 · 从问题开始</p>
      <h2>用一个具体问题建立学习动机</h2>
      <p class="lead">描述读者会遇到的现象、矛盾或任务，不重复平台上的作品标题和简介。</p>
      <div class="insight"><strong>阅读提示：</strong>告诉读者本章要观察的关系。</div>
    </section>
    <section class="chapter" id="mechanism" data-zhijian-chapter>
      <p class="chapter-label">02 · 理解机制</p>
      <h2>把抽象关系变成可以操作的模型</h2>
      <p>先解释变量和因果关系，再让读者操作。不要用动画代替解释。</p>
      <h3 id="demoTitle">观察条件如何改变结果</h3>
      <div class="demo" aria-labelledby="demoTitle">
        <button id="demoButton" type="button" aria-describedby="demoResult">改变条件</button>
      </div>
      <p id="demoResult" class="insight" aria-live="polite">当前结果会在这里说明变化及其含义。</p>
    </section>
    <section class="chapter" id="practice" data-zhijian-chapter>
      <p class="chapter-label">03 · 迁移与实践</p>
      <h2>把机制用于新的情境</h2>
      <p>提供一个练习、判断或现实案例，并给出可以自查的结论。</p>
    </section>
  </main>
  <script>
    document.querySelector('#demoButton').addEventListener('click', () => {
      document.querySelector('#demoResult').textContent = '条件已改变：在这里解释结果为什么变化，以及它验证了什么。';
    });
  </script>
</body>
</html>
```

## 合集规划模板

合集不是一个超长 HTML，而是一个可直接上传的根目录。制作前先用下表锁定跨作品结构：

| 目录键 | 作品标题 | 类型 | 独立学习目标 | 核心操作/阅读任务 | 所属章节 |
|---|---|---|---|---|---|
| `01-topic` | 主题一 | 交互演示 | … | … | 1 |
| `02-topic` | 主题二 | 动态书 | … | … | 2.1 |

生成顺序固定为：确认合集学习路径 → 确认章节树和目录键 → 分别制作并验证各作品 → 制作合集介绍与合集封面 → 生成 `collection.json` → 检查每个目录键恰好被引用一次。合集中的作品可以混用交互演示和动态书，但每个作品自身只能有一个类型。
