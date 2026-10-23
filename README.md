# 小鲸鱼 · 桌面增强升级包

> 依附 [`dsh-whale-widget`](https://github.com/MeteorNOX/DeepSeek-Balance-Whale-Widget) 运行的桌面增强包。
> **包里不含挂件本体**，必须先装好原插件才能启动。

## 这是什么

原插件 `dsh-whale-widget` 是 DSH（DeepSeek Harness）里的余额挂件。这个升级包把它搬到桌面独立窗口里跑，
并在原插件的基础上加了一层控制中心：

- **控制中心**：一盘管子鲸的大小、位置、透明穿透、气泡、音效、记账、报警
- **多平台记账**：除内置 DeepSeek 账本外，可添加 OpenAI 兼容中转站、各家 API 平台，按余额观测记账
- **用量图表**：按平台切日/周柱状图，DeepSeek 另有官方用量页 / 充值页一键直达
- **额度与报警**：日/月预算、余额不足、超预算通知（点通知直接跳到对应页面）
- **Live2D 形象层**：可换模型、点击动作分组、表情、播放速度、视线跟随
- **点击动效**：按压形变幅度与速度可调
- **形象库直连**：控制中心里点一下形象卡片就真正换过去（含多帧 GIF 待机）——不是只打「使用中」标记

## 包内不含什么（重要）

这个副本是按「可直接分发」清过的，以下内容**全部不在包内**：

- 挂件本体（`whale-widget.js`、`assets/`、`lib/`）——运行时从原插件读取
- Electron 运行时与打包产物 —— 用 `npm install` 自己装依赖，或直接用已有 `electron`
- 任何 API Key / 账号凭据 / 余额账本 / 使用记录 —— 一律只存在本机 `$DSH_HOME`
- 任何截图、调试日志、测试账号、个人路径
- 原插件的副本（`vendor/` 目录整个去掉了）

## 前置要求

1. Node.js 18+
2. 已安装原插件 `dsh-whale-widget`（≥ 0.4.0），常见位置：
   `$HOME/.dsh/profiles/<profile>/node_modules/dsh-whale-widget`
   - 装法：`dsh plugin --profile desktop add dsh-whale-widget`
   - 装在别处也行：启动前设 `WHALE_PLUGIN_DIR=<原插件目录>`

## 安装

```powershell
cd D:\WhaleDesktop-Pack
npm install          # 装 electron / koffi
npm run doctor       # 自检：原插件、镜像、补丁、依赖、端口
npm start            # 启动挂件（托盘图标里可开控制中心）
```

也可以直接跑 `install.ps1`（会依次做上面几步，只做检查不做破坏性操作）。

## 它怎么依附原插件

启动时的顺序：

1. `server/plugin-locator.mjs` 找原插件（`WHALE_PLUGIN_DIR` → `whale-config.json` 的 `pluginDir` → `$DSH_HOME/profiles/*/node_modules` 等常见位置）
2. 把原插件目录**镜像**到 `$DSH_HOME/.whale-pack/mirror-<版本>-<指纹>/`
3. 补丁只打在镜像上：`[standalone-patch]`（跳过 DSH 界面自检）、`[click-tap-patch]`（点击形变可调）
4. `server/host-shim.mjs` 用镜像里的 `lib/index.js` 起本地宿主，26 条 `/dsh-whale/*` 路由由原插件自己注册

所以：**原插件目录永远只读**，原插件升级后镜像会按指纹自动重建；卸载就是删掉这个包和 `.whale-pack`。

## 环境变量

| 变量 | 作用 |
| --- | --- |
| `WHALE_PLUGIN_DIR` | 指定原插件目录（多 profile / 便携安装时用） |
| `WHALE_HOME` | 覆盖 DSH 数据目录（默认 `~/.dsh`） |
| `WHALE_CACHE_DIR` | 覆盖镜像/补丁缓存目录（默认 `$DSH_HOME/.whale-pack`） |
| `WHALE_PORT` | 本地宿主端口（默认 32168） |

## 数据与隐私

- 余额、账本、Live2D 模型、配置全部落在 `$DSH_HOME` 与 Electron 的用户数据目录，不进安装目录。
- 日志写在用户数据目录的 `logs/upgrade.log`，不含密钥。
- 包里没有任何硬编码的密钥或账号，第一次用需要在「账号与余额」里填自己的 API Key。

## 目录

```
main.js               Electron 主进程（窗口 / 托盘 / IPC / 通知）
preload.js            挂件页桥
control.*             控制中心（UI + 渲染逻辑 + 样式）
server/
  plugin-locator.mjs  定位并镜像原插件
  patch-vendor.mjs    对镜像里的挂件脚本打补丁（幂等）
  host-shim.mjs       假宿主：注册原插件的 /dsh-whale/* 路由
  web-balance.mjs     余额来源（本机登录态 / 手动 Key）
  doctor.mjs          自检
web/live2d/           Live2D 渲染层与 pixi / cubism 运行库
```

## 已知边界

- 非 DeepSeek 平台没有「每日历史」接口，多平台图表靠每 5 分钟采样，历史从开始采样的那天往后才有。
- 无余额接口的平台（如部分中转站）只能按会话事件估算，精度低于余额观测式记账。

## 更新记录

- 1.0.1：控制中心点选形象后，主进程把 id 写进挂件窗口的 `localStorage['dshw-role']` 再重载，换形象立即生效（此前只写置顶标记，挂件不跟着换）。
- 1.0.0：首个分发版。
