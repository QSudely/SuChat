# SuChat

基于 WebRTC (PeerJS) 的端到端加密 P2P 多人聊天应用，支持文字、文件、图片、语音消息，内置插件系统。

> A WebRTC (PeerJS) based end-to-end encrypted P2P multi-user chat application with text, file, image, voice messaging, and a built-in plugin system.

## 功能概览

- **P2P 通信**：基于 PeerJS 的 WebRTC 直连，无需服务器中转
- **端到端加密**：ECDH 密钥交换 + AES-GCM 消息加密
- **多媒体消息**：文字、文件、图片、语音、表情包
- **主题切换**：暗色 / 亮色双主题
- **存储管理**：聊天记录、文件、插件统一管理
- **插件系统**：支持第三方 `.jsc` 插件扩展功能

> **Features Overview**
> - **P2P Communication**: WebRTC direct connection via PeerJS, no server relay
> - **End-to-End Encryption**: ECDH key exchange + AES-GCM message encryption
> - **Multimedia Messages**: Text, files, images, voice, stickers
> - **Theme Switching**: Dark / Light dual themes
> - **Storage Management**: Unified management of chat records, files, and plugins
> - **Plugin System**: Third-party `.jsc` plugin support for extensibility

---

## 快速开始

直接用浏览器打开 `SuChat.html` 即可运行，无需安装任何依赖。

首次打开会自动生成一个用户 ID，将 ID 分享给对方即可建立连接。

> **Quick Start**
> Open `SuChat.html` directly in a browser — no installation required. A unique user ID is auto-generated on first launch. Share your ID with others to establish a connection.

---

## 插件系统

SuChat 内置插件引擎，插件为纯 JavaScript 文件，使用 `.jsc` 自定义后缀。

> SuChat features a built-in plugin engine. Plugins are pure JavaScript files with a custom `.jsc` extension.

### 安装插件

**我的 → 存储管理 → 插件管理 → ＋** → 选择 `.jsc` 文件导入

> **Install a Plugin**: Navigate to **Profile → Storage Management → Plugins → ＋** and select a `.jsc` file to import.

### 插件文件格式

插件文件必须是纯 JS 内容，后缀为 `.jsc`。文件头部通过 JSDoc 块注释或行注释声明元信息：

> Plugin files must be pure JS content with a `.jsc` extension. Declare metadata in the file header via JSDoc block comments or line comments:

```javascript
/**
 * @id          my-plugin-id          // 唯一标识（必填）
 * @name        我的插件                // 显示名称（必填）
 * @version     1.0.0                 // 版本号（必填）
 * @author      作者名                 // 作者（必填）
 * @desc        插件描述               // 功能描述（必填）
 * @permissions storage, notification  // 所需权限（必填，逗号分隔）
 */

(function (SuChatPlugin) {
    'use strict';
    // 插件逻辑...
})(
    (typeof SuChatPlugin !== 'undefined' && SuChatPlugin) ||
    (typeof window !== 'undefined' && window.SuChatPlugin)
);
```

### 元信息字段

| 字段 | 说明 | 必填 |
|------|------|------|
| `@id` | 插件唯一标识，相同 ID 安装时覆盖旧版 | 是 |
| `@name` | 显示名称 | 是 |
| `@version` | 版本号 | 是 |
| `@author` | 作者 | 是 |
| `@desc` | 功能描述 | 是 |
| `@permissions` | 所需权限，逗号分隔 | 是 |

> **Metadata Fields**
> | Field | Description | Required |
> |-------|-------------|----------|
> | `@id` | Unique plugin identifier; same ID overwrites older versions | Yes |
> | `@name` | Display name | Yes |
> | `@version` | Version number | Yes |
> | `@author` | Author name | Yes |
> | `@desc` | Feature description | Yes |
> | `@permissions` | Required permissions, comma-separated | Yes |

### 权限列表

| 权限标识 | 说明 |
|----------|------|
| `storage` | 独立存储读写 |
| `http` | 外部网络请求（需用户确认授权） |
| `notification` | 系统通知 |
| `schedule` | 定时任务 |
| `hook_send` | 发送消息拦截钩子 |
| `hook_recv` | 接收消息拦截钩子 |
| `render_filter` | 消息渲染过滤器 |
| `ui_toolbar` | 工具栏扩展按钮 |
| `ui_msg_menu` | 消息右键菜单项 |
| `ui_bottom_nav` | 底部导航栏扩展页 |
| `ui_text_filter` | UI 文本过滤器（翻译/字体补丁等） |
| `file_access` | 文件访问 |

> **Permission List**
> | Permission | Description |
> |------------|-------------|
> | `storage` | Isolated storage read/write |
> | `http` | External HTTP requests (requires user confirmation) |
> | `notification` | System notifications |
> | `schedule` | Scheduled tasks |
> | `hook_send` | Pre-send message hook |
> | `hook_recv` | Post-receive message hook |
> | `render_filter` | Message render filter |
> | `ui_toolbar` | Toolbar extension button |
> | `ui_msg_menu` | Message context menu item |
> | `ui_bottom_nav` | Bottom navigation extension page |
> | `ui_text_filter` | UI text filter (translation/font patches, etc.) |
> | `file_access` | File access |

---

## 插件 API

插件通过 `SuChatPlugin`（或别名 `api`）访问宿主能力。所有 API 均受权限控制。

> Plugins access host capabilities through `SuChatPlugin` (alias `api`). All APIs are permission-controlled.

### info

插件自身信息。

> Plugin self-information.

```javascript
SuChatPlugin.info.id       // 插件 ID
SuChatPlugin.info.name     // 插件名称
SuChatPlugin.info.version  // 插件版本
SuChatPlugin.info.author   // 作者
SuChatPlugin.info.description // 描述
```

### storage

独立隔离存储，命名空间为 `suchat_plug_store:{pluginId}:`，与核心数据完全隔离。**同步 API**。

> Isolated storage with namespace `suchat_plug_store:{pluginId}:`, fully separated from core data. **Synchronous API**.

```javascript
SuChatPlugin.storage.get(key)        // 读取，返回 JSON 解析后的值或 null
SuChatPlugin.storage.set(key, value) // 写入（自动 JSON 序列化）
SuChatPlugin.storage.remove(key)     // 删除单个键
SuChatPlugin.storage.keys()          // 返回所有键名数组
SuChatPlugin.storage.clear()         // 清空当前插件的所有存储
```

### http

外部 HTTP 请求。首次调用时弹出确认对话框，用户授权后后续请求不再询问。

> External HTTP requests. A confirmation dialog appears on first call; subsequent requests are no longer prompted after authorization.

```javascript
const res = await SuChatPlugin.http.fetch(url, init);
// init 同标准 fetch options: { method, headers, body, cache, ... }
const data = await res.json();
```

### notify

发送系统通知。自动处理浏览器通知权限申请，不支持时回退为 Toast。

> Sends system notifications. Automatically handles browser notification permission requests; falls back to Toast if unsupported.

```javascript
SuChatPlugin.notify('标题', { body: '通知正文' });
```

### schedule

定时任务管理。插件禁用/卸载时自动清理。

> Scheduled task management. Automatically cleaned up when plugin is disabled/uninstalled.

```javascript
const taskId = SuChatPlugin.schedule.setInterval(fn, 60000); // 定时循环（最小 1000ms）
const taskId = SuChatPlugin.schedule.setTimeout(fn, 5000);   // 延时一次
SuChatPlugin.schedule.clear(taskId);                          // 取消任务
```

### toast / log

```javascript
SuChatPlugin.toast('提示文字');           // 显示 Toast
SuChatPlugin.log('调试信息', someData);   // 带插件名前缀的 console.log
```

### chat

会话相关能力。

> Conversation-related capabilities.

```javascript
// 向指定会话发送文字消息（走正常发送路径，会触发 pre-send hook）
SuChatPlugin.sendMessage(sessionId, text);

// 获取所有会话列表
const sessions = SuChatPlugin.getSessions();
// [{ id, name, type, online, lastMessage, unread }, ...]

// 获取当前打开的会话
const current = SuChatPlugin.getCurrentSession();
// { id, name } 或 null
```

### onUnload

注册卸载清理钩子。插件被禁用或卸载时自动调用，用于清理样式、断开 Observer、清除定时器等。

> Registers an unload cleanup hook. Automatically called when the plugin is disabled or uninstalled — use it to clean up styles, disconnect Observers, clear timers, etc.

```javascript
SuChatPlugin.onUnload(function () {
    // 清理 DOM 元素
    // 断开 MutationObserver
    // 移除事件监听
});
```

---

## UI 扩展

### ui.registerToolbar(opts)

在聊天扩展面板（输入框旁的 **＋** → 文件 tab）注册一个按钮。

> Registers a button in the chat extension panel (the **＋** next to the input → File tab).

```javascript
SuChatPlugin.ui.registerToolbar({
    id: 'my_btn',           // 按钮唯一 ID
    icon: '🔧',             // 图标 emoji
    label: '我的工具',       // 按钮文字
    title: '点击执行功能',   // tooltip
    visible: function () { return true; },  // 是否显示
    onClick: function (api, ctx) {
        api.toast('按钮被点击了');
    },
});
```

### ui.registerToolbarButton(opts)

`registerToolbar` 的别名，完全等价。

> Alias for `registerToolbar`, fully equivalent.

### ui.registerMessageMenuItem(opts)

在消息右键菜单中添加自定义菜单项。

> Adds a custom item to the message context menu.

```javascript
SuChatPlugin.ui.registerMessageMenuItem({
    id: 'translate_msg',
    icon: '🌐',
    label: '翻译',
    visible: function (msg, sessionId) {
        return msg.type === 'text';  // 仅文本消息显示
    },
    onClick: function (msg, sessionId, api) {
        // 处理点击
    },
});
```

### ui.registerBottomNav(opts)

在底部导航栏注册一个新的 Tab 页。

> Registers a new tab page in the bottom navigation bar.

```javascript
SuChatPlugin.ui.registerBottomNav({
    id: 'my_page',
    icon: '📊',
    label: '数据',
    badge: function () { return '3'; },  // 可选，返回角标文字
    position: 'before_profile',           // 插入位置
    onOpen: function (api, containerEl) {
        // containerEl 是一个 DOM 元素，往里面渲染页面内容
        containerEl.innerHTML = '<h1>我的插件页面</h1>';
    },
});
```

**position 取值：**

| 值 | 说明 |
|----|------|
| `start` | 所有导航最前面 |
| `before_sessions` / `before_messages` | 会话/消息前面 |
| `before_requests` | 请求前面 |
| `before_profile` | 我的 前面（默认） |
| `after_profile` / `end` | 我的 后面 |

也可以直接传原生 tab 的 `data-page` 值，如 `before:pageSessions`、`after:pageProfile`。

> **position values:**
> | Value | Description |
> |-------|-------------|
> | `start` | Before all navigation items |
> | `before_sessions` / `before_messages` | Before Sessions/Messages tab |
> | `before_requests` | Before Requests tab |
> | `before_profile` | Before Profile tab (default) |
> | `after_profile` / `end` | After Profile tab |
> You can also pass native tab `data-page` values, e.g. `before:pageSessions`, `after:pageProfile`.

### ui.registerTextFilter(fn)

注册 UI 文本过滤器。宿主在渲染界面文本（会话名、状态标签、按钮文字、Toast、空状态提示等）时会调用所有已注册的过滤器，插件可借此实现翻译补丁、文本替换等功能。

> Registers a UI text filter. The host calls all registered filters when rendering UI text (session names, status labels, button text, Toast, empty state prompts, etc.). Plugins can use this for translation patches, text replacement, etc.

**调用时机**：宿主在以下渲染路径中自动调用：
- 会话列表（会话名、最后消息预览、在线/离线标签、空状态）
- 好友请求列表（请求者名称、同意/拒绝按钮、空状态）
- 个人主页（连接状态文字）
- 聊天面板（标题、在线/离线状态）
- Toast 提示

> **Call sites**: The host automatically calls filters in these render paths:
> - Session list (name, last message preview, online/offline label, empty state)
> - Friend request list (requester name, accept/reject buttons, empty state)
> - Profile page (connection status)
> - Chat panel (title, online/offline status)
> - Toast messages

**参数**：
- `fn(text, context)` — 过滤函数
  - `text`（string）：原始文本
  - `context`（object）：上下文信息，包含 `source` 字段标识文本来源
- 返回值：过滤后的文本（string），返回原 text 表示不修改

> **Parameters**:
> - `fn(text, context)` — Filter function
>   - `text` (string): Original text
>   - `context` (object): Context info with a `source` field identifying the text source
> - Returns: Filtered text (string); return the original text to skip

**context.source 取值**：

| source | 说明 |
|--------|------|
| `toast` | Toast 提示文字 |
| `session_name` | 会话列表中的联系人名称 |
| `session_status` | 会话列表中的在线/离线标签 |
| `session_lastmsg` | 会话列表中的最后消息预览 |
| `chat_title` | 聊天面板标题 |
| `chat_status` | 聊天面板在线/离线状态 |
| `request_name` | 好友请求列表中的名称 |
| `request_action` | 好友请求列表中的按钮文字（同意/拒绝） |
| `request_id_label` | 好友请求列表中的 ID 标签 |
| `profile_status` | 个人主页的连接状态 |
| `empty_state` | 空状态提示文字 |

```javascript
// 示例：将所有中文"在线"翻译为英文
SuChatPlugin.ui.registerTextFilter(function (text, context) {
    var dict = { '在线': 'Online', '离线': 'Offline', '同意': 'Accept', '拒绝': 'Reject' };
    return dict[text] || text;
});
```

```javascript
// 示例：仅过滤特定来源的文本
SuChatPlugin.ui.registerTextFilter(function (text, context) {
    if (context.source === 'session_status') {
        return text === '在线' ? '🟢 Online' : '⚪ Offline';
    }
    return text;  // 其他来源不修改
});
```

> **Tip**: For static DOM text without built-in `runTextFilter` call sites (e.g. navigation bar labels), plugins can use `MutationObserver` to scan and replace DOM text nodes manually.

---

## 钩子（Hooks）

### onPreSend(fn)

发送消息前拦截。在加密前调用，返回 `null` 阻止发送，返回修改后的 msg 对象替换发送内容。

> Intercepts before sending. Called before encryption. Return `null` to block the send; return a modified msg object to replace the payload.

```javascript
SuChatPlugin.onPreSend(function (msg) {
    // msg: { type, text, sessionId, ... }
    if (msg.text && msg.text.includes('敏感词')) {
        return null;  // 阻止发送
    }
    msg.text = msg.text.toUpperCase();  // 修改内容
    return msg;
});
```

### onPostReceive(fn)

接收消息后拦截。在解密后、渲染前调用，返回修改后的 msg。

> Intercepts after receiving. Called after decryption, before rendering. Return the modified msg.

```javascript
SuChatPlugin.onPostReceive(function (msg, peerId) {
    // 过滤、转换、记录消息
    return msg;
});
```

### onRenderMessage(fn)

消息渲染过滤器。在文本插入 DOM 前调用，返回处理后的文本或 HTML。

> Message render filter. Called before text is inserted into the DOM. Return processed text or HTML.

```javascript
SuChatPlugin.onRenderMessage(function (text, msg, sessionId) {
    // 将 URL 转为链接
    return text.replace(/https?:\/\/\S+/g, function (url) {
        return '<a href="' + url + '" target="_blank">' + url + '</a>';
    });
});
```

---

## 生命周期

```
插件安装 / 启用
    │
    ├─ 解析元信息（@id @name @version ...）
    ├─ 权限检查
    ├─ 沙箱执行（new Function 隔离作用域）
    ├─ 注册钩子 / UI / 定时器
    └─ 插件运行中
         │
    禁用 / 卸载
         │
         ├─ 调用 onUnload 钩子（清理样式、Observer 等）
         ├─ 移除所有注册的 UI 元素
         ├─ 清除定时任务
         ├─ 移除钩子注册
         └─ （卸载时）清空插件独立存储
```

> **Lifecycle**
> ```
> Plugin Install / Enable
>     │
>     ├─ Parse metadata (@id @name @version ...)
>     ├─ Permission check
>     ├─ Sandbox execution (new Function, isolated scope)
>     ├─ Register hooks / UI / timers
>     └─ Plugin running
>          │
>     Disable / Uninstall
>          │
>          ├─ Call onUnload hook (cleanup styles, Observers, etc.)
>          ├─ Remove all registered UI elements
>          ├─ Clear scheduled tasks
>          ├─ Remove hook registrations
>          └─ (On uninstall) Clear plugin isolated storage
> ```

### 沙箱机制

- 插件代码通过 `new Function` 构造器执行，不共享宿主闭包变量
- `window.SuChatPlugin` 在执行期间被临时设置为当前插件的 API 对象，执行后还原
- 所有能力通过 `api` 对象受控暴露，不直接访问全局敏感 API
- 插件存储使用独立命名空间 `suchat_plug_store:{pluginId}:`

> **Sandbox Mechanism**
> - Plugin code executes via `new Function` constructor — no shared host closure variables
> - `window.SuChatPlugin` is temporarily set to the current plugin's API object during execution, restored afterward
> - All capabilities are exposed through the controlled `api` object — no direct access to global sensitive APIs
> - Plugin storage uses isolated namespace `suchat_plug_store:{pluginId}:`

---

## 示例插件

### 最小示例

```javascript
/**
 * @id          hello-world
 * @name        Hello World
 * @version     1.0.0
 * @author      Demo
 * @desc        最简示例插件
 * @permissions storage
 */
(function (SuChatPlugin) {
    'use strict';
    SuChatPlugin.toast('👋 Hello World!');
    SuChatPlugin.storage.set('last_run', Date.now());
})(
    (typeof SuChatPlugin !== 'undefined' && SuChatPlugin) ||
    window.SuChatPlugin
);
```

### 底部导航页示例

```javascript
/**
 * @id          notes-plugin
 * @name        笔记本
 * @version     1.0.0
 * @author      Demo
 * @desc        简单的笔记功能
 * @permissions ui_bottom_nav, storage
 */
(function (SuChatPlugin) {
    'use strict';
    SuChatPlugin.ui.registerBottomNav({
        id: 'notes_page',
        icon: '📝',
        label: '笔记',
        position: 'after_profile',
        onOpen: function (api, el) {
            var notes = api.storage.get('notes') || [];
            el.innerHTML = '<div style="padding:16px">'
                + '<h2>我的笔记</h2>'
                + '<textarea id="noteInput" style="width:100%;height:120px"></textarea>'
                + '<button id="saveNote">保存</button>'
                + '</div>';
            el.querySelector('#saveNote').addEventListener('click', function () {
                var text = el.querySelector('#noteInput').value;
                notes.push({ text: text, time: Date.now() });
                api.storage.set('notes', notes);
                api.toast('已保存');
            });
        },
    });
})(
    (typeof SuChatPlugin !== 'undefined' && SuChatPlugin) ||
    window.SuChatPlugin
);
```

### 消息钩子示例

```javascript
/**
 * @id          msg-logger
 * @name        消息记录器
 * @version     1.0.0
 * @author      Demo
 * @desc        记录所有收发消息
 * @permissions hook_send, hook_recv, storage
 */
(function (SuChatPlugin) {
    'use strict';
    SuChatPlugin.onPreSend(function (msg) {
        var logs = SuChatPlugin.storage.get('logs') || [];
        logs.push({ dir: 'out', type: msg.type, time: Date.now() });
        SuChatPlugin.storage.set('logs', logs);
        return msg;
    });
    SuChatPlugin.onPostReceive(function (msg, peerId) {
        var logs = SuChatPlugin.storage.get('logs') || [];
        logs.push({ dir: 'in', type: msg.type, peer: peerId, time: Date.now() });
        SuChatPlugin.storage.set('logs', logs);
        return msg;
    });
})(
    (typeof SuChatPlugin !== 'undefined' && SuChatPlugin) ||
    window.SuChatPlugin
);
```

### UI 文本过滤器示例（翻译补丁）

```javascript
/**
 * @id          en-patch
 * @name        英文补丁
 * @version     1.0.0
 * @author      Demo
 * @desc        将界面文本翻译为英文
 * @permissions ui_text_filter
 */
(function (SuChatPlugin) {
    'use strict';
    var dict = {
        '在线': 'Online',
        '离线': 'Offline',
        '同意': 'Accept',
        '拒绝': 'Reject',
        '暂无会话': 'No conversations',
        '暂无待处理请求': 'No pending requests',
        '未连接': 'Disconnected'
    };
    SuChatPlugin.ui.registerTextFilter(function (text, context) {
        return dict[text] || text;
    });
})(
    (typeof SuChatPlugin !== 'undefined' && SuChatPlugin) ||
    window.SuChatPlugin
);
```

---

## 项目结构

```
SuChat.html                    # 主应用（单文件，含 HTML/CSS/JS）
README.md                      # 本文件
```

> **Project Structure**
> ```
> SuChat.html                    # Main app (single file, HTML/CSS/JS)
> README.md                      # This file
> ```

## 技术栈

- WebRTC (PeerJS 1.5.1) — P2P 连接
- ECDH + AES-GCM — 端到端加密
- Cropper.js — 头像裁剪
- 纯原生 HTML/CSS/JS — 无框架依赖

> **Tech Stack**
> - WebRTC (PeerJS 1.5.1) — P2P connectivity
> - ECDH + AES-GCM — End-to-end encryption
> - Cropper.js — Avatar cropping
> - Vanilla HTML/CSS/JS — No framework dependencies
