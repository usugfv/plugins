# dsh-client-ui-deepseek-bg

DeepSeek 官网风格皮肤：**极光背景 + 粒子鲸鱼 + 星座网格**，以及**侧边栏 / 输入框 / 消息气泡 / 代码块**的玻璃拟态。

> 当前版本（v26）已适配 DSH Desktop 的 2026-11 Web UI 内核，并处理了与 `dsh-claude-style` 共存的问题。原版依赖旧构建的 CSS Module 哈希类名（`.pI_x6G_*`、`.hHd-Xa_*`、`.uV2eYG_*` 等），在桌面端这些类名已不存在；同时桌面端的布局外壳（以及 Claude 皮肤）会用不透明背景和高特异性 `!important` 盖住背景层，所以原版会出现“插件已加载但看不见”的情况。

## 效果

- 🐋 中央粒子化鲸鱼：深色主题用 screen 提亮；浅色主题用**深灰点水印**（下移、缩小、normal 混合，避免压住正文），两套主题都有光线跟随与组装动画
- 🌌 深蓝流体极光背景 + 白色星座网格
- 🧊 侧边栏、底部输入框、用户消息气泡、代码块：半透明磨砂玻璃
- 🌗 **深色 / 浅色主题都生效**：深色是 DeepSeek 深色 hero，浅色是官方浅色 hero（浅蓝渐变 + 星座 + 粒子），切换主题实时生效
- ♿ `prefers-reduced-motion` 时降级为静态画面；触屏设备跳过星座网格

## 安装到桌面端（desktop profile）

> 桌面端读取的是 **desktop** profile，不是 web profile。装到 web profile 只对 `dsh --profile web` 启动的网页端生效。

1. 把整个目录放在任意位置，例如 `D:\agentworkspace\DSHworkspace\dsh-client-ui-deepseek-bg`。
2. 用插件管理器安装（推荐）：
   - 打开 DSH Desktop → **设置 → 插件**，选择“从本地文件夹安装 / Install bundle from folder”，指向上面的目录；
   - 或者用 CLI：`dsh plugin --profile desktop add "D:\agentworkspace\DSHworkspace\dsh-client-ui-deepseek-bg"`
3. 确认 `C:\Users\<你>\.dsh\profiles\desktop\package.json` 中：
   ```json
   "dsh": {
     "profile": {
       "bundles": [
         "...",
         "dsh-client-ui-deepseek-bg"
       ]
     }
   }
   ```
4. **重启 DSH Desktop**（客户端插件在启动时组装，修改后必须重启）。
5. 在 **设置 → 外观** 里切换深色 / 浅色即可看到两套效果。

## 验证是否生效

在页面控制台执行：

```js
document.getElementById('dsh-ds-bg')?.dataset.version        // "26"
document.body.hasAttribute('data-dsh-ds-active')             // true
document.querySelector('[data-dsh-ds-frame]')                // 应用外壳节点
getComputedStyle(document.querySelector('[data-dsh-ds-frame]')).backgroundColor // rgba(0, 0, 0, 0)
```

如果 `dataset.version` 是 `undefined`，说明插件没有被加载：优先检查是不是装错了 profile（desktop / web）。

## 实现说明（排查用）

- `lib/client.js` 的客户端入口仍是 DSH 的 `window.__ModuleLoader__.load({ id, factory })` 懒加载格式，`id` 必须等于包名。
- `apply(ctx)` 是标准 Cordis 客户端插件入口，资源通过 `ctx.effect(...)` 注册、卸载时自动清理。
- 外壳适配不再写死类名：
  - 用 `[data-shell-bottom]` 找到布局 frame；
  - 从 `[data-conversation-content]` 反推主列；
  - 给 frame / 侧边栏 / 主列 / 右栏打上 `data-dsh-ds-*` 标记；
  - 颜色、玻璃、模糊全部写进 CSS，只依赖这些标记和 DSH 自己的 `--dsw-*` 语义令牌。
- 与 `dsh-claude-style` 共存：Claude 皮肤会用 `[data-dsh-claude-part~=layout-frame]{background:...!important}`、`#root{background:...}` 和更高特异性的 `--dsw-*` 令牌盖住背景。本版对 frame / center / sidebar / #root 直接写 **内联 `background: transparent !important`**（内联 `!important` 优先级最高），并用 `!important` 覆盖 `--dsw-specific-*` / `--dsw-alias-markdown-code-block` 令牌。
- 浅色鲸鱼：`mix-blend-mode: normal` + 深灰 `uColor` + `uAlphaBoost` 提亮，点尺寸略放大；图层下移 `12vh`、缩小到 `0.86`，作为水印避开正文，深色模式仍保持原来的 `screen` 混合。
- 主题判断沿用 DSH 官方的 `body[data-ds-dark-theme]`；浅色主题走 `body:not([data-ds-dark-theme])` 分支。
- 玻璃颜色优先覆盖 `--dsw-specific-sidebar-fill`、`--dsw-specific-bubble`、`--dsw-specific-input-major`、`--dsw-alias-markdown-code-block` 这几个语义令牌，所以 DSH 升级后不容易再失效。

## 回滚

本目录修改前的完整备份在：

- `..\dsh-client-ui-deepseek-bg.backup-20261101-165427\`（目录）
- `..\dsh-client-ui-deepseek-bg.backup-20261101-165427.zip`（压缩包）

把备份目录覆盖回来，或在插件管理器里卸载后重装旧目录即可。

## 卸载

```bash
dsh plugin --profile desktop remove dsh-client-ui-deepseek-bg
```

再从 `profiles\desktop\package.json` 的 `dsh.profile.bundles` 里删掉对应行，重启 DSH。
