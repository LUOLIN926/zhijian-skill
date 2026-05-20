# 知见作品模板

以下模板均为完整可运行的 HTML 文件，可直接复制使用。起始模板只保留三类：交互演示、互动图片视频、流程图。所有模板都内置知见 CSS 变量，并保持大地色调、衬线字体、克制圆角和轻阴影。

> 标题、简介、标签等元数据应放在 `metadata/work.json` 中，HTML 专注展示和交互。

---

## 交互演示

适合 Canvas/SVG 动画、数据可视化、概念模拟等交互作品。包含演示画布、状态说明和开始/暂停/重置控制。

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
      --transition-slow: 0.5s ease-out;
      --ease-out-quart: cubic-bezier(0.25, 1, 0.5, 1);
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    body {
      min-height: 100vh;
      font-family: var(--font-family);
      font-weight: var(--font-weight-regular);
      color: var(--color-text);
      background:
        radial-gradient(circle at 16% 10%, rgba(196, 167, 125, 0.2), transparent 28rem),
        linear-gradient(180deg, var(--color-cream) 0%, var(--color-cream-dark) 100%);
      padding: var(--spacing-lg);
    }

    button {
      font: inherit;
    }

    .demo-shell {
      width: min(1080px, 100%);
      margin: 0 auto;
      display: grid;
      gap: var(--spacing-lg);
    }

    .demo-layout {
      display: grid;
      grid-template-columns: minmax(0, 1fr) 280px;
      gap: var(--spacing-lg);
      align-items: stretch;
    }

    .stage-card,
    .info-card {
      background: white;
      border: 1px solid var(--color-cream-dark);
      border-radius: var(--radius-lg);
      box-shadow: var(--shadow-md);
    }

    .stage-card {
      overflow: hidden;
    }

    .canvas-wrap {
      width: 100%;
      aspect-ratio: 16 / 10;
      min-height: 360px;
      background:
        linear-gradient(135deg, rgba(74, 103, 65, 0.08), transparent 45%),
        var(--color-cream);
      position: relative;
    }

    #demoCanvas {
      width: 100%;
      height: 100%;
      display: block;
    }

    .stage-caption {
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: var(--spacing-md);
      padding: var(--spacing-md);
      border-top: 1px solid var(--color-cream-dark);
    }

    .metric {
      display: grid;
      gap: 2px;
    }

    .metric span {
      color: var(--color-text-muted);
      font-size: 0.78rem;
      font-weight: var(--font-weight-semibold);
    }

    .metric strong {
      color: var(--color-primary-dark);
      font-size: 1.2rem;
    }

    .info-card {
      padding: var(--spacing-lg);
      display: grid;
      align-content: start;
      gap: var(--spacing-md);
    }

    .info-card h2 {
      color: var(--color-primary-dark);
      font-size: 1.25rem;
    }

    .info-card p {
      color: var(--color-text-light);
      line-height: 1.7;
    }

    .controls {
      display: grid;
      gap: var(--spacing-sm);
    }

    .btn {
      display: inline-flex;
      align-items: center;
      justify-content: center;
      min-height: 42px;
      padding: 0 16px;
      border: 1px solid transparent;
      border-radius: var(--radius-sm);
      cursor: pointer;
      font-weight: var(--font-weight-bold);
      transition: transform var(--transition-fast), background var(--transition-fast), border-color var(--transition-fast), box-shadow var(--transition-fast);
    }

    .btn:hover {
      transform: translateY(-1px);
      box-shadow: var(--shadow-sm);
    }

    .btn-primary {
      background: var(--color-primary);
      color: white;
    }

    .btn-primary:hover {
      background: var(--color-primary-dark);
    }

    .btn-secondary {
      background: var(--color-cream);
      border-color: var(--color-earth);
      color: var(--color-text);
    }

    .btn-secondary:hover {
      background: var(--color-cream-dark);
    }

    @media (max-width: 820px) {
      body {
        padding: var(--spacing-md);
      }

      .demo-layout {
        grid-template-columns: 1fr;
      }

      .canvas-wrap {
        min-height: 260px;
      }
    }
  </style>
</head>
<body>
  <main class="demo-shell">
    <section class="demo-layout" aria-label="交互演示区域">
      <article class="stage-card">
        <div class="canvas-wrap">
          <canvas id="demoCanvas"></canvas>
        </div>
        <div class="stage-caption">
          <div class="metric">
            <span>当前状态</span>
            <strong id="statusText">待开始</strong>
          </div>
          <div class="metric">
            <span>响应强度</span>
            <strong id="energyText">0%</strong>
          </div>
        </div>
      </article>

      <aside class="info-card">
        <h2>演示说明</h2>
        <p>这个区域适合放置参数含义、观察提示或实验结论。保持文字简短，让用户把注意力留在演示本身。</p>
        <div class="controls">
          <button class="btn btn-primary" id="startBtn" type="button">开始</button>
          <button class="btn btn-secondary" id="pauseBtn" type="button">暂停</button>
          <button class="btn btn-secondary" id="resetBtn" type="button">重置</button>
        </div>
      </aside>
    </section>
  </main>

  <script>
    const canvas = document.getElementById('demoCanvas');
    const ctx = canvas.getContext('2d');
    const statusText = document.getElementById('statusText');
    const energyText = document.getElementById('energyText');
    const points = [];
    let running = false;
    let pointer = { x: 0, y: 0, active: false };
    let frameId = null;

    function fitCanvas() {
      const rect = canvas.getBoundingClientRect();
      const dpr = window.devicePixelRatio || 1;
      canvas.width = Math.round(rect.width * dpr);
      canvas.height = Math.round(rect.height * dpr);
      ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
      seedPoints();
      draw();
    }

    function seedPoints() {
      points.length = 0;
      const rect = canvas.getBoundingClientRect();
      for (let i = 0; i < 54; i++) {
        points.push({
          x: Math.random() * rect.width,
          y: Math.random() * rect.height,
          vx: (Math.random() - 0.5) * 0.7,
          vy: (Math.random() - 0.5) * 0.7,
          r: 3 + Math.random() * 4
        });
      }
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
        point.x = Math.max(point.r, Math.min(rect.width - point.r, point.x));
        point.y = Math.max(point.r, Math.min(rect.height - point.r, point.y));
      });
      energyText.textContent = Math.min(100, Math.round(energy * 8)) + '%';
    }

    function draw() {
      const rect = canvas.getBoundingClientRect();
      ctx.clearRect(0, 0, rect.width, rect.height);
      ctx.fillStyle = '#FAF8F5';
      ctx.fillRect(0, 0, rect.width, rect.height);

      points.forEach((a, index) => {
        points.slice(index + 1).forEach((b) => {
          const distance = Math.hypot(a.x - b.x, a.y - b.y);
          if (distance < 110) {
            ctx.strokeStyle = `rgba(139, 115, 85, ${1 - distance / 110})`;
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.moveTo(a.x, a.y);
            ctx.lineTo(b.x, b.y);
            ctx.stroke();
          }
        });

        ctx.fillStyle = '#4A6741';
        ctx.beginPath();
        ctx.arc(a.x, a.y, a.r, 0, Math.PI * 2);
        ctx.fill();
      });
    }

    function loop() {
      if (!running) return;
      update();
      draw();
      frameId = requestAnimationFrame(loop);
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
      cancelAnimationFrame(frameId);
    });

    document.getElementById('resetBtn').addEventListener('click', () => {
      running = false;
      statusText.textContent = '待开始';
      energyText.textContent = '0%';
      cancelAnimationFrame(frameId);
      seedPoints();
      draw();
    });

    canvas.addEventListener('pointermove', (event) => {
      const rect = canvas.getBoundingClientRect();
      pointer = {
        x: event.clientX - rect.left,
        y: event.clientY - rect.top,
        active: true
      };
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

## 互动图片视频

适合图片解读、视频讲解、作品拆解、地图或场景导览。包含媒体展示区、章节切换、热点标记、说明面板和播放/切换/重置控件。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>互动图片视频</title>
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
      --transition-slow: 0.5s ease-out;
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
      background: linear-gradient(180deg, var(--color-cream) 0%, var(--color-cream-dark) 100%);
    }

    button {
      font: inherit;
    }

    .media-shell {
      width: min(1120px, 100%);
      margin: 0 auto;
      display: grid;
      gap: var(--spacing-lg);
    }

    .media-grid {
      display: grid;
      grid-template-columns: minmax(0, 1fr) 320px;
      gap: var(--spacing-lg);
      align-items: start;
    }

    .viewer {
      background: white;
      border: 1px solid var(--color-cream-dark);
      border-radius: var(--radius-lg);
      box-shadow: var(--shadow-md);
      overflow: hidden;
    }

    .media-frame {
      position: relative;
      aspect-ratio: 16 / 9;
      background: var(--color-cream);
      overflow: hidden;
    }

    .media-frame img,
    .media-frame video {
      width: 100%;
      height: 100%;
      display: block;
      object-fit: cover;
    }

    .media-frame video {
      display: none;
      background: var(--color-text);
    }

    .media-frame.video-mode img {
      display: none;
    }

    .media-frame.video-mode video {
      display: block;
    }

    .hotspot {
      position: absolute;
      width: 34px;
      height: 34px;
      border: 2px solid white;
      border-radius: var(--radius-full);
      background: var(--color-secondary);
      color: white;
      cursor: pointer;
      font-weight: var(--font-weight-bold);
      box-shadow: var(--shadow-md);
      transition: transform var(--transition-fast), background var(--transition-fast);
    }

    .hotspot:hover,
    .hotspot.active {
      background: var(--color-primary);
      transform: translateY(-2px) scale(1.04);
    }

    .hotspot[data-index="0"] {
      left: 22%;
      top: 32%;
    }

    .hotspot[data-index="1"] {
      left: 58%;
      top: 44%;
    }

    .hotspot[data-index="2"] {
      left: 74%;
      top: 22%;
    }

    .viewer-toolbar {
      display: flex;
      flex-wrap: wrap;
      gap: var(--spacing-sm);
      padding: var(--spacing-md);
      border-top: 1px solid var(--color-cream-dark);
    }

    .btn {
      display: inline-flex;
      align-items: center;
      justify-content: center;
      min-height: 40px;
      padding: 0 14px;
      border: 1px solid transparent;
      border-radius: var(--radius-sm);
      cursor: pointer;
      font-weight: var(--font-weight-bold);
      transition: transform var(--transition-fast), background var(--transition-fast), border-color var(--transition-fast), box-shadow var(--transition-fast);
    }

    .btn:hover {
      transform: translateY(-1px);
      box-shadow: var(--shadow-sm);
    }

    .btn-primary {
      background: var(--color-primary);
      color: white;
    }

    .btn-primary:hover {
      background: var(--color-primary-dark);
    }

    .btn-secondary {
      background: var(--color-cream);
      border-color: var(--color-earth);
      color: var(--color-text);
    }

    .btn-secondary:hover {
      background: var(--color-cream-dark);
    }

    .side-panel {
      display: grid;
      gap: var(--spacing-md);
    }

    .chapter-list,
    .detail-panel {
      padding: var(--spacing-lg);
      background: white;
      border: 1px solid var(--color-cream-dark);
      border-radius: var(--radius-lg);
      box-shadow: var(--shadow-sm);
    }

    .chapter-list h2,
    .detail-panel h2 {
      margin-bottom: var(--spacing-md);
      color: var(--color-primary-dark);
      font-size: 1.18rem;
    }

    .chapter-button {
      width: 100%;
      display: grid;
      gap: 4px;
      margin-bottom: var(--spacing-sm);
      padding: 12px;
      border: 1px solid var(--color-cream-dark);
      border-radius: var(--radius-sm);
      background: var(--color-cream);
      color: var(--color-text);
      cursor: pointer;
      text-align: left;
      transition: border-color var(--transition-fast), background var(--transition-fast), transform var(--transition-fast);
    }

    .chapter-button:hover,
    .chapter-button.active {
      border-color: var(--color-primary);
      background: color-mix(in oklch, var(--color-primary) 8%, white);
      transform: translateY(-1px);
    }

    .chapter-button strong {
      color: var(--color-primary-dark);
    }

    .chapter-button span {
      color: var(--color-text-muted);
      font-size: 0.86rem;
    }

    .detail-panel p {
      color: var(--color-text-light);
      line-height: 1.75;
    }

    @media (max-width: 860px) {
      body {
        padding: var(--spacing-md);
      }

      .media-grid {
        grid-template-columns: 1fr;
      }

      .viewer-toolbar {
        display: grid;
      }
    }
  </style>
</head>
<body>
  <main class="media-shell">
    <section class="media-grid" aria-label="互动媒体讲解">
      <article class="viewer">
        <div class="media-frame" id="mediaFrame">
          <img id="mainImage" src="https://images.unsplash.com/photo-1518005020951-eccb494ad742?auto=format&fit=crop&w=1400&q=80" alt="示例讲解图片">
          <video id="mainVideo" controls muted playsinline>
            <source src="your-video.mp4" type="video/mp4">
          </video>
          <button class="hotspot active" type="button" data-index="0" aria-label="查看热点一">1</button>
          <button class="hotspot" type="button" data-index="1" aria-label="查看热点二">2</button>
          <button class="hotspot" type="button" data-index="2" aria-label="查看热点三">3</button>
        </div>
        <div class="viewer-toolbar">
          <button class="btn btn-primary" id="toggleMediaBtn" type="button">切换视频</button>
          <button class="btn btn-secondary" id="playBtn" type="button">播放/暂停</button>
          <button class="btn btn-secondary" id="resetBtn" type="button">重置</button>
        </div>
      </article>

      <aside class="side-panel">
        <section class="chapter-list" aria-label="章节列表">
          <h2>讲解章节</h2>
          <button class="chapter-button active" type="button" data-chapter="0">
            <strong>整体观察</strong>
            <span>先建立全局印象</span>
          </button>
          <button class="chapter-button" type="button" data-chapter="1">
            <strong>局部拆解</strong>
            <span>聚焦关键节点</span>
          </button>
          <button class="chapter-button" type="button" data-chapter="2">
            <strong>结论回看</strong>
            <span>把细节收束成观点</span>
          </button>
        </section>

        <section class="detail-panel" aria-live="polite">
          <h2 id="detailTitle">整体观察</h2>
          <p id="detailText">从全图开始介绍主题、对象和观察方法。这里适合放 2 到 3 句说明，让用户知道应该看哪里。</p>
        </section>
      </aside>
    </section>
  </main>

  <script>
    const details = [
      {
        title: '整体观察',
        text: '从全图开始介绍主题、对象和观察方法。这里适合放 2 到 3 句说明，让用户知道应该看哪里。'
      },
      {
        title: '局部拆解',
        text: '解释被标记区域的结构、关系或变化。可以把专业概念转换成可观察的画面线索。'
      },
      {
        title: '结论回看',
        text: '回到核心问题，提示用户把刚才看到的细节和最终结论连接起来。'
      }
    ];

    const mediaFrame = document.getElementById('mediaFrame');
    const mainVideo = document.getElementById('mainVideo');
    const detailTitle = document.getElementById('detailTitle');
    const detailText = document.getElementById('detailText');
    const toggleMediaBtn = document.getElementById('toggleMediaBtn');

    function setDetail(index) {
      const detail = details[index];
      detailTitle.textContent = detail.title;
      detailText.textContent = detail.text;
      document.querySelectorAll('.hotspot').forEach((button) => {
        button.classList.toggle('active', Number(button.dataset.index) === index);
      });
      document.querySelectorAll('.chapter-button').forEach((button) => {
        button.classList.toggle('active', Number(button.dataset.chapter) === index);
      });
    }

    document.querySelectorAll('.hotspot').forEach((button) => {
      button.addEventListener('click', () => setDetail(Number(button.dataset.index)));
    });

    document.querySelectorAll('.chapter-button').forEach((button) => {
      button.addEventListener('click', () => setDetail(Number(button.dataset.chapter)));
    });

    toggleMediaBtn.addEventListener('click', () => {
      const videoMode = mediaFrame.classList.toggle('video-mode');
      toggleMediaBtn.textContent = videoMode ? '切换图片' : '切换视频';
      if (!videoMode) mainVideo.pause();
    });

    document.getElementById('playBtn').addEventListener('click', () => {
      mediaFrame.classList.add('video-mode');
      toggleMediaBtn.textContent = '切换图片';
      if (mainVideo.paused) {
        mainVideo.play().catch(() => {});
      } else {
        mainVideo.pause();
      }
    });

    document.getElementById('resetBtn').addEventListener('click', () => {
      mediaFrame.classList.remove('video-mode');
      toggleMediaBtn.textContent = '切换视频';
      mainVideo.pause();
      mainVideo.currentTime = 0;
      setDetail(0);
    });
  </script>
</body>
</html>
```

---

## 流程图

适合步骤引导、操作清单、学习路径、实验流程。结构参考 `https://tkgptplus.bigbiscuit.fun/#gift-card` 的组织形式：总体进度、阶段导航、阶段区块、任务卡片、完成状态、风险提示和行动按钮。视觉仍使用知见配色、字体、圆角、阴影和控件样式。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>流程图</title>
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
      --transition-slow: 0.5s ease-out;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    html {
      scroll-behavior: smooth;
    }

    body {
      min-height: 100vh;
      padding: var(--spacing-lg);
      font-family: var(--font-family);
      color: var(--color-text);
      background:
        linear-gradient(180deg, rgba(196, 167, 125, 0.22) 0, transparent 340px),
        var(--color-cream);
    }

    button,
    a {
      font: inherit;
    }

    .flow-shell {
      width: min(1180px, 100%);
      margin: 0 auto;
      display: grid;
      gap: var(--spacing-lg);
    }

    .flow-progress,
    .flow-guide,
    .flow-stage,
    .flow-card,
    .flow-warning {
      background: white;
      border: 1px solid var(--color-cream-dark);
      border-radius: var(--radius-lg);
      box-shadow: var(--shadow-sm);
    }

    .stage-label,
    .risk-badge {
      display: inline-flex;
      width: fit-content;
      align-items: center;
      min-height: 28px;
      padding: 0 10px;
      border-radius: var(--radius-full);
      font-size: 0.8rem;
      font-weight: var(--font-weight-bold);
    }

    .stage-label {
      background: color-mix(in oklch, var(--color-primary) 12%, white);
      color: var(--color-primary-dark);
    }

    .flow-btn {
      display: inline-flex;
      align-items: center;
      justify-content: center;
      min-height: 40px;
      padding: 0 14px;
      border: 1px solid transparent;
      border-radius: var(--radius-sm);
      cursor: pointer;
      font-weight: var(--font-weight-bold);
      text-decoration: none;
      transition: transform var(--transition-fast), background var(--transition-fast), border-color var(--transition-fast), box-shadow var(--transition-fast);
    }

    .flow-btn:hover {
      transform: translateY(-1px);
      box-shadow: var(--shadow-sm);
    }

    .flow-btn.primary {
      background: var(--color-primary);
      color: white;
    }

    .flow-btn.primary:hover {
      background: var(--color-primary-dark);
    }

    .flow-btn.secondary {
      background: var(--color-cream);
      border-color: var(--color-earth);
      color: var(--color-text);
    }

    .flow-btn.secondary:hover {
      background: var(--color-cream-dark);
    }

    .flow-progress {
      display: grid;
      grid-template-columns: minmax(86px, max-content) minmax(0, 1fr) max-content;
      gap: var(--spacing-md);
      align-items: center;
      padding: var(--spacing-lg);
    }

    .progress-number {
      color: var(--color-primary-dark);
      font-size: clamp(2.4rem, 6vw, 3.4rem);
      font-weight: var(--font-weight-bold);
      line-height: 1;
    }

    .progress-copy {
      display: grid;
      gap: var(--spacing-xs);
    }

    .progress-copy strong {
      color: var(--color-text);
    }

    .progress-copy span {
      color: var(--color-text-muted);
      line-height: 1.55;
    }

    .progress-track {
      grid-column: 2 / 4;
      height: 12px;
      overflow: hidden;
      border-radius: var(--radius-full);
      background: var(--color-cream-dark);
    }

    .progress-track span {
      display: block;
      width: 0%;
      height: 100%;
      border-radius: inherit;
      background: linear-gradient(90deg, var(--color-primary), var(--color-secondary));
      transition: width var(--transition-normal);
    }

    .flow-warning {
      display: grid;
      grid-template-columns: 160px minmax(0, 1fr);
      gap: var(--spacing-md);
      padding: var(--spacing-lg);
      background: color-mix(in oklch, var(--color-warning) 10%, white);
      border-color: color-mix(in oklch, var(--color-warning) 28%, white);
    }

    .flow-warning strong {
      color: color-mix(in oklch, var(--color-warning) 58%, var(--color-text));
    }

    .flow-warning ul {
      display: grid;
      gap: var(--spacing-xs);
      padding-left: var(--spacing-lg);
      color: var(--color-text-light);
      line-height: 1.65;
    }

    .flow-layout {
      display: grid;
      grid-template-columns: 280px minmax(0, 1fr);
      gap: var(--spacing-lg);
      align-items: start;
    }

    .flow-guide {
      position: sticky;
      top: var(--spacing-md);
      display: grid;
      gap: var(--spacing-sm);
      padding: var(--spacing-sm);
    }

    .guide-link {
      display: grid;
      grid-template-columns: minmax(0, 1fr) auto;
      gap: var(--spacing-sm);
      align-items: center;
      padding: 12px;
      border: 1px solid transparent;
      border-radius: var(--radius-sm);
      color: var(--color-text);
      text-decoration: none;
      transition: background var(--transition-fast), border-color var(--transition-fast);
    }

    .guide-link:hover,
    .guide-link.active {
      background: var(--color-cream);
      border-color: var(--color-earth);
    }

    .guide-link strong {
      display: block;
      color: var(--color-primary-dark);
      font-size: 0.96rem;
    }

    .guide-link small {
      display: block;
      margin-top: 3px;
      color: var(--color-text-muted);
      line-height: 1.45;
    }

    .guide-link em {
      display: inline-flex;
      min-width: 42px;
      min-height: 28px;
      align-items: center;
      justify-content: center;
      border-radius: var(--radius-full);
      background: var(--color-cream-dark);
      color: var(--color-text-light);
      font-style: normal;
      font-weight: var(--font-weight-bold);
      font-size: 0.78rem;
    }

    .flow-stack {
      display: grid;
      gap: var(--spacing-lg);
    }

    .flow-stage {
      display: grid;
      gap: var(--spacing-md);
      padding: var(--spacing-lg);
      scroll-margin-top: var(--spacing-lg);
    }

    .stage-head {
      display: flex;
      justify-content: space-between;
      gap: var(--spacing-md);
      align-items: flex-start;
    }

    .stage-head h2 {
      margin-top: var(--spacing-sm);
      color: var(--color-primary-dark);
      font-size: clamp(1.4rem, 3vw, 2rem);
    }

    .stage-head p {
      margin-top: var(--spacing-xs);
      color: var(--color-text-light);
      line-height: 1.65;
    }

    .risk-badge.low {
      background: color-mix(in oklch, var(--color-success) 14%, white);
      color: var(--color-success);
    }

    .risk-badge.medium {
      background: color-mix(in oklch, var(--color-warning) 15%, white);
      color: color-mix(in oklch, var(--color-warning) 70%, var(--color-text));
    }

    .risk-badge.high {
      background: color-mix(in oklch, var(--color-danger) 12%, white);
      color: var(--color-danger);
    }

    .flow-card-list {
      display: grid;
      gap: var(--spacing-sm);
    }

    .flow-card {
      display: grid;
      grid-template-columns: 132px minmax(0, 1fr);
      gap: var(--spacing-md);
      padding: var(--spacing-md);
      transition: background var(--transition-fast), border-color var(--transition-fast), box-shadow var(--transition-fast);
    }

    .flow-card.done {
      background: color-mix(in oklch, var(--color-success) 7%, white);
      border-color: color-mix(in oklch, var(--color-success) 38%, white);
      box-shadow: var(--shadow-md);
    }

    .task-toggle {
      align-self: start;
      display: inline-flex;
      align-items: center;
      justify-content: center;
      min-height: 40px;
      border: 1px solid var(--color-earth);
      border-radius: var(--radius-sm);
      background: var(--color-cream);
      color: var(--color-text);
      cursor: pointer;
      font-weight: var(--font-weight-bold);
      transition: background var(--transition-fast), border-color var(--transition-fast), transform var(--transition-fast);
    }

    .task-toggle:hover {
      background: var(--color-cream-dark);
      transform: translateY(-1px);
    }

    .flow-card.done .task-toggle {
      background: color-mix(in oklch, var(--color-success) 12%, white);
      border-color: var(--color-success);
      color: var(--color-success);
    }

    .task-body {
      display: grid;
      gap: var(--spacing-sm);
    }

    .task-body h3 {
      color: var(--color-text);
      font-size: 1.08rem;
    }

    .task-body p,
    .task-body li {
      color: var(--color-text-light);
      line-height: 1.65;
    }

    .task-body ul {
      display: grid;
      gap: var(--spacing-xs);
      padding-left: var(--spacing-lg);
    }

    @media (max-width: 900px) {
      .flow-layout {
        grid-template-columns: 1fr;
      }

      .flow-guide {
        position: static;
        grid-template-columns: repeat(2, minmax(0, 1fr));
      }
    }

    @media (max-width: 680px) {
      body {
        padding: var(--spacing-md);
      }

      .flow-stage,
      .flow-progress,
      .flow-warning {
        padding: var(--spacing-md);
      }

      .flow-progress,
      .flow-warning,
      .flow-card {
        grid-template-columns: 1fr;
      }

      .progress-track {
        grid-column: auto;
      }

      .flow-guide {
        grid-template-columns: 1fr;
      }

      .stage-head {
        display: grid;
      }
    }
  </style>
</head>
<body>
  <main class="flow-shell">
    <section class="flow-progress" aria-label="总体进度">
      <strong class="progress-number" id="progressNumber">0%</strong>
      <div class="progress-copy">
        <strong id="progressCount">0 / 6 个任务完成</strong>
        <span id="progressHint">进度只保存在当前页面状态，刷新后会重置。</span>
      </div>
      <a class="flow-btn secondary" href="#stage-review">查看收尾</a>
      <div class="progress-track" aria-hidden="true">
        <span id="progressFill"></span>
      </div>
    </section>

    <section class="flow-warning" aria-label="重要提示">
      <strong>先确认这些</strong>
      <ul>
        <li>把风险、前置条件或准备事项放在这里。</li>
        <li>避免一次给太多说明，每条只写用户最需要记住的判断。</li>
      </ul>
    </section>

    <div class="flow-layout">
      <nav class="flow-guide" aria-label="阶段导航">
        <a class="guide-link active" href="#stage-prepare" data-stage-link="stage-prepare">
          <span>
            <strong>准备阶段</strong>
            <small>明确目标与材料</small>
          </span>
          <em id="count-prepare">0/2</em>
        </a>
        <a class="guide-link" href="#stage-action" data-stage-link="stage-action">
          <span>
            <strong>执行阶段</strong>
            <small>按顺序完成核心任务</small>
          </span>
          <em id="count-action">0/2</em>
        </a>
        <a class="guide-link" href="#stage-review" data-stage-link="stage-review">
          <span>
            <strong>复盘阶段</strong>
            <small>检查结果并记录经验</small>
          </span>
          <em id="count-review">0/2</em>
        </a>
      </nav>

      <div class="flow-stack">
        <section class="flow-stage" id="stage-prepare" data-stage="prepare">
          <div class="stage-head">
            <div>
              <span class="stage-label">阶段 1</span>
              <h2>准备阶段</h2>
              <p>先把目标、材料和判断标准整理清楚，降低后续操作成本。</p>
            </div>
            <span class="risk-badge low">低风险</span>
          </div>

          <div class="flow-card-list">
            <article class="flow-card" data-task data-stage-key="prepare">
              <button class="task-toggle" type="button" aria-pressed="false">标记完成</button>
              <div class="task-body">
                <h3>确认目标</h3>
                <p>用一句话写下本流程最终要得到什么结果。</p>
                <ul>
                  <li>目标可被检查。</li>
                  <li>用户知道完成后的下一步。</li>
                </ul>
              </div>
            </article>

            <article class="flow-card" data-task data-stage-key="prepare">
              <button class="task-toggle" type="button" aria-pressed="false">标记完成</button>
              <div class="task-body">
                <h3>准备材料</h3>
                <p>列出需要提前获得的信息、文件、工具或账号。</p>
                <ul>
                  <li>缺少材料时给出替代路径。</li>
                  <li>敏感信息只提示用户自行保管。</li>
                </ul>
              </div>
            </article>
          </div>
        </section>

        <section class="flow-stage" id="stage-action" data-stage="action">
          <div class="stage-head">
            <div>
              <span class="stage-label">阶段 2</span>
              <h2>执行阶段</h2>
              <p>把核心动作拆成少量清晰任务，用户每完成一项就能看到进度变化。</p>
            </div>
            <span class="risk-badge medium">谨慎操作</span>
          </div>

          <div class="flow-card-list">
            <article class="flow-card" data-task data-stage-key="action">
              <button class="task-toggle" type="button" aria-pressed="false">标记完成</button>
              <div class="task-body">
                <h3>完成关键动作</h3>
                <p>描述最重要的一步，并把容易误解的地方写成检查项。</p>
                <ul>
                  <li>按页面实际提示操作。</li>
                  <li>遇到异常时先停止重复尝试。</li>
                </ul>
              </div>
            </article>

            <article class="flow-card" data-task data-stage-key="action">
              <button class="task-toggle" type="button" aria-pressed="false">标记完成</button>
              <div class="task-body">
                <h3>保存结果</h3>
                <p>记录关键结果、截图或确认信息，方便后续复查。</p>
                <ul>
                  <li>只记录必要信息。</li>
                  <li>不要把密码、验证码或密钥写进页面。</li>
                </ul>
              </div>
            </article>
          </div>
        </section>

        <section class="flow-stage" id="stage-review" data-stage="review">
          <div class="stage-head">
            <div>
              <span class="stage-label">阶段 3</span>
              <h2>复盘阶段</h2>
              <p>确认任务结果是否符合预期，并留下可复用的经验。</p>
            </div>
            <span class="risk-badge high">重点核对</span>
          </div>

          <div class="flow-card-list">
            <article class="flow-card" data-task data-stage-key="review">
              <button class="task-toggle" type="button" aria-pressed="false">标记完成</button>
              <div class="task-body">
                <h3>核对结果</h3>
                <p>对照最初目标，检查是否完成、是否遗漏、是否需要补救。</p>
                <ul>
                  <li>结果与目标一致。</li>
                  <li>异常情况有处理记录。</li>
                </ul>
              </div>
            </article>

            <article class="flow-card" data-task data-stage-key="review">
              <button class="task-toggle" type="button" aria-pressed="false">标记完成</button>
              <div class="task-body">
                <h3>沉淀经验</h3>
                <p>把本次流程中最有价值的判断和提醒整理给下一位读者。</p>
                <ul>
                  <li>保留有效步骤。</li>
                  <li>删掉不再必要的旁支说明。</li>
                </ul>
              </div>
            </article>
          </div>
        </section>
      </div>
    </div>
  </main>

  <script>
    const tasks = Array.from(document.querySelectorAll('[data-task]'));
    const progressNumber = document.getElementById('progressNumber');
    const progressCount = document.getElementById('progressCount');
    const progressHint = document.getElementById('progressHint');
    const progressFill = document.getElementById('progressFill');
    const stageKeys = ['prepare', 'action', 'review'];

    function updateProgress() {
      const doneTasks = tasks.filter((task) => task.classList.contains('done'));
      const percent = Math.round((doneTasks.length / tasks.length) * 100);
      progressNumber.textContent = percent + '%';
      progressCount.textContent = `${doneTasks.length} / ${tasks.length} 个任务完成`;
      progressFill.style.width = percent + '%';
      progressHint.textContent = doneTasks.length === tasks.length
        ? '全部完成。最后再核对一次重点提示。'
        : `还剩 ${tasks.length - doneTasks.length} 个任务。`;

      stageKeys.forEach((key) => {
        const stageTasks = tasks.filter((task) => task.dataset.stageKey === key);
        const stageDone = stageTasks.filter((task) => task.classList.contains('done'));
        document.getElementById(`count-${key}`).textContent = `${stageDone.length}/${stageTasks.length}`;
      });
    }

    tasks.forEach((task) => {
      const button = task.querySelector('.task-toggle');
      button.addEventListener('click', () => {
        const done = task.classList.toggle('done');
        button.textContent = done ? '已完成' : '标记完成';
        button.setAttribute('aria-pressed', String(done));
        updateProgress();
      });
    });

    document.querySelectorAll('.guide-link').forEach((link) => {
      link.addEventListener('click', () => {
        document.querySelectorAll('.guide-link').forEach((item) => item.classList.remove('active'));
        link.classList.add('active');
      });
    });

    updateProgress();
  </script>
</body>
</html>
```
