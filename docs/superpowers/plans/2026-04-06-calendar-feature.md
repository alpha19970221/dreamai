# 梦境封印录日历视图 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将 `step-archive` 页面从静态卡片列表改为全屏月历，有梦日期显示 ◈ 封印符文，点击日期底部抽屉滑出显示摘要。

**Architecture:** 单文件原型 `prototype/index.html`，CSS 新增日历相关样式，HTML 替换 `step-archive` 内容为日历骨架，JS 末尾新增数据数组与渲染函数。底部抽屉以 `position: absolute` 挂在 `step-archive` 内，月份导航通过重渲染整个格子实现。

**Tech Stack:** Vanilla HTML/CSS/JS，无外部依赖，Google Fonts 已载入（Noto Serif SC + Inter）。

---

## 文件变更范围

- **Modify:** `prototype/index.html`
  - CSS：删除旧 `.archive-list`/`.archive-card` 规则，新增日历全套样式（约 100 行）
  - HTML：替换 `<div id="step-archive">` 内部内容（约 30 行），保留 `step-archived-detail` 不动
  - JS：新增 `dreams` 数组 + 日历函数（约 80 行），更新 3 处 `onclick` 入口

---

## Task 1: 新增 dreams 数据数组

**Files:**
- Modify: `prototype/index.html`（`</script>` 前，约第 1428 行）

- [ ] **Step 1: 在 `</script>` 标签前插入 dreams 数组**

找到文件末尾的 `</script>` 标签，在它之前添加：

```js
        // --- 梦境封印录数据 ---
        const dreams = [
            {
                id: 1,
                date: '2026-04-04',
                title: '高维的机器与光洞',
                tags: ['光球', '新生', '高维游戏'],
                preview: '我站在一个巨大的机械装置前，光洞从中心向外延伸，周围的空间开始扭曲…'
            },
            {
                id: 2,
                date: '2026-03-28',
                title: '深海无止境的下坠',
                tags: ['失重感', '水', '深海'],
                preview: '深不见底的海水将我包裹，身体不受控制地向下坠落，耳边只有流水的轰鸣…'
            },
            {
                id: 3,
                date: '2026-03-15',
                title: '在长廊中被追逐',
                tags: ['奔跑', '迷宫', '焦虑'],
                preview: '长廊的尽头永远看不见，脚步声在身后越来越近，转角处永远是另一段长廊…'
            }
        ];
```

- [ ] **Step 2: 在浏览器控制台验证**

```
python3 -m http.server 8080
```

打开 `http://localhost:8080/prototype/index.html`，在控制台执行：

```js
dreams[0].title
// 期望输出: "高维的机器与光洞"
```

- [ ] **Step 3: 提交**

```bash
git add prototype/index.html
git commit -m "feat: add dreams data array for calendar feature"
```

---

## Task 2: 删除旧 Archive CSS，新增日历 CSS

**Files:**
- Modify: `prototype/index.html`（`<style>` 块内，约第 456–466 行）

- [ ] **Step 1: 删除旧 `.archive-list` 和 `.archive-card` 规则**

找到如下 CSS 块（约 456–466 行），完整删除这 3 条规则（保留 `.archive-btn`、`.archive-text`、`.nav-back`、`.archive-date`、`.archive-tags`）：

```css
        .archive-list { display: flex; flex-direction: column; gap: 16px; padding-bottom: 30px; margin-top: 20px;}
        .archive-card {
            background: linear-gradient(145deg, rgba(30, 30, 45, 0.4), rgba(15, 15, 25, 0.6));
            border: 1px solid rgba(255,255,255,0.05); border-radius: 20px; padding: 20px;
            cursor: pointer; transition: transform 0.3s ease, border-color 0.3s ease;
        }
        .archive-card:hover { transform: translateY(-3px); border-color: rgba(139, 92, 246, 0.4); box-shadow: 0 10px 20px rgba(0,0,0,0.5); }
        .archive-card-title { font-family: 'Noto Serif SC', serif; font-size: 1.1rem; color: #fff; margin-bottom: 12px; }
```

- [ ] **Step 2: 在删除位置插入日历 CSS**

在删掉上述内容的位置（`.nav-back` 规则之后、`.detail-section-title` 之前）插入：

```css
        /* --- 日历视图 (Calendar) --- */
        #step-archive { padding: 50px 0 0; overflow: hidden !important; }

        .cal-header {
            display: flex; align-items: center; gap: 8px;
            padding: 0 20px 12px; border-bottom: 1px solid rgba(255,255,255,0.05);
        }
        .cal-header-title {
            font-family: 'Noto Serif SC', serif; font-size: 1.1rem; color: #fff;
        }
        .cal-dream-count {
            margin-left: auto; font-size: 0.72rem; color: rgba(139,92,246,0.7);
        }

        .cal-month-nav {
            display: flex; justify-content: space-between; align-items: center;
            padding: 14px 28px 8px;
        }
        .cal-nav-btn {
            font-size: 1.3rem; color: rgba(255,255,255,0.3); cursor: pointer;
            padding: 4px 10px; transition: color 0.2s;
        }
        .cal-nav-btn:hover { color: #a78bfa; }
        .cal-month-title {
            font-family: 'Noto Serif SC', serif; font-size: 1rem; color: #e2d9f3;
        }

        .cal-weekdays {
            display: grid; grid-template-columns: repeat(7, 1fr);
            padding: 0 12px; margin-bottom: 4px;
        }
        .cal-weekday {
            text-align: center; font-size: 0.68rem; color: rgba(255,255,255,0.2); padding: 4px 0;
        }

        .cal-grid {
            display: grid; grid-template-columns: repeat(7, 1fr);
            gap: 4px; padding: 0 12px;
        }
        .cal-day {
            aspect-ratio: 1; display: flex; align-items: center; justify-content: center;
            font-size: 0.8rem; color: rgba(255,255,255,0.3); border-radius: 50%;
            cursor: default; position: relative; transition: background 0.2s;
            font-family: 'Inter', sans-serif;
        }
        .cal-day.today {
            color: rgba(255,255,255,0.75);
            border: 1px solid rgba(139,92,246,0.4);
        }
        .cal-day.has-dream {
            color: #e2d9f3; cursor: pointer;
        }
        .cal-day.has-dream::before {
            content: '◈'; position: absolute;
            font-size: 1.8rem; color: rgba(139,92,246,0.22);
            top: 50%; left: 50%; transform: translate(-50%, -50%);
            pointer-events: none; line-height: 1;
        }
        .cal-day.has-dream:hover { background: rgba(139,92,246,0.12); }
        .cal-day.selected {
            background: rgba(139,92,246,0.45) !important;
            color: #fff !important;
            box-shadow: 0 0 12px rgba(139,92,246,0.4);
        }
        .cal-day.selected::before { display: none; }

        .cal-count-bar {
            padding: 8px 16px 0; font-size: 0.72rem;
            color: rgba(139,92,246,0.55); text-align: right;
        }

        .cal-empty-state {
            flex: 1; display: flex; flex-direction: column;
            align-items: center; justify-content: center;
            opacity: 0.45; padding: 0 24px; pointer-events: none;
        }
        .cal-empty-symbol { font-size: 2.5rem; color: rgba(139,92,246,0.4); margin-bottom: 12px; }
        .cal-empty-text {
            font-family: 'Noto Serif SC', serif; font-size: 0.85rem;
            color: rgba(255,255,255,0.4); text-align: center; line-height: 2;
        }

        /* 底部抽屉 */
        .cal-bottom-sheet {
            position: absolute; bottom: 0; left: 0; right: 0;
            background: linear-gradient(160deg, rgba(22,16,42,0.98), rgba(8,8,20,0.99));
            border-top: 1px solid rgba(139,92,246,0.25);
            border-radius: 22px 22px 0 0;
            padding: 10px 20px 36px;
            backdrop-filter: blur(20px);
            box-shadow: 0 -8px 30px rgba(0,0,0,0.5);
            transform: translateY(100%);
            transition: transform 0.35s cubic-bezier(0.16, 1, 0.3, 1);
            z-index: 20;
        }
        .cal-bottom-sheet.open { transform: translateY(0); }
        .cal-bs-handle {
            width: 36px; height: 3px; background: rgba(255,255,255,0.12);
            border-radius: 2px; margin: 0 auto 14px;
        }
        .cal-bs-date {
            font-size: 0.72rem; color: rgba(139,92,246,0.8);
            letter-spacing: 1px; margin-bottom: 10px; text-align: center;
        }
        .cal-bs-card {
            background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.07);
            border-radius: 14px; padding: 14px 16px; margin-bottom: 14px;
        }
        .cal-bs-title {
            font-family: 'Noto Serif SC', serif; font-size: 1.05rem; color: #fff; margin-bottom: 8px;
        }
        .cal-bs-tags { display: flex; gap: 6px; flex-wrap: wrap; margin-bottom: 8px; }
        .cal-bs-tag {
            font-size: 0.72rem; background: rgba(255,255,255,0.07);
            padding: 3px 10px; border-radius: 10px; color: rgba(255,255,255,0.6);
        }
        .cal-bs-preview {
            font-size: 0.8rem; color: rgba(255,255,255,0.38); line-height: 1.7;
        }
        .cal-bs-btn {
            display: block; text-align: center;
            font-family: 'Noto Serif SC', serif; font-size: 0.9rem;
            background: rgba(139,92,246,0.15); border: 1px solid rgba(139,92,246,0.35);
            border-radius: 14px; padding: 12px; color: #a78bfa; cursor: pointer;
            transition: background 0.2s;
        }
        .cal-bs-btn:hover { background: rgba(139,92,246,0.25); }
```

- [ ] **Step 3: 刷新浏览器，确认现有页面无样式错乱**

重新加载 `http://localhost:8080/prototype/index.html`，检查主页、详情页视觉正常（`.archive-date`、`.archive-tags` 仍生效）。

- [ ] **Step 4: 提交**

```bash
git add prototype/index.html
git commit -m "feat: add calendar CSS, remove old archive list styles"
```

---

## Task 3: 替换 step-archive HTML 内容

**Files:**
- Modify: `prototype/index.html`（约第 711–747 行）

- [ ] **Step 1: 找到并替换 `step-archive` div 的全部内部内容**

找到：
```html
        <!-- 1.5 The Archive Page (Dream List) -->
        <div class="view-step" id="step-archive">
            <div class="nav-back" onclick="goToStep('step-archive', 'step-home')">⟵ 返回</div>
            <div class="step-title" style="text-align: left; font-size: 1.8rem;">梦境封印录</div>
            <div class="step-subtitle" style="text-align: left; margin-bottom: 10px;">重新凝视你遗落的潜意识碎片</div>

            <div class="archive-list">
                <div class="archive-card" onclick="goToStep('step-archive', 'step-archived-detail')">
                    <div class="archive-date">2026.04.04</div>
                    <div class="archive-card-title">高维的机器与光洞</div>
                    <div class="archive-tags">
                        <span>#光球</span>
                        <span>#新生</span>
                        <span>#高维游戏</span>
                    </div>
                </div>

                <div class="archive-card">
                    <div class="archive-date">2026.03.28</div>
                    <div class="archive-card-title">深海无止境的下坠</div>
                    <div class="archive-tags">
                        <span>#失重感</span>
                        <span>#水</span>
                        <span>#深海</span>
                    </div>
                </div>

                <div class="archive-card">
                    <div class="archive-date">2026.03.15</div>
                    <div class="archive-card-title">在长廊中被追逐</div>
                    <div class="archive-tags">
                        <span>#奔跑</span>
                        <span>#迷宫</span>
                        <span>#焦虑</span>
                    </div>
                </div>
            </div>
        </div>
```

替换为：

```html
        <!-- 1.5 The Archive Page (Calendar View) -->
        <div class="view-step" id="step-archive">
            <!-- 顶部导航 -->
            <div class="cal-header">
                <span class="nav-back" onclick="goToStep('step-archive', 'step-home')" style="margin-bottom:0;">⟵</span>
                <span class="cal-header-title">梦境封印录</span>
                <span class="cal-dream-count" id="cal-dream-count"></span>
            </div>

            <!-- 月份导航 -->
            <div class="cal-month-nav">
                <span class="cal-nav-btn" onclick="changeMonth(-1)">‹</span>
                <span class="cal-month-title" id="cal-month-title"></span>
                <span class="cal-nav-btn" onclick="changeMonth(1)">›</span>
            </div>

            <!-- 星期标题行 -->
            <div class="cal-weekdays">
                <div class="cal-weekday">日</div>
                <div class="cal-weekday">一</div>
                <div class="cal-weekday">二</div>
                <div class="cal-weekday">三</div>
                <div class="cal-weekday">四</div>
                <div class="cal-weekday">五</div>
                <div class="cal-weekday">六</div>
            </div>

            <!-- 日历格子（JS 动态渲染） -->
            <div class="cal-grid" id="cal-grid"></div>

            <!-- 本月计数 -->
            <div class="cal-count-bar" id="cal-count-bar"></div>

            <!-- 未选中日期时的引导文案 -->
            <div class="cal-empty-state" id="cal-empty-state">
                <div class="cal-empty-symbol">◈</div>
                <div class="cal-empty-text">轻触日历中的日期<br>重新凝视那夜的梦境碎片</div>
            </div>

            <!-- 底部抽屉（点击有梦日期后滑入） -->
            <div class="cal-bottom-sheet" id="cal-bottom-sheet">
                <div class="cal-bs-handle"></div>
                <div class="cal-bs-date" id="cal-bs-date"></div>
                <div class="cal-bs-card">
                    <div class="cal-bs-title" id="cal-bs-title"></div>
                    <div class="cal-bs-tags" id="cal-bs-tags"></div>
                    <div class="cal-bs-preview" id="cal-bs-preview"></div>
                </div>
                <div class="cal-bs-btn" onclick="goToStep('step-archive', 'step-archived-detail')">深入凝视 →</div>
            </div>
        </div>
```

- [ ] **Step 2: 刷新浏览器，点击主页的"梦境封印录"按钮**

期望：进入封印录页面，看到月份导航栏和星期行标题，日历格子为空（JS 还没挂），底部无抽屉。

- [ ] **Step 3: 提交**

```bash
git add prototype/index.html
git commit -m "feat: replace archive list HTML with calendar skeleton"
```

---

## Task 4: 新增日历 JS 函数

**Files:**
- Modify: `prototype/index.html`（`dreams` 数组之后，`</script>` 之前）

- [ ] **Step 1: 在 `dreams` 数组之后追加日历逻辑**

```js
        // --- 日历视图逻辑 ---
        let calState = { year: new Date().getFullYear(), month: new Date().getMonth() };

        function initCalendar() {
            calState = { year: new Date().getFullYear(), month: new Date().getMonth() };
            renderCalendar(calState.year, calState.month);
            hideCalBottomSheet();
        }

        function renderCalendar(year, month) {
            const monthNames = ['1月','2月','3月','4月','5月','6月','7月','8月','9月','10月','11月','12月'];
            document.getElementById('cal-month-title').textContent = year + '年 ' + monthNames[month];

            const monthStr = year + '-' + String(month + 1).padStart(2, '0');
            const monthDreams = dreams.filter(function(d) { return d.date.startsWith(monthStr); });
            const countBar = document.getElementById('cal-count-bar');
            countBar.textContent = monthDreams.length > 0 ? '本月 ' + monthDreams.length + ' 个封印 ·' : '';

            const dreamDateSet = new Set(dreams.map(function(d) { return d.date; }));
            const today = new Date();
            const firstWeekday = new Date(year, month, 1).getDay();
            const daysInMonth = new Date(year, month + 1, 0).getDate();

            const grid = document.getElementById('cal-grid');
            grid.innerHTML = '';

            // 月首空白格
            for (let i = 0; i < firstWeekday; i++) {
                const blank = document.createElement('div');
                blank.className = 'cal-day';
                grid.appendChild(blank);
            }

            // 日期格
            for (let d = 1; d <= daysInMonth; d++) {
                const dateStr = year + '-' + String(month + 1).padStart(2, '0') + '-' + String(d).padStart(2, '0');
                const cell = document.createElement('div');
                cell.className = 'cal-day';
                cell.textContent = d;

                const isToday = today.getFullYear() === year && today.getMonth() === month && today.getDate() === d;
                if (isToday) cell.classList.add('today');

                if (dreamDateSet.has(dateStr)) {
                    cell.classList.add('has-dream');
                    cell.dataset.date = dateStr;
                    (function(ds, el) {
                        el.addEventListener('click', function() { selectCalDay(ds, el); });
                    })(dateStr, cell);
                } else {
                    cell.addEventListener('click', hideCalBottomSheet);
                }

                grid.appendChild(cell);
            }

            hideCalBottomSheet();
        }

        function changeMonth(delta) {
            calState.month += delta;
            if (calState.month > 11) { calState.month = 0; calState.year++; }
            if (calState.month < 0)  { calState.month = 11; calState.year--; }
            renderCalendar(calState.year, calState.month);
        }

        function selectCalDay(dateStr, clickedCell) {
            // 清除之前的选中状态
            document.querySelectorAll('#cal-grid .cal-day.selected').forEach(function(el) {
                el.classList.remove('selected');
            });
            clickedCell.classList.add('selected');

            const dream = dreams.find(function(d) { return d.date === dateStr; });
            if (!dream) return;

            // 格式化日期文字
            const parts = dateStr.split('-');
            const y = parseInt(parts[0]), m = parseInt(parts[1]), day = parseInt(parts[2]);
            const weekLabels = ['日','一','二','三','四','五','六'];
            const weekLabel = weekLabels[new Date(y, m - 1, day).getDay()];
            document.getElementById('cal-bs-date').textContent =
                '· ' + y + '年 ' + m + '月 ' + day + '日 · 周' + weekLabel + ' ·';

            document.getElementById('cal-bs-title').textContent = dream.title;

            const tagsEl = document.getElementById('cal-bs-tags');
            tagsEl.innerHTML = dream.tags.map(function(t) {
                return '<span class="cal-bs-tag">#' + t + '</span>';
            }).join('');

            document.getElementById('cal-bs-preview').textContent = dream.preview;

            // 显示底部抽屉
            document.getElementById('cal-empty-state').style.display = 'none';
            document.getElementById('cal-bottom-sheet').classList.add('open');
        }

        function hideCalBottomSheet() {
            document.querySelectorAll('#cal-grid .cal-day.selected').forEach(function(el) {
                el.classList.remove('selected');
            });
            document.getElementById('cal-empty-state').style.display = '';
            document.getElementById('cal-bottom-sheet').classList.remove('open');
        }
```

- [ ] **Step 2: 刷新浏览器，在控制台手动测试**

```js
// 直接测试渲染
initCalendar();
// 期望: 日历格子渲染，有梦日期显示 ◈ 符文

// 测试月份切换
changeMonth(-1);
// 期望: 切到上个月，2026年3月，28日和15日有 ◈

// 测试选中日期
const cell = document.querySelector('#cal-grid .cal-day.has-dream');
cell.click();
// 期望: 底部抽屉滑出，显示梦境标题和标签
```

- [ ] **Step 3: 提交**

```bash
git add prototype/index.html
git commit -m "feat: add calendar render, month nav, and bottom sheet JS logic"
```

---

## Task 5: 更新入口 onclick，挂载 initCalendar

**Files:**
- Modify: `prototype/index.html`（3 处 onclick 属性）

- [ ] **Step 1: 主页入口 — 添加 `initCalendar()` 调用**

找到（约第 704 行）：
```html
            <div class="archive-btn" onclick="goToStep('step-home', 'step-archive')">
```

替换为：
```html
            <div class="archive-btn" onclick="goToStep('step-home', 'step-archive'); initCalendar();">
```

- [ ] **Step 2: 视觉页入口 — 添加 `initCalendar()` 调用**

找到（约第 919 行）：
```html
                <button class="btn-manifest" id="archive-btn" style="display: none; margin-top: 10px; background: rgba(139, 92, 246, 0.1); border-color: rgba(139, 92, 246, 0.3);" onclick="goToStep('step-visual', 'step-archive')">
```

替换为：
```html
                <button class="btn-manifest" id="archive-btn" style="display: none; margin-top: 10px; background: rgba(139, 92, 246, 0.1); border-color: rgba(139, 92, 246, 0.3);" onclick="goToStep('step-visual', 'step-archive'); initCalendar();">
```

- [ ] **Step 3: 详情页返回 — 添加 `hideCalBottomSheet()` 调用**

找到（约第 751 行）：
```html
            <div class="nav-back" onclick="goToStep('step-archived-detail', 'step-archive')">⟵ 返回封印录</div>
```

替换为：
```html
            <div class="nav-back" onclick="goToStep('step-archived-detail', 'step-archive'); hideCalBottomSheet();">⟵ 返回封印录</div>
```

- [ ] **Step 4: 提交**

```bash
git add prototype/index.html
git commit -m "feat: wire initCalendar and hideCalBottomSheet to navigation entry points"
```

---

## Task 6: 手动端到端测试

**Files:** 无代码变更，验证后提交修复（如有）

- [ ] **Step 1: 启动服务**

```bash
python3 -m http.server 8080
```

打开 `http://localhost:8080/prototype/index.html`

- [ ] **Step 2: 测试流程 A — 从主页进入**

1. 点击右下角「📜 梦境封印录」
2. 期望：日历显示当前月份（2026年4月），4日显示 ◈ 符文
3. 点击左上角「‹」切换到3月，期望：15日、28日显示 ◈
4. 切回4月，点击「4」，期望：底部抽屉滑入，显示「高维的机器与光洞」、标签、预览文字
5. 点击其他无梦日期，期望：抽屉收起，引导文案重新出现
6. 再次点击「4」，点击「深入凝视 →」，期望：跳转到详情页
7. 详情页点击「⟵ 返回封印录」，期望：回到日历，抽屉已收起

- [ ] **Step 3: 测试流程 B — 重复进入不残留状态**

1. 从主页进入封印录，选中某日期（抽屉打开）
2. 点返回主页，再次点击「梦境封印录」
3. 期望：日历重新初始化，回到当月，无残留选中状态，抽屉已关闭

- [ ] **Step 4: 测试详情页视觉**

进入详情页（`step-archived-detail`），确认 `.archive-date`、`.archive-tags` 样式正常，图片正常显示。

- [ ] **Step 5: 检查文件大小**

```bash
du -h prototype/index.html
```

确认没有意外的体积膨胀。

- [ ] **Step 6: 最终提交**

```bash
git add prototype/index.html
git commit -m "feat: complete calendar view for 梦境封印录"
```
