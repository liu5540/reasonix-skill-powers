# Visual Companion 指南

基于浏览器的可视化脑暴助手，用于展示原型图、图表和选项。

## 何时使用

按问题而非按会话决策。判断标准：**用户是看到比读到更理解？**

**当内容本身就是视觉的，使用浏览器：**

- **UI 原型图** — 线框图、布局、导航结构、组件设计
- **架构图** — 系统组件、数据流、关系图
- **并排视觉对比** — 比较两种布局、两种配色、两个设计方向
- **设计打磨** — 当问题涉及外观和感觉、间距、视觉层级时
- **空间关系** — 状态机、流程图、实体关系图等以图表形式呈现

**当内容是文本或表格时，使用终端：**

- **需求和范围问题** — "X 是什么意思？"、"哪些功能在范围内？"
- **概念性 A/B/C 选择** — 用文字描述不同方案并挑选
- **权衡列表** — 优缺点、对比表
- **技术决策** — API 设计、数据建模、架构方案选择
- **澄清问题** — 任何答案都是文字而非视觉偏好的问题

一个关于 UI 话题的问题不一定是视觉问题。"你想要哪种向导？"是概念问题——用终端。"这些向导布局哪个感觉对？"是视觉问题——用浏览器。

## 工作原理

服务器监听一个目录中的 HTML 文件，并把最新的文件提供给浏览器。你将 HTML 内容写入 `screen_dir`，用户在浏览器中看到并可点击选择选项。选择会记录到 `state_dir/events`，你在下一轮读取。

**内容片段 vs 完整文档：** 如果你的 HTML 文件以 `<!DOCTYPE` 或 `<html` 开头，服务器会原样提供（只注入辅助脚本）。否则，服务器会自动把你的内容包装进 frame 模板——添加标题、CSS 主题、连接状态和所有交互基础设施。**默认写内容片段。** 只有在你需要完全控制页面时，才写完整文档。

## 启动会话

```bash
# 在用户同意可视化助手后再启动。--open 会自动在浏览器中打开第一个页面；
# --project-dir 会持久化原型图并支持同端口重启。
scripts/start-server.sh --project-dir /path/to/project --open

# 返回: {"type":"server-started","port":52341,
#           "url":"http://localhost:52341/?key=ab12…",
#           "screen_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/content",
#           "state_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/state"}
```

保存响应中的 `screen_dir` 和 `state_dir`。使用 `--open` 时，浏览器会在你推送第一个屏幕时自动打开——你不需要让用户手动打开，但仍可将 URL 作为 fallback 分享（无头/远程环境不会自动打开）。

**URL 包含会话密钥（`?key=…`）。** 服务器会拒绝没有密钥的请求，因此始终给用户 `url` 字段中的**完整** URL——不要去掉查询字符串，也不要只给裸的 `http://host:port`。该密钥控制 HTTP 和 WebSocket 访问，防止 stray 浏览器标签页或网络中其他机器读取屏幕或注入事件。首次加载后，浏览器会通过 cookie 记住密钥，因此刷新和 `/files/*` 资源访问无需重复带密钥。

**查找连接信息：** 服务器将启动 JSON 写入 `$STATE_DIR/server-info`。如果你在后台启动服务器且没有捕获 stdout，读取该文件即可获取 URL 和端口。使用 `--project-dir` 时，在 `<project>/.superpowers/brainstorm/` 下查找会话目录。

**注意：** 将项目根目录作为 `--project-dir` 传入，以便原型图持久化在 `.superpowers/brainstorm/` 中，并在服务器重启后保留。如果不传，文件会放到 `/tmp` 并被清理。提醒用户将 `.superpowers/` 加入 `.gitignore`（如果尚未添加）。

**按平台启动服务器：**

**Claude Code:**
```bash
# 默认模式即可——脚本会自行后台运行服务器。
scripts/start-server.sh --project-dir /path/to/project --open
```

在 Windows 上，脚本会自动检测并切换到前台模式（这会阻塞工具调用）。在 Bash 工具调用上使用 `run_in_background: true`，使服务器跨会话轮次保持运行，然后在下一轮读取 `$STATE_DIR/server-info` 获取 URL 和端口。

**Codex:**
```bash
# Codex 会回收后台进程。脚本会自动检测 CODEX_CI 并切换到前台模式。
# 正常运行即可——不需要额外参数。
scripts/start-server.sh --project-dir /path/to/project --open
```

**Copilot CLI:**
```bash
# 使用 --foreground 并通过 bash 工具以 async 模式启动服务器，
# 使进程跨轮次保持运行。保存返回的 shellId，以便后续通过 read_bash / stop_bash 与其交互。
scripts/start-server.sh --project-dir /path/to/project --open --foreground
```

**其他环境：** 服务器必须在后台跨会话轮次保持运行。如果你的环境会回收 detached 进程，使用 `--foreground` 并通过你平台的背景执行机制启动命令。

如果你的浏览器无法访问该 URL（常见于远程/容器化环境），绑定一个非 loopback 主机：

```bash
scripts/start-server.sh \
  --project-dir /path/to/project \
  --host 0.0.0.0 \
  --url-host localhost
```

使用 `--url-host` 控制返回 URL JSON 中打印的主机名。

## 循环

1. **检查服务器是否存活**，然后向 `screen_dir` 中的新文件**写入 HTML**：
   - **必须：在引用 URL 或推送屏幕前确认服务器存活。** 检查 `$STATE_DIR/server-info` 是否存在且 `$STATE_DIR/server-stopped` 不存在。如果服务器已关闭，用 **相同的 `--project-dir`** 重启 `start-server.sh`——它会复用同一端口，因此用户已打开的浏览器标签页会自动重连（服务器关闭期间显示"暂停"遮罩），你也不需要发送新 URL。服务器默认在空闲 4 小时后自动退出（可用 `--idle-timeout-minutes` 配置）。
   - 使用语义化文件名：`platform.html`、`visual-style.html`、`layout.html`
   - **永远不要复用文件名** — 每个屏幕都要用新文件
   - 使用你的文件创建工具 — **永远不要用 cat/heredoc**（会把噪音输出到终端）
   - 服务器自动提供最新的文件

2. **告诉用户会发生什么并结束本轮：**
   - 每一步都提醒 URL（不只是第一步）
   - 给屏幕内容一个简短的文字摘要（例如："展示首页的 3 种布局选项"）
   - 让用户在终端回复："看一看，告诉我你的想法。如果你想选择，可以点击某个选项。"

3. **下一轮** — 用户在终端回复后：
   - 如果存在，读取 `$STATE_DIR/events` — 其中包含用户的浏览器交互（点击、选择），以 JSON lines 格式记录
   - 将其与用户的终端文字合并，形成完整画面
   - 终端消息是主要反馈；`state_dir/events` 提供结构化的交互数据

4. **迭代或推进** — 如果反馈改变了当前屏幕，写一个新文件（例如 `layout-v2.html`）。只有当前步骤被确认后，才进入下一个问题。

5. **返回终端时卸载** — 当下一步不需要浏览器时（例如澄清问题、权衡讨论），推送一个等待页面来清空旧内容：

   ```html
   <!-- filename: waiting.html (或 waiting-2.html 等) -->
   <div style="display:flex;align-items:center;justify-content:center;min-height:60vh">
     <p class="subtitle">Continuing in terminal...</p>
   </div>
   ```

   这能防止用户在对话已经推进时，还盯着已经做出选择的旧屏幕。当下一个视觉问题出现时，像往常一样推送新内容文件。

6. 重复直到完成。

## 撰写内容片段

只写页面中要显示的内容。服务器会自动将其包装进 frame 模板（标题、主题 CSS、连接状态、所有交互基础设施）。

**最小示例：**

```html
<h2>Which layout works better?</h2>
<p class="subtitle">Consider readability and visual hierarchy</p>

<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>Single Column</h3>
      <p>Clean, focused reading experience</p>
    </div>
  </div>
  <div class="option" data-choice="b" onclick="toggleSelect(this)">
    <div class="letter">B</div>
    <div class="content">
      <h3>Two Column</h3>
      <p>Sidebar navigation with main content</p>
    </div>
  </div>
</div>
```

就这些。不需要 `<html>`、CSS 或 `<script>` 标签。服务器会提供所有内容。

## 可用 CSS 类

frame 模板为你的内容提供以下 CSS 类：

### 选项（A/B/C 选择）

```html
<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>Title</h3>
      <p>Description</p>
    </div>
  </div>
</div>
```

**多选：** 在容器上添加 `data-multiselect`，让用户可以选择多个选项。每次点击都会切换该选项的选中样式。

```html
<div class="options" data-multiselect>
  <!-- 选项标记相同 — 用户可以多选/取消选择 -->
</div>
```

### 卡片（视觉设计）

```html
<div class="cards">
  <div class="card" data-choice="design1" onclick="toggleSelect(this)">
    <div class="card-image"><!-- mockup content --></div>
    <div class="card-body">
      <h3>Name</h3>
      <p>Description</p>
    </div>
  </div>
</div>
```

### 原型容器

```html
<div class="mockup">
  <div class="mockup-header">Preview: Dashboard Layout</div>
  <div class="mockup-body"><!-- your mockup HTML --></div>
</div>
```

### 分屏视图（并排）

```html
<div class="split">
  <div class="mockup"><!-- left --></div>
  <div class="mockup"><!-- right --></div>
</div>
```

### 优缺点

```html
<div class="pros-cons">
  <div class="pros"><h4>Pros</h4><ul><li>Benefit</li></ul></div>
  <div class="cons"><h4>Cons</h4><ul><li>Drawback</li></ul></div>
</div>
```

### 模拟元素（线框构建块）

```html
<div class="mock-nav">Logo | Home | About | Contact</div>
<div style="display: flex;">
  <div class="mock-sidebar">Navigation</div>
  <div class="mock-content">Main content area</div>
</div>
<button class="mock-button">Action Button</button>
<input class="mock-input" placeholder="Input field">
<div class="placeholder">Placeholder area</div>
```

### 排版与章节

- `h2` — 页面标题
- `h3` — 章节标题
- `.subtitle` — 标题下方的次要文字
- `.section` — 带底部间距的内容块
- `.label` — 小写大写标签文字

## 浏览器事件格式

当用户点击浏览器中的选项时，交互会记录到 `$STATE_DIR/events`（每行一个 JSON 对象）。推送新屏幕时，该文件会自动清空。

```jsonl
{"type":"click","choice":"a","text":"Option A - Simple Layout","timestamp":1706000101}
{"type":"click","choice":"c","text":"Option C - Complex Grid","timestamp":1706000108}
{"type":"click","choice":"b","text":"Option B - Hybrid","timestamp":1706000115}
```

完整的事件流展示了用户的探索路径——他们可能在确定前点击多个选项。最后一个 `choice` 事件通常是最终选择，但点击模式也能反映犹豫或值得追问的偏好。

如果 `$STATE_DIR/events` 不存在，说明用户没有与浏览器交互——只使用他们的终端文字。

## 设计建议

- **按问题调整保真度** — 布局用线框图，打磨问题用高保真
- **每页都要解释问题** — "哪个布局看起来更专业？"而不是简单的"选一个"
- **推进前先迭代** — 如果反馈改变了当前屏幕，写一个新版本
- **每屏最多 2-4 个选项**
- **重要时使用真实内容** — 摄影作品集就用真实图片（Unsplash）。占位内容会掩盖设计问题。
- **保持原型图简洁** — 聚焦布局和结构，而非像素级完美

## 文件命名

- 使用语义化名称：`platform.html`、`visual-style.html`、`layout.html`
- 永远不要复用文件名 — 每个屏幕必须是一个新文件
- 迭代时附加版本后缀，如 `layout-v2.html`、`layout-v3.html`
- 服务器按修改时间提供最新的文件

## 清理

```bash
scripts/stop-server.sh $SESSION_DIR
```

如果会话使用了 `--project-dir`，原型图文件会持久化在 `.superpowers/brainstorm/` 中，供后续参考。只有 `/tmp` 中的会话在停止时会被删除。

## 参考

- frame 模板（CSS 参考）：`scripts/frame-template.html`
- 辅助脚本（客户端）：`scripts/helper.js`
