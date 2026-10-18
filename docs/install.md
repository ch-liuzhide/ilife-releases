# 安装说明

## 系统要求

| 系统 | 要求 | 安装包 |
|---|---|---|
| macOS | 13（Ventura）及更新；Apple 芯片与 Intel 芯片各一个包 | [iLife-mac-arm64.dmg](https://github.com/ch-liuzhide/ilife-releases/releases/latest/download/iLife-mac-arm64.dmg) / [iLife-mac-x64.dmg](https://github.com/ch-liuzhide/ilife-releases/releases/latest/download/iLife-mac-x64.dmg) |
| Windows | 10 / 11，64 位 | [iLife-win-x64.exe](https://github.com/ch-liuzhide/ilife-releases/releases/latest/download/iLife-win-x64.exe) |

- 不确定 Mac 是哪种芯片：苹果菜单 →「关于本机」，显示「芯片 Apple M…」选 arm64，显示「处理器 … Intel …」选 x64。
- macOS 的 `.zip` 是 dmg 的替代格式，解压得到 `iLife.app`，用法相同。
- 链接始终指向最新的正式版；[Releases](https://github.com/ch-liuzhide/ilife-releases/releases) 里同时有带版本号的同一份文件（例如 `iLife-0.3.0-mac-arm64.dmg`）。
- 命令行下载：

  ```bash
  curl -LO https://github.com/ch-liuzhide/ilife-releases/releases/latest/download/iLife-mac-arm64.dmg
  ```

> 安装包**没有开发者证书签名，也没有经过 Apple 公证**（macOS 只做了 ad-hoc 签名），所以首次打开时系统会拦截一次，按下面的步骤放行即可。
> 请只从本仓库的 Releases 页面下载。

## macOS：安装与首次打开

1. 打开 dmg，把 **iLife** 拖到「应用程序」文件夹（升级时选择「替换」）。
2. 首次打开，任选一种方式放行（只需一次）：
   - **macOS 15（Sequoia）及更新**：双击 iLife，提示「无法验证……」时点「完成」；打开「系统设置 → 隐私与安全性」，
     在页面下方「安全性」处找到「已阻止"iLife"……」，点「**仍要打开**」，输入登录密码确认，再点「打开」。
   - **macOS 14 及更早**：在「应用程序」里**按住 Control 点按（或右键）iLife → 打开**，在弹窗里再点「打开」。
   - **终端**（任何版本）：去掉下载隔离标记后即可正常双击打开：

     ```bash
     xattr -dr com.apple.quarantine /Applications/iLife.app
     ```

3. 如果提示「"iLife"已损坏，无法打开」（某些下载 / 解压工具会这样），同样执行上面的 `xattr` 命令。

### 电脑操作需要的系统权限

**电脑操作默认关闭**，不用它就不需要下面的权限。需要时在 iLife「设置 → 能力 → 电脑操作」打开「允许智能体操作电脑」并确认，
再按下表授权。「设置 → 能力」里能看到授权状态，并能一键打开对应的系统设置页。

| 权限 | 位置 | 用途 |
|---|---|---|
| 屏幕录制 | 系统设置 → 隐私与安全性 → 屏幕与系统录音（旧版本叫「屏幕录制」） | 截屏，让智能体看到桌面；授权后需重启 iLife |
| 辅助功能 | 系统设置 → 隐私与安全性 → 辅助功能 | 模拟鼠标、键盘 |
| 自动化 | 首次操作时系统弹窗询问 | 允许 iLife 控制「System Events」 |

- **关闭「允许智能体操作电脑」不会撤销这些系统授权**，只是不再向智能体提供电脑操作工具。不再需要时，请在上表的系统设置页里关掉 iLife。
- 智能体在终端里执行的命令以「**iLife Helper**」的身份运行，不沿用你授予 iLife 的辅助功能授权，系统设置里不需要给「iLife Helper」授权。
  例外是屏幕录制：macOS 按整个 iLife.app 判断，授权过屏幕录制后，智能体执行的命令也可能截屏。
- 由于没有开发者证书，**每次升级到新版本后，macOS 会把它当作另一个应用**：之前打开的开关可能显示为已允许但不生效。
  此时在上述列表中选中 iLife 点「−」删除后重新添加（或重新打开开关），也可以在终端重置后重新授权：

  ```bash
  tccutil reset ScreenCapture com.ilife.desktop
  tccutil reset Accessibility com.ilife.desktop
  tccutil reset AppleEvents com.ilife.desktop
  ```

iLife 在 macOS 上不使用钥匙串保存 API Key（配置文件以仅本人可读的权限保存），升级后不会出现钥匙串授权弹窗。

## Windows：安装与首次打开

1. 浏览器提示「不常下载」或「可能有害」时，选择「保留」。
2. 运行下载的 `iLife-win-x64.exe`（从 Releases 页面手动下载的是带版本号的 `iLife-<版本>-win-x64.exe`，同一个安装包）。
   出现 SmartScreen「Windows 已保护你的电脑」时，点「**更多信息**」→「**仍要运行**」。
3. 安装向导可以修改安装目录（默认按当前用户安装到 `%LOCALAPPDATA%\Programs\iLife`，不需要管理员权限），完成后桌面与开始菜单有 iLife 快捷方式。
4. 个别杀毒软件会拦截未签名的程序，把 iLife 安装目录加入信任即可。

- 电脑操作同样默认关闭（「设置 → 能力」里开启）。它在 Windows 上没有系统授权弹窗把关，开启前请确认你信任任务内容；
  普通权限运行的 iLife 无法操作「以管理员身份运行」的程序窗口（系统限制）。
- Windows 上「工作区可写」档位只限制智能体的写文件工具（只能写工作目录与临时目录），**它运行的命令没有系统沙箱**，
  与你自己在终端里运行的命令权限相同；对任务内容没把握时用「只读」档位。
- Windows 版用系统密钥（DPAPI）加密保存模型 API Key 与内置浏览器的 Cookie，不需要任何操作。

## 数据、升级与卸载

| 内容 | macOS | Windows |
|---|---|---|
| 数据目录（配置、会话、插件、日志） | `~/Library/Application Support/iLife` | `%APPDATA%\iLife` |
| 日志 | `<数据目录>/logs/main.log` | 同左 |
| 默认工作目录（智能体读写文件的位置，可在设置里修改） | `~/Documents/iLife` | `文档\iLife` |

### 升级

没有自动更新。下载新版本的安装包覆盖安装即可：macOS 拖入「应用程序」选择「替换」，Windows 直接运行新安装包。数据目录保留。

- macOS 上开启过电脑操作的，升级后需要按上文重新授权屏幕录制 / 辅助功能。
- 从 0.2.0 升级到 0.3.0：智能体底座更换，0.2.0 的对话记录不会迁移；模型服务等设置请在升级后检查一遍（见[更新说明](../CHANGELOG.md)）。

### 卸载

macOS 把 iLife 移到废纸篓；Windows 在「设置 → 应用」中卸载。数据目录不会自动删除，需要时手动删除。
