# DSH 本地预设仓库（agent presets）

本仓库保存本机自编的 DeepSeek Harness **agent preset** 的版本历史与备份：每个预设一个目录。

⚠️ **0.2.0-rc.1 起，harness 不再读取本目录。** 预设的载体已经改成 **bundle patch**：一份
`- insert:` patch 里的 `@deepseek-ai/dsh-agent-preset` 声明，由 profile 的
`dsh.profile.bundles` 引入。本机白晶预设的真实载体是：

```
C:\Users\admin\.dsh\local-bundles\dsh-user-presets\cordis.patch.yml     # 源文件（改这份）
C:\Users\admin\.dsh\profiles\web\node_modules\@local\dsh-user-presets\cordis.patch.yml  # profile 里的安装副本
```

因此本仓库现在的角色是：**预设的历史、备份与 diff 视图**；改动要落到上面那份 bundle patch，
再重装/热重载才会生效（见「恢复」）。

## 目录

| 路径 | 说明 |
| --- | --- |
| `whale/` | 预设 `whale`，显示名「白晶」 |
| `whale/cordis.patch.yml` | **权威载体**的逐字节副本（bundle patch 全文，含声明包装与全部注释） |
| `whale/agent.cordis.yml` | 同一份组合的可读镜像：就是上面那份 patch 里 `config.plugins` 的逐行内容，便于逐行 diff |
| `whale/preset.yml` | 选择器里显示的 `name` 与 `description` |
| `README.md` | 本文件 |
| `.gitignore` | 忽略 `*.bak` / `*.bak-*` 等本地备份与临时产物 |

`cordis.patch.yml` 与 `agent.cordis.yml` 必须一致：前者是还原用的事实，后者只用于阅读。
改了 preset 就同时更新两份（`agent.cordis.yml` = patch 的 `config.plugins` 去掉 10 空格缩进）。

## whale（白晶）

- 默认预设：`profiles/web/cordis.patch.yml` 里 `agent-preset-registry.selectedDefault: whale`
  （新建会话默认挂载本预设；本例 `~/.dsh/cordis.patch.yml` 与 `settings.yaml` 已不再承载该设置）。
- 能力来源：`standard` 的全量行，加上 `ptc` 的 `present` 工具与 `presentation: both`，
  加上 `minimal` 的常驻 PTY 栈（默认关），加上 `cordis`（创造模式）的 `tool-cordis` 自省/自改工具。
  四个内置预设里完全相同的行只保留一份；`tool-ralph` 四个内置预设都关着，本预设同样关闭。
- 人格：由 `persona` 行的 `prefix` 承载（称呼用户为「主人」、自称白晶、只用简体中文、
  喜欢米饭、拒绝「胖」），`suffix` 故意留空以保留完整身份说明。
- 平面划分：注册表、sandbox 与审批栈、持久化、模型路由、subagent 注册表与后端留在 host
  composition；本文件只贡献本会话的工具与 prompt 段落。发布服务的行一律待在带 `isolate`
  realm 的 group 里（`terminals`、`planMode`、`compaction`/`toolResultPruner`、`workflowEngine`）。

### 默认关闭、按需开启的行

| 行 | 默认 | 开启 / 关闭方式 | 代价 / 前提 |
| --- | --- | --- | --- |
| `persistent-shell`（常驻 PTY 栈） | `disabled: true` | 去掉该 group 的 `disabled`，并给一次性 shell 行（`tool-pwsh`/`tool-bash`）加 `disabled: true` | 两套 shell 的工具同名（`pwsh`/`bash`），同时启用会在挂载时报 “already registered” |
| `tool-cordis`（运行时自省 / 自改） | **开启** | 想关：给该行加 `disabled: true` | 曾经与创造模式预设**同进程互斥**（Host 的 Cordis inspect provider id 是进程全局的）；上游 `4aba48ec03`（2026-09-21）已把 4 个 provider 移到 host composition，预设行只注册 `cordis_inspect_list` / `cordis_inspect_query` 两个工具，互斥已消失，2026-09-29 实测两者同进程共存正常 |
| `tool-ralph`（自循环） | `disabled: true` | 去掉 `disabled` | 四个内置预设都关着；本预设服从上游默认 |
| `skill-filesystem` 的 `customSkillDirs` | 指向 harness 的 agent-preset 包 `skills/` | 检出目录搬家就必须同步改这个**写死路径** | 指向不存在的目录会让该行报错、整份预设挂载失败 |
| `tool-subagent-codex` / `tool-subagent-claude-code` | `disabled: true` | 先在本 Profile 里 `dsh plugin --profile <name> add @deepseek-ai/dsh-subagent-<x>` 并重启 Host，再去掉对应 `disabled` | 未安装对应 Bundle 时开启会让该预设挂载失败 |

### 校验状态

- 2026-09-17 重新校验通过：名册 `broken` 为空，`standingKeyFor('whale')` 正常返回
  （2026-09-16 首次入库时同样通过；之后随 DSH 换包出现过一次行级依赖失效，见「变更记录」）。
- 2026-09-29 复核通过：两次真实白晶会话（大肥鱼工作区、检出目录）读会话日志确认——
  技能目录出现 `editing-cordis-compositions` 等 4 个 harness 技能、工具表出现
  `cordis_inspect_list` / `cordis_inspect_query` 且 `ralph` 消失、人格与 `run_code` 形态不变；
  白晶会话与本机的创造模式会话同进程并跑，未再报 inspect provider 重复注册。
- 改动后请**开一个新的白晶会话**确认工具清单：preset 决定工具 schema 与 prompt 段落，
  只有真实会话才展示这份组合产出的 agent；已存在的会话保留它启动时的插件版本。

### 行级依赖与校验方式

- 每个 `name:` 都必须能在当前部署中解析到真实包。本机是**源码直跑**（`D:\DSH_Server\deepseek-harness`
  经 tsx 启动），解析根是运行中的 dsh 安装，不是 profile 的 `node_modules`。
- `customSkillDirs` 与任何写死路径同理：只在当前目录布局下有效，检出搬家后必须跟着改。
- DSH 升级 / 换包之后，预设里的行可能失效（包已不在）。校验方式（诚实、可重复）：

| 步骤 | 方式 | 通过标准 |
| --- | --- | --- |
| 1. 解析检查 | 在创造模式会话里读 `ctx.agentPresets.list()` 中 `whale` 的 `broken` 字段 | `broken === undefined` |
| 2. 真实挂载 | 开一个新的白晶会话（或用 `ctx.agentPresets.standingKeyFor('whale')`） | 正常组合，工具表符合预期、不报错 |

第 2 步按开会话的同一流程组合整棵插件子树，才能抓到包不存在、配置非法、行从未激活、
服务被发布进 root realm 这四类失败；第 1 步只覆盖 YAML 形状与包能否解析。

### 变更记录

| 日期 | 改动 | 原因与方式 |
| --- | --- | --- |
| 2026-09-16 | 首次纳入版本管理 | 归档 `whale`（白晶），预设内容本身未改动；提交 `bec519f`。 |
| 2026-09-17 | `delegation` 组：`workflow-worker-thread` 行 → `workflow-ptc` 行 | 名册报 `row "workflow-worker-thread" names a plugin that cannot be resolved: @deepseek-ai/dsh-workflow-worker-thread`，整份预设无法挂载。该包已不在部署的包集合里（`packages/workflow` 只剩 `workflow` / `workflow-ptc` / `tool-workflow` / `tool-ralph`），profile 中的 junction 指向已被删除的旧全局安装。改法：换成 `standard` 预设同款、解析正常的 `@deepseek-ai/dsh-workflow-ptc`。 |
| 2026-09-29 | ①`skill-filesystem` 补 `customSkillDirs`；②`tool-cordis` 打开；③`tool-ralph` 关闭；④persona 里预设载体说明改写 | ①persona 要求改 composition 前加载 `editing-cordis-compositions`，而 agent-preset 包的 `skills/` 不在默认发现根里，白晶会话看不到该技能（用写死路径，`!!js createRequire(baseUrl)` 从 profile/bundle 解析会 MODULE_NOT_FOUND）。②上游 `4aba48ec03` 已修掉 provider 同进程互斥。③对齐四个内置预设。④`.agent-presets` 已不再被 harness 读取，旧说明会把模型引去改死目录。载体改为 bundle patch，`agent.cordis.yml` 保留为可读镜像，新增 `cordis.patch.yml` 逐字节副本。 |

### 硬性禁忌

- **不要**编辑随部署安装的预设（`packages/bundle/web-app/presets/` 下的
  `standard` / `ptc` / `minimal` / `cordis`）：它属于部署，升级会覆盖，写坏 `cordis`
  会连预设创作能力一起失去。要改随附预设，就复制成本仓库里的一份新预设再改副本。
- **不要**把发布服务的行裸放进 preset：那会注册进进程全局 realm，第二个挂载该预设的会话
  直接冲突；必须用带 `isolate` realm 的 group 包住提供者与其全部消费者。

## 版本管理约定

- 预设的历史即本仓库的提交历史；`*.bak` / `*.bak-*` 之类的本地备份不纳入跟踪。
- 一次改动要同时更新：`whale/cordis.patch.yml`（权威载体副本）、`whale/agent.cordis.yml`
  （可读镜像）、`whale/preset.yml`（若元数据变）、本 README 的变更记录。
- 提交信息格式：`预设(<id>): <一句话改动>`，正文写清改动的**行**、**原因**与**影响**。

## 恢复

```powershell
$repo = 'C:\Users\admin\.dsh\.agent-presets'
$src  = 'C:\Users\admin\.dsh\local-bundles\dsh-user-presets\cordis.patch.yml'
$inst = 'C:\Users\admin\.dsh\profiles\web\node_modules\@local\dsh-user-presets\cordis.patch.yml'

git -C $repo log --oneline                                  # 看历史
git -C $repo diff HEAD -- whale/agent.cordis.yml            # 看这份预设逐行改了什么

# 1) 回退到某个提交：把仓库里的权威载体拷回 live 位置（两份都要）
git -C $repo checkout <commit> -- whale/cordis.patch.yml
Copy-Item "$repo\whale\cordis.patch.yml" $src  -Force
Copy-Item "$repo\whale\cordis.patch.yml" $inst -Force

# 2) 让运行中的 Host 重新组合（无需 pnpm；install_bundle 会跑包管理器）
#    plugin_manager: action=set_bundle, target=@local/dsh-user-presets, enabled=true
#    或重启 Host；随后开一个新的白晶会话验证工具表。
```

> 注：本机 profile 的 `pnpm-lock.yaml` 里有一批当天发布的 `@linxin666/*@0.4.4`，会触发
> pnpm 的 `ERR_PNPM_MINIMUM_RELEASE_AGE_VIOLATION` 供应链策略，`install_bundle` /
> `pnpm dsh plugin --profile web update` 目前都会在校验阶段失败——所以热重载用
> `set_bundle`，或者等这批包过了最短发布时长。
