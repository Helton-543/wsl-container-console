WSL Manager（便携版）
=====================

版本：0.10.1

运行方式
--------
1. 解压到任意目录（保持 WSL Manager.exe 与 WebView2Loader.dll 在同一目录）；
2. 双击 WSL Manager.exe 即可，无需安装、无需管理员权限。

运行要求
--------
- Windows 10 1809+ / Windows 11；
- WebView2 Runtime（Win11 自带；Win10 若缺，装一次 Edge WebView2 即可，微软常青更新）；
- 管理对象：WSL 2 发行版 与 wslc 容器/镜像（都需要在本机已安装）。

数据位置
--------
设置与状态保存在 %APPDATA%\wslc-console\，删除该目录即完全重置。

常见问题
--------
- 提示「Windows 已保护你的电脑」：这是未签名程序的 SmartScreen 提示，
  点「更多信息 → 仍要运行」；发布物校验和见同目录 SHA256SUMS.txt。
- 找不到 WebView2Loader.dll：它必须和 exe 在同一目录，请重新解压完整压缩包。
