# WSL Manager

One console for **WSL distributions** and **wslc containers** — available as a **VS Code extension** and a **Windows desktop app**.

[Download the desktop app](https://github.com/Helton-543/wsl-container-console/blob/master/WSL%20Manager%20x64-setup.exe) · [GitHub](https://github.com/Helton-543/wsl-container-console/tree/master) · [VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=HeltonLHD.wslc-console)

> **Windows only.** WSL and wslc are Windows features. The desktop app runs on Windows 10/11 x64.

## What you can do

- **Distributions** — see state, version and resource usage; start/stop, set default, switch version, install, export/import backup, uninstall.
- **Files** — browse any distribution, upload (drag and drop in the desktop app), download, rename, delete, open in Explorer or a terminal.
- **Embedded desktop** — start a Linux desktop in a window with one click, no extra setup. Only listens on `127.0.0.1` and uses a random password.
- **Containers** — list, start, stop, restart, remove, open logs, inspect details, and copy a rebuild command.
- **Images** — single or batch delete, with a typed `DELETE` confirmation.
- **Ports** — real connectivity checks instead of trusting “published” state; stopped containers show “not running”.
- **Startup order** — drag to reorder, then start all, start in order, or stop all.
- **Networks, volumes, sessions, events** — inventory and live event stream.
- **Settings** — edit `.wslconfig` and `/etc/wsl.conf` with forms or raw text; app settings for refresh interval, desktop resolution and port.
- **Deploy from a file** _(VS Code extension only)_ — scan a `Dockerfile`, `docker-compose.yml` / `compose.yml` or `*.wslc.json`, then build → replace → run → verify ports.

## Install

### VS Code extension

1. Open **Extensions** and search for **WSL Manager**.
2. Or use **Install from VSIX** and pick `wslc-console-<version>.vsix`.
3. Reload the window; the container icon appears in the activity bar.

### Desktop app

1. Download [`WSL Manager x64-setup.exe`](https://github.com/Helton-543/wsl-container-console/blob/master/WSL%20Manager%20x64-setup.exe).
2. Run the installer. WebView2 and WSL 2 are required; the installer can fetch WebView2 when online.
3. Launch **WSL Manager** from the Start menu.

## Desktop and VS Code

|                                                  | Desktop app | VS Code extension |
| ------------------------------------------------ | ----------- | ----------------- |
| Distributions, files, embedded desktop, settings | ✓           | ✓                 |
| Containers, images, ports, startup order, logs   | ✓           | ✓                 |
| Deploy from Dockerfile / compose                 | —           | ✓                 |
| Shared look and behaviour                        | ✓           | ✓                 |

## Quick start

1. Open **Overview** and check that your distribution is running.
2. Use **Files** to browse the Linux filesystem, or **Desktop** to open a graphical desktop.
3. For containers, open the **Containers** page; logs and details are one click away.

## Safety

- Removing a container requires typing its name; removing images requires typing `DELETE`.
- The embedded desktop only accepts connections from the local machine and uses a random password per session.
- Port checks only run for running containers.

## Settings

| Setting                   | Purpose                                                     |
| ------------------------- | ----------------------------------------------------------- |
| Display language          | Follow VS Code, English or Simplified Chinese (extension)   |
| wslc executable / session | Path to `wslc` and optional session name (extension)        |
| Refresh intervals         | How often containers and resource usage refresh (extension) |
| Port checks               | Enable real connectivity checks (extension)                 |
| Desktop resolution / port | Defaults for the next desktop session (desktop app)         |

## FAQ

**A log window says there is no output.** The container has not written anything yet. Refresh; new lines appear automatically.

**Some containers cannot be started or stopped.** They were created outside wslc, so wslc does not manage them. The row is marked “not managed by wslc”.

**A port shows “timed out”.** The Windows-side forwarding was lost after the container was recreated. Recreate it with the rebuild command from **Details**.

**CPU shows more than 100%.** Resource usage is measured per core, so a multi-threaded container can exceed 100%; the total is normalised by logical cores.

**I changed the language but command titles did not change.** VS Code reads titles at activation; reload the window. Everything the extension draws changes immediately.

## Known limitations

- Windows only.
- Containers created outside wslc cannot be managed.
- Traditional Chinese falls back to Simplified Chinese.
- Port checks cover running containers only.

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

## Feedback

When reporting an issue, include the container name, what you did, and the related lines from the Output panel (`WSLC: Show Logs`).

---

## 简体中文

一个界面管理 **WSL 发行版**和 **wslc 容器**——同时提供 **VS Code 扩展**和 **Windows 桌面版**。

[下载桌面版](https://github.com/Helton-543/wsl-container-console/blob/master/WSL%20Manager%20x64-setup.exe) · [GitHub](https://github.com/Helton-543/wsl-container-console/tree/master) · [VS Code 扩展商店](https://marketplace.visualstudio.com/items?itemName=HeltonLHD.wslc-console)

> **仅支持 Windows。** WSL 与 wslc 都是 Windows 功能；桌面版支持 Windows 10/11 x64。

### 能做什么

- **发行版**：查看状态、版本与资源占用；启动/停止、设为默认、切换版本、安装、导出/导入备份、注销。
- **文件**：浏览任意发行版的文件系统，上传（桌面版支持拖拽）、下载、重命名、删除、在资源管理器或终端中打开。
- **内嵌桌面**：一键打开 Linux 图形桌面，无需额外安装；只监听 `127.0.0.1`，每次会话随机密码。
- **容器**：列表、启动、停止、重启、删除、查看日志、查看详情、复制重建命令。
- **镜像**：单个或批量删除，需输入 `DELETE` 确认。
- **端口**：做真实连接检查，而不是只看“已发布”；已停止的容器显示“未启动”。
- **启动顺序**：拖拽排序，支持全部启动、按顺序启动、全部停止。
- **网络 / 卷 / 会话 / 事件**：清单与实时事件流。
- **设置**：表单或原始文本编辑 `.wslconfig` 与 `/etc/wsl.conf`；应用设置可调刷新间隔、桌面分辨率与端口。
- **根据文件部署**（仅 VS Code 扩展）：扫描 `Dockerfile`、`docker-compose.yml` / `compose.yml` 或 `*.wslc.json`，依次构建 → 替换 → 启动 → 端口检查。

### 安装

#### VS Code 扩展

1. 打开 **扩展**，搜索 **WSL Manager**。
2. 或选择 **从 VSIX 安装**，选中 `wslc-console-<版本>.vsix`。
3. 重载窗口，左侧活动栏出现容器图标。

#### 桌面版

1. 下载 [`WSL Manager x64-setup.exe`](https://github.com/Helton-543/wsl-container-console/blob/master/WSL%20Manager%20x64-setup.exe)。
2. 运行安装包。需要 WebView2 与 WSL 2；联网时安装包可自动获取 WebView2。
3. 从开始菜单启动 **WSL Manager**。

### 桌面版与 VS Code 扩展

|                                  | 桌面版 | VS Code 扩展 |
| -------------------------------- | ------ | ------------ |
| 发行版、文件、内嵌桌面、设置     | ✓      | ✓            |
| 容器、镜像、端口、启动顺序、日志 | ✓      | ✓            |
| 根据 Dockerfile / compose 部署   | —      | ✓            |
| 相同界面与操作方式               | ✓      | ✓            |

### 快速上手

1. 打开 **总览**，确认发行版正在运行。
2. 用 **文件** 浏览 Linux 文件系统，或用 **桌面** 打开图形桌面。
3. 管理容器时打开 **容器** 页，日志和详情都在行内操作里。

### 安全说明

- 删除容器需输入容器名；删除镜像需输入 `DELETE`。
- 内嵌桌面只接受本机连接，每次会话使用随机密码。
- 端口检查只针对运行中的容器。

### 设置

| 设置              | 说明                                      |
| ----------------- | ----------------------------------------- |
| 显示语言          | 跟随 VS Code、英文或简体中文（扩展）      |
| wslc 路径 / 会话  | `wslc` 可执行文件路径与可选会话名（扩展） |
| 刷新间隔          | 容器与资源占用的刷新频率（扩展）          |
| 端口检查          | 是否启用真实连接检查（扩展）              |
| 桌面分辨率 / 端口 | 下一次桌面会话使用的默认值（桌面版）      |

### 常见问题

**日志窗口显示暂无输出？** 容器还没有写入日志；点刷新，产生新输出后会自动出现。

**有些容器不能启停？** 它们是在 wslc 之外创建的，wslc 无法管理；行内会标记“非 wslc 托管”。

**端口显示“超时”？** 容器重建后 Windows 侧转发会丢失；用“详情”里的重建命令重新创建容器。

**CPU 为什么超过 100%？** 资源占用按单核统计，多线程容器会超过 100%；界面已按逻辑核折算。

**改了语言但命令标题没变？** VS Code 在激活时读取命令标题，需重载窗口；扩展自己绘制的界面会立即更新。

### 已知限制

- 仅支持 Windows。
- 在 wslc 之外创建的容器无法管理。
- 繁体中文暂回退到简体中文。
- 端口检查只覆盖运行中的容器。

### 赞助

<!-- sponsor:qr:start -->

|                            微信赞赏码                             |                            支付宝收款码                             |
| :---------------------------------------------------------------: | :-----------------------------------------------------------------: |
| <img src="/sponsor/wechat.png" width="230" alt="微信赞赏码"> | <img src="/sponsor/alipay.png" width="230" alt="支付宝收款码"> |

<!-- sponsor:qr:end -->

**中国大陆以外** —— 支付宝 / 微信的国际版支持绑定外卡扫码；也可以走信用卡 / PayPal：

<!-- sponsor:overseas:start -->

**[Ko-fi →](https://ko-fi.com/helton70470)** · **[PayPal.Me →](https://paypal.me/HeltonLHD)**
<!-- sponsor:overseas:end -->

编辑器内也可用：`WSLC: 赞助作者`（命令面板，或侧边栏标题栏的 ♥ 按钮）。

### 反馈

请附上容器名、操作步骤，以及 Output 面板（`WSLC: 显示日志`）里的相关行。
