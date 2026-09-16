# DSH 本地预设仓库（agent presets）

本仓库保存本机自编的 DeepSeek Harness **agent preset**。每个预设一个目录，目录里放一份
`agent.cordis.yml`（agent-plane 组合文件），可选一份 `preset.yml` 提供选择器里显示的元数据。

仓库根目录**就是** harness 读取本地预设的那个根：

```
${DSH_HOME}/.agent-presets/          # 本机即 C:\Users\admin\.dsh\.agent-presets
```

即：克隆/恢复本仓库到该路径，预设就会被 harness 直接发现。

## 目录

| 路径 | 说明 |
| --- | --- |
| `whale/` | 预设 `whale`，显示名「白晶」 |
| `whale/agent.cordis.yml` | 组合文件（agent plane），逐行注释说明每行的平面与 realm 归属 |
| `whale/preset.yml` | 选择器里显示的 `name` 与 `description` |
| `README.md` | 本文件 |
| `.gitignore` | 忽略 `*.bak` / `*.bak-*` 等本地备份与临时产物 |

## whale（白晶）

- `~/.dsh/settings.yaml` 里 `agent-presets.default: whale`，新会话默认挂载本预设。
- 能力来源：`standard` 的全量行，加上 `ptc` 的 `present` 工具与 `presentation: both`，
  加上 `minimal` 的常驻 PTY 栈（默认关），加上 `cordis`（创造模式）的 `tool-cordis`（默认关）。
  四个内置预设里完全相同的行只保留一份。
- 人格：由 `persona` 行的 `prefix` 承载（称呼用户为「主人」、自称白晶、只用简体中文、
  喜欢米饭、拒绝「胖」），`suffix` 故意留空以保留完整身份说明。
- 平面划分：注册表、sandbox 与审批栈、持久化、模型路由、subagent 注册表与后端留在 host
  composition；本文件只贡献本会话的工具与 prompt 段落。发布服务的行一律待在带 `isolate`
  realm 的 group 里（`terminals`、`planMode`、`compaction`/`toolResultPruner`、`workflowEngine`）。

### 默认关闭、按需开启的行

| 行 | 默认 | 开启方式 | 代价 / 前提 |
| --- | --- | --- | --- |
| `persistent-shell`（常驻 PTY 栈） | `disabled: true` | 去掉该 group 的 `disabled`，并给一次性 shell 行（`tool-pwsh`/`tool-bash`）加 `disabled: true` | 两套 shell 的工具同名（`pwsh`/`bash`），同时启用会在挂载时报 “already registered” |
| `tool-cordis`（运行时自省 / 自改） | `disabled: true` | 去掉该行的 `disabled` | 与创造模式（`cordis`）预设在**同一进程内互斥**：Host 的 Cordis inspect provider id 是进程全局的，先挂载者占位，后者整个会话起不来 |
| `tool-subagent-codex` / `tool-subagent-claude-code` | `disabled: true` | 先在本 Profile 里 `dsh plugin --profile <name> add @deepseek-ai/dsh-subagent-<x>` 并重启 Host，再去掉对应 `disabled` | 未安装对应 Bundle 时开启会让该预设挂载失败 |

### 校验状态

- 本预设是当前会话实际运行的组合（`agent-presets.default: whale`），会话里的工具目录与
  `agent.cordis.yml` 的行一一对应，因此「可正常挂载」已由实机验证。
- 改动后请**开一个新的白晶会话**确认工具清单：preset 决定工具 schema 与 prompt 段落，
  只有真实会话才展示这份组合产出的 agent。`cordis_mount` / `cordis_inspect` 这类探针
  只在创造模式预设下可用，白晶默认拿不到。

### 硬性禁忌

- **不要**编辑随部署安装的预设（`@deepseek-ai/dsh-agent-presets/presets/` 下的
  `standard` / `ptc` / `minimal` / `cordis`）：它属于部署，升级会覆盖，写坏 `cordis`
  会连预设创作能力一起失去。要改随附预设，就复制成本仓库里的一份新预设再改副本。
- **不要**把发布服务的行裸放进 preset：那会注册进进程全局 realm，第二个挂载该预设的会话
  直接冲突；必须用带 `isolate` realm 的 group 包住提供者与其全部消费者。

## 版本管理约定

- 预设的历史即本仓库的提交历史；`*.bak` / `*.bak-*` 之类的本地备份不纳入跟踪。
- 提交信息格式：`预设(<id>): <一句话改动>`，正文写清改动的**行**、**原因**与**影响**。

## 恢复

```powershell
git -C C:\Users\admin\.dsh\.agent-presets log --oneline
git -C C:\Users\admin\.dsh\.agent-presets diff HEAD -- whale/agent.cordis.yml
git -C C:\Users\admin\.dsh\.agent-presets checkout <commit> -- whale/agent.cordis.yml
```
