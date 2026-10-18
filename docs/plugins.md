# 团队插件格式

给团队管理员与插件作者：团队下发给成员的插件（服务端叫「模块」）是一个 **zip 包**，根目录有清单 `ilife.module.json`。
桌面端只执行三类内容：**技能**、**MCP 服务**、**提示词**；清单里的其它贡献类型在桌面端显示为「本客户端不支持」，不会执行。

## 包结构

```
my-plugin/
├── ilife.module.json        # 必需：清单
├── skills/                  # 可选：技能，每个子目录一个 SKILL.md
│   └── my-skill/SKILL.md
├── prompts/                 # 可选：提示词片段（Markdown）
│   └── style.md
└── mcp/server.mjs           # 可选：MCP 服务（stdio）
```

zip 根目录可以直接是这些文件，也可以只有一个顶层目录（上传时自动剥离）。脚本请用 `.mjs` 后缀（解压后的目录没有 package.json）。

## 清单 `ilife.module.json`

| 字段 | 必填 | 说明 |
|---|---|---|
| `id` | ✔ | 小写字母、数字、`. _ -`，2～64 位，团队内唯一 |
| `name` | ✔ | 显示名 |
| `version` | ✔ | semver，如 `1.2.0`；同一插件的版本不可重复（「本地」状态可覆盖） |
| `description` / `author` | | 说明与作者 |
| `targets` | | `["client"]`（默认，桌面端） |
| `clientVersionRange` | | 默认绑定的客户端版本范围（semver range，如 `>=0.3.0`），后台可改 |
| `manifestVersion` | | 用 `prompts` 时写 `2` |
| `contributes.skills` | | 技能目录：`"skills"`，或 `{ "dir": "skills" }`。其下每个 `<name>/SKILL.md` 是一个技能，YAML 前言要有合规的 `name` 与 `description` |
| `contributes.mcpServers[]` | | `{ "name", "command", "args"?, "env"?, "cwd"? }`。`command: "node"` 使用 iLife 自带的 Node（推荐，三个平台一致）；其它命令不经 shell 启动，Windows 上须是真正的可执行文件（`npx`、`*.cmd` 这类垫片无法启动）；参数里的 `${moduleDir}` 替换为插件安装目录。工具以 `mcp__<name>__<tool>` 出现 |
| `contributes.prompts[]` | | `{ "id", "file" 或 "text", "into"?, "position"? }`，拼进智能体的系统提示词。`into` 是注入目标数组，桌面端只有默认助手，写 `["@default"]` 或省略；`position` 为 `"prefix"` 或 `"suffix"` |

示例：一个技能 + 一个 MCP 服务 + 一段提示词。

```json
{
  "manifestVersion": 2,
  "id": "acme-tools",
  "name": "Acme 工具包",
  "version": "1.0.0",
  "description": "Acme 团队的报表技能、内部系统 MCP 与写作规范",
  "author": "Acme",
  "targets": ["client"],
  "clientVersionRange": ">=0.3.0",
  "contributes": {
    "skills": "skills",
    "mcpServers": [
      { "name": "acme", "command": "node", "args": ["${moduleDir}/mcp/server.mjs"] }
    ],
    "prompts": [
      { "id": "style", "file": "prompts/style.md", "into": ["@default"], "position": "suffix" }
    ]
  }
}
```

## 打包注意事项

插件包要能在 Windows 与 macOS 上安装：

- 不要有**仅大小写不同**的路径（Windows / macOS 默认文件系统上解压时相互覆盖），上传时会被拒绝。
- 避免 Windows 保留名（`con` `aux` `nul` `com1`…）、以点或空格结尾的文件名、过长的相对路径（超过 120 字符），上传时会给出警告。
- 用 Windows 自带的「压缩」或 PowerShell `Compress-Archive` 生成的 zip 可以正常上传与安装。

## 上传与发布

在团队后台（`<接入地址>/admin/`）**模块 → 上传** zip，然后：

1. **状态流转**：`本地(local)` → `预发(staging)` → `正式(production)`，可回退。设备渠道决定可见性：正式渠道只看正式；预发渠道看预发 + 正式；本地渠道看全部。设备渠道在**设备**页下发。
2. **上线 / 下线**：下线后任何设备都看不到该版本，客户端会回落到其它可见版本或卸载。
3. **绑定客户端版本**：semver range，例如 `>=0.3.0 <1.0.0`（后台列出已注册设备的版本分布，可点击添加）。
4. **分组限定**：只对指定分组可见（空 = 全部）。
5. **可见性预览**：模拟某个端（渠道 / 版本 / 分组）会拿到哪些插件。
6. **紧急下线**：模块详情右上角，全部版本下线并通知在线客户端立即停用。

任何变更都会实时推送，在线的桌面端立即同步；离线的在下次启动或定时同步时更新。
模块详情的**发布进度**按当前规则算出每台客户端应得的版本，与客户端上报对比得到 已装上 / 安装失败 / 待同步 / 待卸载；
MCP 服务在明文地址下被停用也会显示为「部分失效」，原因见[团队 · 插件下发](team.md#插件下发)。

## 成员那边会发生什么

桌面端在启动时、收到通知时、以及每 5 分钟同步一次：下载 → 校验 → 解压到数据目录 → 校验清单 → 智能体空闲时重启生效。任一步失败保留旧版本。
技能与 MCP 服务出现在「设置 → 技能 / MCP」的「团队」一栏，成员可以逐个停用但不能编辑。
