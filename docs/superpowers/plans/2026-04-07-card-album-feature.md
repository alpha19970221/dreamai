# 回忆卡册 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在「梦境回忆录」页面内新增标签切换栏，支持在现有日历视图与新增卡册视图之间切换，卡册以单列列表展示所有梦境，点击卡片跳转详情页。

**Architecture:** 所有改动都在单文件 `prototype/index.html` 中完成，分三段：CSS 新增卡牌样式、HTML 新增标签栏与卡册容器、JS 新增 `switchCalView()` 与 `renderCards()` 函数，同时为已有月份导航和星期栏补充 `id` 属性以便 JS 控制显隐。`calView` 变量（`'calendar'` | `'cards'`）管理当前视图状态。

**Tech Stack:** 原生 HTML/CSS/JS，单文件，禁止引入外部依赖。

---

## File Structure

| 文件 | 改动说明 |
|------|----------|
| `prototype/index.html` (CSS ~456–587行) | 新增 tab 栏与卡册列表 CSS |
| `prototype/index.html` (HTML ~840–884行) | 补 id、插入 tab 栏、插入卡册列表容器 |
| `prototype/index.html` (JS ~1593–1696行) | 新增 `calView` 变量、`switchCalView()`、`renderCards()`，更新 `initCalendar()` |

---

### Task 1: Add CSS for tab bar and card list

**Files:**
- Modify: `prototype/index.html` (CSS 段，在现有 `.cal-bs-btn:hover` 规则之后，约第 586 行)

**背景知识：** `prototype/index.html` 是单文件 SPA，所有 CSS 写在 `<style>` 内。现有日历样式以 `/* --- 日历视图 (Calendar) --- */` 开头（约第 456 行），`.cal-bs-btn:hover` 是该段最后一条规则（约第 585 行）。在其后追加新样式。

- [ ] **Step 1: 在 `.cal-bs-btn:hover { ... }` 规则之后追加以下 CSS**

在 `prototype/index.html` 中找到：
```css
        .cal-bs-btn:hover { background: rgba(139,92,246,0.25); }
```

在其后插入：
```css

        /* Tab 切换栏 */
        .cal-tab-bar {
            display: flex;
            border-bottom: 1px solid rgba(255,255,255,0.07);
            flex-shrink: 0;
        }
        .cal-tab {
            flex: 1; text-align: center; padding: 10px 0;
            font-size: 0.82rem; color: rgba(255,255,255,0.3);
            cursor: pointer; transition: color 0.2s;
            border-bottom: 2px solid transparent;
        }
        .cal-tab.active {
            color: #a78bfa;
            border-bottom: 2px solid #a78bfa;
        }

        /* 卡册列表 */
        .cal-card-list {
            flex: 1; overflow-y: auto;
            padding: 12px 20px;
            display: flex; flex-direction: column; gap: 10px;
        }
        .cal-card-item {
            display: flex; align-items: flex-start; gap: 12px;
            background: rgba(255,255,255,0.03);
            border: 1px solid rgba(139,92,246,0.15);
            border-radius: 12px; padding: 14px 16px;
            cursor: pointer; transition: background 0.2s;
            flex-shrink: 0;
        }
        .cal-card-item:hover { background: rgba(139,92,246,0.08); }
        .cal-card-rune {
            font-size: 1.4rem; color: rgba(139,92,246,0.4);
            line-height: 1; flex-shrink: 0; margin-top: 2px;
        }
        .cal-card-body { flex: 1; min-width: 0; }
        .cal-card-date {
            font-size: 0.7rem; color: rgba(139,92,246,0.6); margin-bottom: 4px;
        }
        .cal-card-title {
            font-family: 'Noto Serif SC', serif; font-size: 1rem;
            color: #fff; margin-bottom: 4px;
        }
        .cal-card-preview {
            font-size: 0.78rem; color: rgba(255,255,255,0.35);
            overflow: hidden; text-overflow: ellipsis; white-space: nowrap;
        }
```

- [ ] **Step 2: 手动验证 CSS 已插入**

打开 `prototype/index.html`，搜索 `.cal-tab-bar`，确认规则存在于文件中。

- [ ] **Step 3: Commit**

```bash
git add prototype/index.html
git commit -m "style: add tab bar and card list CSS for 回忆卡册"
```

---

### Task 2: Add HTML — tab bar and card list to step-archive

**Files:**
- Modify: `prototype/index.html` (HTML 段，`#step-archive` 内部，约第 834–884 行)

**背景知识：**
- `#step-archive` 结构：`.cal-header` → `.cal-month-nav` → `.cal-weekdays` → `#cal-grid` → `#cal-count-bar` → `#cal-empty-state` → `.cal-bottom-sheet`
- `.cal-month-nav` 和 `.cal-weekdays` 目前没有 `id`，需要补上，以便 JS 通过 `getElementById` 控制显隐
- tab 栏插入在 `.cal-header` 之后、`.cal-month-nav` 之前
- 卡册列表容器插入在 `.cal-bottom-sheet` 之前（兄弟节点，不在抽屉内）

- [ ] **Step 1: 给 `.cal-month-nav` 补 id**

找到：
```html
            <!-- 月份导航 -->
            <div class="cal-month-nav">
```
改为：
```html
            <!-- 月份导航 -->
            <div class="cal-month-nav" id="cal-month-nav">
```

- [ ] **Step 2: 给 `.cal-weekdays` 补 id**

找到：
```html
            <!-- 星期标题行 -->
            <div class="cal-weekdays">
```
改为：
```html
            <!-- 星期标题行 -->
            <div class="cal-weekdays" id="cal-weekdays">
```

- [ ] **Step 3: 在 `.cal-header` 结束后插入 tab 切换栏**

找到：
```html
            </div>

            <!-- 月份导航 -->
            <div class="cal-month-nav" id="cal-month-nav">
```
改为：
```html
            </div>

            <!-- 视图切换标签 -->
            <div class="cal-tab-bar">
                <div class="cal-tab active" id="tab-calendar" onclick="switchCalView('calendar')">📅 日历</div>
                <div class="cal-tab" id="tab-cards" onclick="switchCalView('cards')">🃏 卡册</div>
            </div>

            <!-- 月份导航 -->
            <div class="cal-month-nav" id="cal-month-nav">
```

- [ ] **Step 4: 在 `.cal-bottom-sheet` 之前插入卡册列表容器**

找到：
```html
            <!-- 底部抽屉（点击有梦日期后滑入） -->
            <div class="cal-bottom-sheet" id="cal-bottom-sheet">
```
改为：
```html
            <!-- 卡册视图（默认隐藏） -->
            <div class="cal-card-list" id="cal-card-list" style="display:none;"></div>

            <!-- 底部抽屉（点击有梦日期后滑入） -->
            <div class="cal-bottom-sheet" id="cal-bottom-sheet">
```

- [ ] **Step 5: 手动验证 HTML 结构**

在浏览器打开 `http://localhost:8080/prototype/index.html`（运行 `python3 -m http.server 8080`），进入梦境回忆录，确认顶部出现两个 tab（📅 日历 / 🃏 卡册），日历视图正常显示。此时点击「🃏 卡册」不会有效果（JS 尚未实现），属正常。

- [ ] **Step 6: Commit**

```bash
git add prototype/index.html
git commit -m "feat: add tab bar and card list HTML to step-archive"
```

---

### Task 3: Add JS — switchCalView() and renderCards()

**Files:**
- Modify: `prototype/index.html` (JS 段，在 `hideCalBottomSheet` 函数之后、`</script>` 之前，约第 1696 行)

**背景知识：**
- 现有 JS 的最后一个函数是 `hideCalBottomSheet()`，它负责关闭底部抽屉并显示空状态引导文
- `dreams` 数组定义在约第 1569 行，包含 3 条梦境记录（id, date, title, tags, preview）
- `initCalendar()` 约在第 1596 行，每次进入页面时调用，需更新以重置 tab 状态
- CLAUDE.md 规定：新增 JS 逻辑写在 `<script>` 末尾，不污染全局变量

- [ ] **Step 1: 在 `hideCalBottomSheet` 函数之后追加新变量和函数**

找到：
```js
        function hideCalBottomSheet() {
            document.querySelectorAll('#cal-grid .cal-day.selected').forEach(function(el) {
                el.classList.remove('selected');
            });
            document.getElementById('cal-empty-state').style.display = 'flex';
            document.getElementById('cal-bottom-sheet').classList.remove('open');
        }
    </script>
```

改为：
```js
        function hideCalBottomSheet() {
            document.querySelectorAll('#cal-grid .cal-day.selected').forEach(function(el) {
                el.classList.remove('selected');
            });
            document.getElementById('cal-empty-state').style.display = 'flex';
            document.getElementById('cal-bottom-sheet').classList.remove('open');
        }

        // --- 回忆卡册逻辑 ---
        let calView = 'calendar';

        function switchCalView(view) {
            calView = view;
            var isCalendar = view === 'calendar';

            // 更新 tab 高亮
            document.getElementById('tab-calendar').classList.toggle('active', isCalendar);
            document.getElementById('tab-cards').classList.toggle('active', !isCalendar);

            // 切换日历专属元素的显隐
            document.getElementById('cal-month-nav').style.display = isCalendar ? '' : 'none';
            document.getElementById('cal-weekdays').style.display = isCalendar ? '' : 'none';
            document.getElementById('cal-grid').style.display = isCalendar ? 'grid' : 'none';
            document.getElementById('cal-count-bar').style.display = isCalendar ? '' : 'none';
            document.getElementById('cal-empty-state').style.display = isCalendar ? 'flex' : 'none';

            // 关闭底部抽屉（不触发 hideCalBottomSheet 以免干扰 empty-state 显隐）
            document.querySelectorAll('#cal-grid .cal-day.selected').forEach(function(el) {
                el.classList.remove('selected');
            });
            document.getElementById('cal-bottom-sheet').classList.remove('open');

            // 切换卡册列表
            var cardList = document.getElementById('cal-card-list');
            cardList.style.display = isCalendar ? 'none' : 'flex';
            if (!isCalendar) { renderCards(); }
        }

        function renderCards() {
            var listEl = document.getElementById('cal-card-list');
            listEl.innerHTML = '';

            // 按日期降序排列（最新在上）
            var sorted = dreams.slice().sort(function(a, b) {
                return b.date.localeCompare(a.date);
            });

            var weekLabels = ['日','一','二','三','四','五','六'];

            sorted.forEach(function(dream) {
                var parts = dream.date.split('-');
                var y = parseInt(parts[0]), m = parseInt(parts[1]), d = parseInt(parts[2]);
                var weekLabel = weekLabels[new Date(y, m - 1, d).getDay()];
                var dateText = y + '年 ' + m + '月 ' + d + '日 · 周' + weekLabel;

                var card = document.createElement('div');
                card.className = 'cal-card-item';
                card.innerHTML =
                    '<div class="cal-card-rune">◈</div>' +
                    '<div class="cal-card-body">' +
                        '<div class="cal-card-date">' + dateText + '</div>' +
                        '<div class="cal-card-title">' + dream.title + '</div>' +
                        '<div class="cal-card-preview">' + dream.preview + '</div>' +
                    '</div>';
                card.onclick = function() {
                    goToStep('step-archive', 'step-archived-detail');
                };
                listEl.appendChild(card);
            });
        }
    </script>
```

- [ ] **Step 2: 更新 `initCalendar()` 以在每次进入页面时重置到日历视图**

找到：
```js
        function initCalendar() {
            calState = { year: new Date().getFullYear(), month: new Date().getMonth() };
            renderCalendar(calState.year, calState.month);
            hideCalBottomSheet();
        }
```

改为：
```js
        function initCalendar() {
            calState = { year: new Date().getFullYear(), month: new Date().getMonth() };
            switchCalView('calendar');
            renderCalendar(calState.year, calState.month);
            hideCalBottomSheet();
        }
```

- [ ] **Step 3: 手动验证完整功能**

打开 `http://localhost:8080/prototype/index.html`，按以下步骤验证：

1. 进入首页 → 点击「📜 梦境回忆录」
2. 确认默认显示日历视图，「📅 日历」tab 高亮（紫色底线）
3. 点击「🃏 卡册」tab
4. 确认日历消失，出现 3 张梦境卡片，按日期降序排列：
   - 高维的机器与光洞（2026年4月4日 · 周六）
   - 深海无止境的下坠（2026年3月28日 · 周六）
   - 在长廊中被追逐（2026年3月15日 · 周日）
5. 确认每张卡片有 ◈ 符文、日期、标题、预览文（单行截断）
6. 点击任意卡片 → 跳转到详情页
7. 详情页左上角「⟵ 返回回忆录」→ 返回，确认回到日历视图（已 reset）
8. 再次点击「🃏 卡册」tab → 卡册正常显示
9. 在卡册视图下点击「📅 日历」→ 日历正常恢复

- [ ] **Step 4: Commit**

```bash
git add prototype/index.html
git commit -m "feat: implement 回忆卡册 with tab switching and card list"
```
