# Chat Interface Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在 `#step-interpret` 视图中，点击"我想继续追问"后原地展开一个仿 WeChat 风格的聊天区域，支持用户与祭司多轮模拟对话，"生成梦境图"始终固定在底部。

**Architecture:** 所有改动均在 `prototype/index.html` 内完成。新增 CSS 追加到 `<style>` 末尾，新增 HTML 插入 `#step-interpret` 内，新增 JS 追加到 `<script>` 末尾。聊天区固定高度 `240px` + `overflow-y: auto`，不破坏现有布局。

**Tech Stack:** 纯 HTML/CSS/JS，无外部依赖。

---

### Task 1: 新增聊天区域 CSS

**Files:**
- Modify: `prototype/index.html` — `<style>` 标签末尾（`</style>` 之前）

- [ ] **Step 1: 在 `</style>` 前追加以下 CSS**

```css
        /* ── Chat Interface ── */
        #chat-area {
            display: none;
            flex-direction: column;
            gap: 0;
            margin-bottom: 10px;
            animation: slideUpFadeIn 0.4s forwards;
        }
        #chat-messages {
            height: 240px;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            gap: 10px;
            padding: 12px 4px;
            scrollbar-width: none;
        }
        #chat-messages::-webkit-scrollbar { display: none; }
        .chat-bubble {
            max-width: 78%;
            padding: 10px 14px;
            font-size: 0.88rem;
            line-height: 1.5;
            word-break: break-word;
        }
        .chat-bubble-bot {
            align-self: flex-start;
            background: rgba(255, 255, 255, 0.07);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 18px 18px 18px 4px;
            color: #c4b5fd;
        }
        .chat-bubble-bot .bubble-label {
            font-size: 0.72rem;
            color: rgba(167, 139, 250, 0.7);
            margin-bottom: 4px;
            font-family: 'Noto Serif SC', serif;
        }
        .chat-bubble-user {
            align-self: flex-end;
            background: rgba(139, 92, 246, 0.38);
            border: 1px solid rgba(167, 139, 250, 0.3);
            border-radius: 18px 18px 4px 18px;
            color: #ede9fe;
        }
        .typing-indicator {
            align-self: flex-start;
            background: rgba(255, 255, 255, 0.07);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 18px 18px 18px 4px;
            padding: 10px 16px;
            display: flex;
            gap: 5px;
            align-items: center;
        }
        .typing-dot {
            width: 6px;
            height: 6px;
            border-radius: 50%;
            background: #a78bfa;
            animation: typingBounce 1s infinite;
        }
        .typing-dot:nth-child(2) { animation-delay: 0.2s; }
        .typing-dot:nth-child(3) { animation-delay: 0.4s; }
        @keyframes typingBounce {
            0%, 60%, 100% { transform: translateY(0); opacity: 0.5; }
            30% { transform: translateY(-5px); opacity: 1; }
        }
        #chat-input-row {
            display: flex;
            gap: 8px;
            align-items: center;
            margin-bottom: 10px;
        }
        #chat-input {
            flex: 1;
            background: rgba(255, 255, 255, 0.06);
            border: 1px solid rgba(139, 92, 246, 0.4);
            border-radius: 22px;
            padding: 10px 16px;
            color: #e2e8f0;
            font-family: 'Inter', sans-serif;
            font-size: 0.88rem;
            outline: none;
            resize: none;
            height: 40px;
            line-height: 20px;
        }
        #chat-input::placeholder { color: rgba(255,255,255,0.3); }
        #chat-input:focus { border-color: rgba(139, 92, 246, 0.7); }
        #chat-send-btn {
            width: 40px;
            height: 40px;
            border-radius: 50%;
            background: rgba(139, 92, 246, 0.5);
            border: 1px solid rgba(167, 139, 250, 0.5);
            color: white;
            font-size: 1rem;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
            flex-shrink: 0;
            transition: background 0.2s;
        }
        #chat-send-btn:hover { background: rgba(139, 92, 246, 0.8); }
```

- [ ] **Step 2: 本地预览验证 CSS 无语法错误**

```bash
python3 -m http.server 8080
# 浏览器打开 http://localhost:8080/prototype/index.html，检查控制台无 CSS 错误
```

---

### Task 2: 新增聊天区域 HTML

**Files:**
- Modify: `prototype/index.html` — `#step-interpret` 视图内部

- [ ] **Step 1: 定位目标区域**

找到以下 HTML 片段（在 `#step-interpret` 内）：
```html
            <div style="flex-grow: 1;"></div>

            <div id="gen-action-btns" style="display: flex; gap: 12px; margin-bottom: 8px;">
```

- [ ] **Step 2: 在 `<div style="flex-grow: 1;"></div>` 和 `<div id="gen-action-btns"...>` 之间插入以下 HTML**

```html
            <!-- Chat Interface -->
            <div id="chat-area">
                <div id="chat-messages">
                    <!-- 开场白气泡，由 JS 初始化时注入 -->
                </div>
                <div id="chat-input-row">
                    <input type="text" id="chat-input" placeholder="向祭司追问..." />
                    <button id="chat-send-btn" onclick="sendChatMessage()">↑</button>
                </div>
            </div>
```

完整插入位置上下文：
```html
            <div style="flex-grow: 1;"></div>

            <!-- Chat Interface -->
            <div id="chat-area">
                <div id="chat-messages">
                </div>
                <div id="chat-input-row">
                    <input type="text" id="chat-input" placeholder="向祭司追问..." />
                    <button id="chat-send-btn" onclick="sendChatMessage()">↑</button>
                </div>
            </div>

            <div id="gen-action-btns" style="display: flex; gap: 12px; margin-bottom: 8px;">
```

- [ ] **Step 3: 修改"我想继续追问"按钮的 onclick**

将：
```html
<button class="btn-manifest" onclick="goToStep('step-interpret', 'step-tarot')" style="flex: 1; font-size: 0.85rem; background: rgba(139, 92, 246, 0.1); border-color: rgba(139, 92, 246, 0.3);">我想继续追问</button>
```
改为：
```html
<button class="btn-manifest" onclick="openChat()" style="flex: 1; font-size: 0.85rem; background: rgba(139, 92, 246, 0.1); border-color: rgba(139, 92, 246, 0.3);">我想继续追问</button>
```

---

### Task 3: 新增聊天 JS 逻辑

**Files:**
- Modify: `prototype/index.html` — `<script>` 标签末尾（`</script>` 之前）

- [ ] **Step 1: 在 `</script>` 前追加以下 JS**

```javascript
        // ── Chat Interface Logic ──
        const SHAMAN_REPLIES = [
            '你所追问的，正是潜意识试图让你看见的裂缝。继续说，它在倾听。',
            '这个问题本身，就是一把打开内室的钥匙。你感受到了什么？',
            '梦境的语言从不撒谎。那个画面背后，藏着你尚未命名的情绪。',
            '高维视角下，所有的困惑都是方向。你内心真正想放下的是什么？',
            '宇宙用这个梦回应了你的某个请求。那个请求，你还记得吗？',
            '意识是一面镜子，梦是镜子背后的光。你看到了自己的哪个侧面？',
            '不必急于解读，有时候只是带着这个意象生活几天，答案会自己浮现。',
            '你的追问本身就是疗愈的一部分。继续，祭司在此聆听。'
        ];
        let _lastReplyIndex = -1;

        function openChat() {
            // 隐藏"我想继续追问"按钮，保留"生成梦境图"
            const btns = document.getElementById('gen-action-btns');
            btns.style.transition = 'opacity 0.3s';
            btns.style.opacity = '0';
            setTimeout(() => {
                btns.style.display = 'none';
                // 展开聊天区
                const chatArea = document.getElementById('chat-area');
                chatArea.style.display = 'flex';
                // 注入开场白
                appendBotBubble('✦ 祭司已感应到你的追问。说吧，梦境之中，还有什么让你困惑？');
                // 重新在底部添加一个"生成梦境图"按钮（独立行）
                document.getElementById('chat-gen-btn-row').style.display = 'block';
                // 聚焦输入框
                document.getElementById('chat-input').focus();
            }, 300);
        }

        function appendBotBubble(text) {
            const msgs = document.getElementById('chat-messages');
            const bubble = document.createElement('div');
            bubble.className = 'chat-bubble chat-bubble-bot';
            bubble.innerHTML = '<div class="bubble-label">✦ 祭司</div>' + text;
            msgs.appendChild(bubble);
            msgs.scrollTop = msgs.scrollHeight;
        }

        function appendUserBubble(text) {
            const msgs = document.getElementById('chat-messages');
            const bubble = document.createElement('div');
            bubble.className = 'chat-bubble chat-bubble-user';
            bubble.textContent = text;
            msgs.appendChild(bubble);
            msgs.scrollTop = msgs.scrollHeight;
        }

        function showTyping() {
            const msgs = document.getElementById('chat-messages');
            const typing = document.createElement('div');
            typing.className = 'typing-indicator';
            typing.id = 'typing-indicator';
            typing.innerHTML = '<div class="typing-dot"></div><div class="typing-dot"></div><div class="typing-dot"></div>';
            msgs.appendChild(typing);
            msgs.scrollTop = msgs.scrollHeight;
        }

        function hideTyping() {
            const el = document.getElementById('typing-indicator');
            if (el) el.remove();
        }

        function getNextReply() {
            let idx;
            do { idx = Math.floor(Math.random() * SHAMAN_REPLIES.length); }
            while (idx === _lastReplyIndex && SHAMAN_REPLIES.length > 1);
            _lastReplyIndex = idx;
            return SHAMAN_REPLIES[idx];
        }

        function sendChatMessage() {
            const input = document.getElementById('chat-input');
            const text = input.value.trim();
            if (!text) return;
            input.value = '';
            appendUserBubble(text);
            showTyping();
            setTimeout(() => {
                hideTyping();
                appendBotBubble(getNextReply());
            }, 1200);
        }

        // 回车发送
        document.addEventListener('DOMContentLoaded', function() {
            const input = document.getElementById('chat-input');
            if (input) {
                input.addEventListener('keydown', function(e) {
                    if (e.key === 'Enter' && !e.isComposing) {
                        e.preventDefault();
                        sendChatMessage();
                    }
                });
            }
        });
```

- [ ] **Step 2: 在 HTML 中 `#chat-area` 之后、`#gen-action-btns` 之前，新增一个独立的生成梦境图按钮行**

在 Task 2 插入的 `</div><!-- end chat-area -->` 之后、`<div id="gen-action-btns"` 之前，再插入：
```html
            <div id="chat-gen-btn-row" style="display: none; margin-bottom: 8px;">
                <button class="btn-manifest" onclick="triggerGenImage()" style="width: 100%; font-size: 0.85rem;">生成梦境图</button>
            </div>
```

> 注意：原 `#gen-action-btns` 在 `openChat()` 后被隐藏，此新按钮在 `openChat()` 时展示，保证"生成梦境图"始终在底部可用。

---

### Task 4: 验证与提交

- [ ] **Step 1: 本地预览完整流程**

```bash
python3 -m http.server 8080
# 打开 http://localhost:8080/prototype/index.html
# 流程：进入解梦流程 → 到达"指引：高维的游戏" → 点击"我想继续追问"
# 验证：
# 1. 两个按钮淡出消失
# 2. 聊天区滑入，顶部有祭司开场白气泡
# 3. 输入文字，点击发送或按回车
# 4. 用户气泡右对齐出现，输入框清空
# 5. typing 三点动效出现约 1.2s
# 6. 祭司回复气泡左对齐出现
# 7. "生成梦境图"按钮在最底部可见可点击
# 8. 点击"生成梦境图"正常触发生成流程
```

- [ ] **Step 2: 检查布局无溢出**

```bash
# 在浏览器中检查：
# - 没有水平滚动条
# - 聊天区超过 4-5 条消息时内部滚动，不撑破容器
# - 414px 宽度下气泡不超宽
```

- [ ] **Step 3: 提交**

```bash
git add prototype/index.html docs/superpowers/specs/2026-04-06-chat-interface-design.md docs/superpowers/plans/2026-04-06-chat-interface.md
git commit -m "feat: add inline chat interface to step-interpret view

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```
