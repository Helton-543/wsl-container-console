# WSL Manager

Manage WSL containers (`wslc`) inside VS Code: status, start/stop, live logs, port probes and `wslc run` reconstruction — without memorising CLI flags.

[简体中文](#简体中文)

## Why this extension

- **wslc has no GUI.** Container, image, network, volume and session state is scattered across commands; checking one thing costs several invocations.
- **Ports that look published but are dead.** The Windows-side forwarding is proxied and is _silently lost_ after a container is recreated, while `wslc list` still reports the port. Only a real TCP connect tells the truth — that is what the Ports tab does.
- **`wslc restart` always fails** for containers with Windows-path `-v` mounts. The correct fix is "remove, then `wslc run` with the original arguments", and the extension reconstructs that command for you.
- **Logs need a terminal.** Now one click opens a dedicated log window with follow / pause / filter / refresh.

## Install

1. Extensions view → `…` (top right) → **Install from VSIX**
2. Pick `wslc-console-<version>.vsix`
3. Reload the window — a container icon appears in the activity bar

> **Windows only.** `wslc` is the CLI that ships with WSL **on Windows** (2.9.3+); it does not exist on macOS or Linux, so the extension is published as a Windows-only extension. Check your setup with `wslc list`.

## Three-minute tour

1. **Containers** — click the activity bar icon. The view groups containers into Running / Stopped / Not managed by wslc.
2. **Dashboard** — click the dashboard icon in the view title, the "Open dashboard" row at the top of the container view, or the status bar item (`Dashboard 12/12`).
3. **Actions** — each row has Start / Stop / Logs / Details / Remove. While an action runs the row shows "Working…" and locks its buttons.

## What's inside

### Dashboard

The header (running/total, broken ports, CPU, memory, inventories, Top 5 by usage) stays fixed; only the tab content scrolls.

| Tab                                    | Content                                                      |
| -------------------------------------- | ------------------------------------------------------------ |
| Containers                             | State, image, per-port probe dot, row actions                |
| Startup order                          | Drag to reorder, then Start all / Start in order / Stop all  |
| Ports                                  | Real TCP result, latency, binding address per published port |
| Images / Networks / Volumes / Sessions | Inventory lists                                              |
| Events                                 | Live `wslc events` stream                                    |

### Live log window

"Logs" on a container row opens one tab per container:

- **Follow** — new lines scroll into view; turn it off to scroll freely
- **Pause / Resume** — stop updating the view while you read
- **Filter lines** — show only matching lines
- **Refresh** — drop the buffer and re-read the last 200 lines (when a stream looks stuck)
- **Copy all / Clear / Close** — closing the tab stops the underlying `wslc logs -f`
- The status chip shows `Stream stopped (exit N)`; when connected but the container has no output yet it says "no log output available yet"

### Port probes

Every published port gets a real TCP connect:

| Result                         | Meaning                                                     | What to do                                                                                       |
| ------------------------------ | ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| ok                             | Forwarding works                                            | Nothing                                                                                          |
| nothing listening              | No process on the host port                                 | Check the service inside the container; re-publish if the container was recreated                |
| timed out (forwarding problem) | Something listens, but traffic does not reach the container | Typical stale forwarding after a rebuild — recreate the container with the reconstructed command |

Ticking **Only broken ports** switches to the Ports tab automatically and shows "N probed · M broken".

### Start all / Start in order / Stop all

In the **Startup order** tab, drag rows (or use ↑ ↓) to set the order — it is remembered.

- **Start all** — start every _stopped_ managed container, one at a time
- **Start in order** — start them in your order and **wait until each container is actually running** before the next (45 s cap per container)
- **Stop all** — stop every running container, one at a time

The same actions are available from the command palette (`WSLC: Start All Containers`, `WSLC: Start in Saved Order`, `WSLC: Stop All Containers`) and from the title bar of all five views.

### Details and rebuild command

"Details" opens a read-only drawer with image/status/network, the reconstructed **`wslc run` command** (copyable, never contains `--restart` because wslc run has no such option), risk notes (guessed single-file mount path, anonymous volume data will not be reused, truncated multi-part entrypoint, rounded memory limit) and the raw `wslc inspect` output.

When `wslc restart` fails because of Windows-path mounts, the correct flow is: copy the rebuild command → remove the container → run the command.

### Also

- **Events** — live `wslc events` in the dashboard, reconnects automatically
- **Guarded destructive actions** — removing a container/image or terminating a session requires typing the object name
- **Status bar** — running/total, with a warning icon when ports are broken; click to open the dashboard

### Deploy from a file

Open the **Deploy** tab (or run `WSLC: Deploy Project from File`). The extension scans the workspace for a `Dockerfile`, a `docker-compose.yml` / `compose.yml` or a `*.wslc.json` manifest, and you can also pick any file yourself.

One click runs the whole chain per service — **build → remove the same-named container → run → probe the published ports** — and streams each phase into the panel. Deploy profiles are saved to `.wslc/deploy.json` in the workspace, so they can be committed and reused on another machine.

```text
Deploy
├── build       wslc build <context> -t <tag>
├── remove      wslc rm -f <container>        (optional, on by default)
├── run         wslc run -d --name … -p … -v … -e … <image>
└── probe       real TCP connect to every published host port
```

Supported compose syntax (all verified against real containers):

| Area      | What works                                                                                                                                                              |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Structure | anchors `&a` / aliases `*a`, merge keys `<<: *a`, block scalars `                                                                                                       | ` `>`(with`-`/`+`), comments                                 |
| Services  | `image`, `container_name`, `build` (string or `{context, dockerfile, target, args}`), `entrypoint` (string → `sh -c`), `command`, `user`, `working_dir`, `network_mode` |
| Ports     | `"8080:80"`, `"127.0.0.1:8080:80"`, `"53:53/udp"`, and the long form `{target, published, protocol, host_ip}`                                                           |
| Config    | `environment` (list or map), `env_file`, `volumes` (short **and** long with `type: bind                                                                                 | volume`), `depends_on`(start order),`networks` (list or map) |
| Variables | `${VAR}`, `${VAR:-default}`, `${VAR:?required}`, `${VAR:+alt}`, `$$` — values come from `.env` → `.env.prd` → `.env.dev` plus the process environment                   |

Only inline mappings (`{a: b}` on one line) are rejected — with a line-numbered error rather than a silently wrong plan.

Two safety notes:

- **Secrets never reach the profile file.** Interpolated values whose key looks like a secret (`PASSWORD` / `SECRET` / `TOKEN` / `CREDENTIAL` / `*_KEY`) are stored empty in `.wslc/deploy.json` and shown as `••••••`; before deploying, the file is re-parsed with your `.env` so the real value is used from memory only.
- **Re-deploying re-reads the file**, so edits to the compose file take effect without re-importing.

You can also right-click a `docker-compose.yml` / `Dockerfile` / `*.wslc.json` in the Explorer and pick **Deploy This File to wslc**.

```json
{
  "name": "demo",
  "services": [
    {
      "name": "web",
      "image": "demo-web:latest",
      "build": { "context": ".", "dockerfile": "Dockerfile", "tag": "demo-web:latest" },
      "ports": [{ "host": 8080, "container": 80 }],
      "env": { "ASPNETCORE_URLS": "http://+:80" },
      "volumes": [{ "source": "./data", "target": "/app/data", "readOnly": false }],
      "command": [],
      "dependsOn": []
    }
  ]
}
```

> Deploying removes containers with the same name first (that checkbox can be turned off). Data on Windows paths stays on disk; anything written inside the container is lost.

## UI language

Follows the VS Code display language by default; Chinese and English are supported.

- Switch the extension UI: command palette → `WSLC: Switch Display Language` (takes effect immediately for everything the extension draws)
- Command titles and view names are read by VS Code at activation — reload the window after changing the language
- Or pin it with the `wslcConsole.locale` setting (`auto` / `en` / `zh-cn`)

## Settings

| Setting                            | Default | Purpose                                                    |
| ---------------------------------- | ------- | ---------------------------------------------------------- |
| `wslcConsole.executable`           | `wslc`  | Path to the wslc binary                                    |
| `wslcConsole.session`              | empty   | Value passed to `--session` (empty = default session)      |
| `wslcConsole.locale`               | `auto`  | UI language: follow VS Code / English / Simplified Chinese |
| `wslcConsole.polling.containersMs` | `5000`  | Foreground refresh interval (background is 6×)             |
| `wslcConsole.polling.statsMs`      | `15000` | Resource usage interval (`wslc stats` itself takes ~7 s)   |
| `wslcConsole.timeouts.stopMs`      | `25000` | `wslc stop` watchdog; the process is killed after this     |
| `wslcConsole.portProbe.enabled`    | `true`  | Probe published ports with a real TCP connect              |

## FAQ

**The log window keeps saying "no log output available yet".**
`wslc logs` genuinely has nothing for that container (common right after creation). Press Refresh; new output appears automatically once the container writes something.

**Why can't some containers be started or stopped?**
They were created with `docker run` inside a wslc session, so they have no wslc metadata label and wslc cannot manage them. They are marked "not managed by wslc" and their actions are disabled.

**Stopping feels slow.**
`wslc stop` hangs forever on already-stopped containers, so every command has a watchdog (25 s by default). A "timed out" result usually means the container was already stopped.

**A port shows "timed out (forwarding problem)".**
Windows-side forwarding is lost after a container is recreated while the port list still shows it as published. Recreate the container with the rebuilt command from Details.

**I changed the language but command titles stayed.**
VS Code reads titles/view names at activation — reload the window. Everything the extension renders itself changes immediately.

**Why is CPU sometimes above 100%?**
`wslc stats` reports per-core percentages, so a multi-threaded container can show 400%. The dashboard normalises the total by logical cores and prints the raw sum underneath.

## Known limitations

- Activity bar icons are masked by some themes (colour is dropped); shapes remain recognisable
- Traditional Chinese (zh-TW / zh-HK) currently falls back to Simplified Chinese
- Port probing only covers running containers and requires `wslcConsole.portProbe.enabled`

## Sponsor

<!-- sponsor:qr:start -->

|                            WeChat reward code                             |                            Alipay payment code                             |
| :-----------------------------------------------------------------------: | :------------------------------------------------------------------------: |
| <img src="/sponsor/wechat.png" width="230" alt="WeChat reward code"> | <img src="/sponsor/alipay.png" width="230" alt="Alipay payment code"> |

<!-- sponsor:qr:end -->

**Outside mainland China** — Alipay and WeChat Pay both accept foreign cards in their international apps, so you can scan the codes above directly. Prefer a card / PayPal checkout? Use the link below:

<!-- sponsor:overseas:start -->

**[Ko-fi →](https://ko-fi.com/helton70470)** · **[PayPal.Me →](https://paypal.me/HeltonLHD)**
<!-- sponsor:overseas:end -->

Inside VS Code: `WSLC: Sponsor / Donate` (command palette, or the ♥ button in the sidebar title bar).

---

## Feedback

When reporting an issue, include the container name, what you did, and the related lines from the Output panel (`WSLC: Show Logs`).

---

## 简体中文

在 VS Code 里管理 WSL 容器（`wslc`）：看状态、启停、翻日志、查端口、还原启动命令——不用再记一长串命令行。

### 这个扩展解决什么

- **wslc 没有图形界面**：容器、镜像、网络、卷、会话分散在命令里，看一眼要点好几次
- **端口"以为通、其实不通"**：容器重建后 Windows 侧转发会丢，但 `wslc list` 仍显示端口已发布；这里对每个宿主端口做真实 TCP 连接来区分
- **`wslc restart` 对带 Windows 路径挂载的容器必失败**：正确做法是删掉再用原参数 `wslc run`，扩展能把那条命令还原出来
- **日志要开终端敲命令**：现在点一下就开独立窗口，边看边过滤

### 安装

1. 扩展视图 → 右上角 `…` → **从 VSIX 安装**
2. 选择 `wslc-console-<版本>.vsix`
3. 重载窗口，左侧活动栏出现集装箱图标

> **仅支持 Windows。** `wslc` 是 **Windows 版** WSL 自带的命令行（2.9.3+），macOS / Linux 上并不存在，所以本扩展在商店里标记为 Windows 专用。验证：终端执行 `wslc list`。

### 三分钟上手

1. **看容器**：点活动栏图标 → 「容器」按「运行中 / 已停止 / 非 wslc 托管」分组
2. **打开仪表盘**：视图顶部仪表盘图标、容器视图首行「打开仪表盘」、或状态栏 `仪表盘 12/12`
3. **启停**：每行右侧有 启动 / 停止 / 日志 / 详情 / 删除；操作中该行显示「处理中…」并锁住按钮

### 功能

**仪表盘**：顶部（运行数、端口异常、CPU 整机占用、内存、清单计数、占用 Top 5）固定，只有标签内容区滚动。

| 标签                    | 内容                                        |
| ----------------------- | ------------------------------------------- |
| 容器                    | 状态、镜像、端口探活小圆点、行内操作        |
| 启动顺序                | 拖拽排序 + 全部启动 / 按顺序启动 / 全部停止 |
| 端口                    | 每个已发布端口的实连结果、耗时、绑定地址    |
| 镜像 / 网络 / 卷 / 会话 | 清单                                        |
| 事件                    | `wslc events` 实时流                        |

**实时日志窗口**（一容器一页签）：跟随、暂停/继续、过滤行、刷新（重拉最近 200 行）、复制全部、清空、关闭（关闭即结束背后的 `wslc logs -f`）。流结束显示 `流已结束（exit N）`；连接正常但暂无输出显示「已连接，暂无可显示的日志」。

**端口探活**：`通` / `无监听` / `超时（转发异常）`。勾选「只看异常端口」会自动切到端口标签，并显示「已探活 N 个 · 异常 M 个」。

**一键启动 / 按顺序启动 / 全部停止**：「启动顺序」标签里拖动排序（顺序会记住）；「按顺序启动」会等上一个容器真正进入运行中再启动下一个（每个最多 45 秒）。命令面板与五个视图标题栏都有入口。

**详情（只读）**：镜像/状态/网络、还原出的 `wslc run` 重建命令（可复制，不含 `--restart`）、风险提示、`wslc inspect` 原文。`wslc restart` 失败时的正确流程：复制重建命令 → 删除容器 → 执行命令。

### 根据文件部署

打开仪表盘的**部署**标签页（或执行 `WSLC: 根据文件部署项目`）。扩展会自动扫描工作区里的 `Dockerfile`、`docker-compose.yml` / `compose.yml` 或 `*.wslc.json` 清单，也可以手动选任意文件。

点一下「部署」就按服务依次走完：**构建镜像 → 删除同名容器 → 启动 → 端口实连探活**，每个阶段实时显示在面板上。部署档案保存在工作区的 `.wslc/deploy.json`，可以提交进仓库、换机器复用。

```text
部署
├── 构建      wslc build <上下文> -t <标签>
├── 删除      wslc rm -f <容器>              （可选，默认开启）
├── 启动      wslc run -d --name … -p … -v … -e … <镜像>
└── 探活      对每个宿主端口做一次真实 TCP 连接
```

支持的 compose 写法（都已在真实容器上验证过）：

| 类别 | 支持内容                                                                                                                                                                     |
| ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 结构 | 锚点 `&a` / 别名 `*a`、合并键 `<<: *a`、块标量 `\|` `>`（含 `-`/`+`）、注释                                                                                                  |
| 服务 | `image`、`container_name`、`build`（字符串或 `{context, dockerfile, target, args}`）、`entrypoint`（字符串自动转 `sh -c`）、`command`、`user`、`working_dir`、`network_mode` |
| 端口 | `"8080:80"`、`"127.0.0.1:8080:80"`、`"53:53/udp"`，以及长语法 `{target, published, protocol, host_ip}`                                                                       |
| 配置 | `environment`（列表或映射）、`env_file`、`volumes`（短语法与长语法 `type: bind\|volume`）、`depends_on`（启动顺序）、`networks`（列表或映射）                                |
| 变量 | `${VAR}`、`${VAR:-默认}`、`${VAR:?必填}`、`${VAR:+替代}`、`$$`——取值来自 `.env` → `.env.prd` → `.env.dev`，再叠加进程环境                                                    |

只有行内映射（同一行写 `{a: b}`）不支持——会给带行号的报错，而不是一份「看着像成功」的错误编排。

两点安全设计：

- **凭据不落盘**：插值后的值若键名像密钥（`PASSWORD` / `SECRET` / `TOKEN` / `CREDENTIAL` / `*_KEY`），写入 `.wslc/deploy.json` 时为空串，界面显示 `••••••`；部署前会按源文件 + `.env` 重新解析，真实值只在内存里用过一次。
- **重新部署会重读文件**，改完 compose 不用重新导入。

也可以在资源管理器里右键 `docker-compose.yml` / `Dockerfile` / `*.wslc.json`，选 **部署此文件到 wslc**。

```json
{
  "name": "demo",
  "services": [
    {
      "name": "web",
      "image": "demo-web:latest",
      "build": { "context": ".", "dockerfile": "Dockerfile", "tag": "demo-web:latest" },
      "ports": [{ "host": 8080, "container": 80 }],
      "env": { "ASPNETCORE_URLS": "http://+:80" },
      "volumes": [{ "source": "./data", "target": "/app/data", "readOnly": false }],
      "command": [],
      "dependsOn": []
    }
  ]
}
```

> 部署会先删除同名容器（该勾选项可关闭）。Windows 路径上的数据保留在磁盘；容器内部写入的数据会丢失。

### 界面语言

默认跟随 VS Code 显示语言，支持中英文。切换：命令面板 `WSLC: 切换显示语言`（扩展自己画的界面立刻生效）；命令标题与视图名由 VS Code 在激活时读取，**需重载窗口**；也可用设置 `wslcConsole.locale` 固定。

### 设置项

| 设置                               | 默认    | 说明                                         |
| ---------------------------------- | ------- | -------------------------------------------- |
| `wslcConsole.executable`           | `wslc`  | wslc 可执行文件路径                          |
| `wslcConsole.session`              | 空      | 传给 wslc 的 `--session`                     |
| `wslcConsole.locale`               | `auto`  | 界面语言：跟随 VS Code / 英文 / 简体中文     |
| `wslcConsole.polling.containersMs` | `5000`  | 前台刷新间隔（后台为其 6 倍）                |
| `wslcConsole.polling.statsMs`      | `15000` | 资源占用刷新间隔（`wslc stats` 单次约 7 秒） |
| `wslcConsole.timeouts.stopMs`      | `25000` | `wslc stop` 看门狗，超时强制结束             |
| `wslcConsole.portProbe.enabled`    | `true`  | 是否对已发布端口做 TCP 实连探活              |

### 常见问题

**日志一直「暂无可显示的日志」？** 说明容器确实还没有输出（常见于刚创建）。点「刷新」重拉，容器产生新输出后会自动出现。

**有些容器不能启停？** 它们在 wslc 会话里用 `docker run` 创建，没有 wslc 元数据标签，wslc 管不了；行内标「非 wslc 托管」，按钮禁用。

**停止很慢？** `wslc stop` 对已停止容器会挂死，所以每条命令带看门狗（默认 25 秒后强制结束）。看到超时通常表示容器本来就已经停了。

**端口显示「超时（转发异常）」？** 容器重建后 Windows 侧转发丢失，但列表仍显示已发布；用「详情」里的重建命令重新创建容器。

**改了语言但命令标题没变？** 命令标题/视图名由 VS Code 激活时读取，需重载窗口；扩展自己渲染的部分立即生效。

**CPU 为什么超过 100%？** `wslc stats` 是"占单核"百分比，多线程容器可到 400%；仪表盘合计已按逻辑核折算成整机占比，并在下方标注原始合计。

### 已知限制

- 部分主题会把活动栏图标按遮罩渲染（丢色），形状仍可辨认
- 繁体中文（zh-TW / zh-HK）暂回退到简体
- 端口探活只覆盖运行中的容器，且需要 `wslcConsole.portProbe.enabled` 开启

## 赞助

<!-- sponsor:qr:start -->

|                            微信赞赏码                             |                            支付宝收款码                             |
| :---------------------------------------------------------------: | :-----------------------------------------------------------------: |
| <img src="/sponsor/wechat.png" width="230" alt="微信赞赏码"> | <img src="/sponsor/alipay.png" width="230" alt="支付宝收款码"> |

<!-- sponsor:qr:end -->

**中国大陆以外** —— 支付宝 / 微信的国际版都支持绑定外卡扫码，可以直接扫上面两张；想走信用卡 / PayPal 结账，用下面的链接：

<!-- sponsor:overseas:start -->

**[Ko-fi →](https://ko-fi.com/helton70470)** · **[PayPal.Me →](https://paypal.me/HeltonLHD)**
<!-- sponsor:overseas:end -->

编辑器内也可用：`WSLC: 赞助作者`（命令面板，或侧边栏标题栏的 ♥ 按钮）。

---

### 反馈

请附上容器名、操作步骤，以及 Output 面板（`WSLC: 显示日志`）里的相关行。
