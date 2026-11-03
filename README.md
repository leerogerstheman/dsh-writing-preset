# dsh-writing-preset — 文学创作模式

给 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)（DSH）加一个**面向文学写作**的 Agent 预设：注册 id 为 `writing`，在模式选择里显示为「**文学创作**」。

内置的四个模式（标准 / PTC / 极简 / 创造）都以编程为中心，这个预设换掉的是**人格**和**工具集**：

- 它是写作搭档，不是编码助手：工作区里的文件对它是稿件、设定和资料，不是代码库。
- 修订默认做最小改动，不整段重写、不覆盖作者的声音。
- 体裁、篇幅、视角、结局这类决定交还作者，其余细节按合理默认推进并说明假设。

## 它包含什么

声明本身就是一份普通的 Cordis 补丁（`cordis.patch.yml`），只插入一条 `@deepseek-ai/dsh-agent-preset` 行：

| 插件 | 作用 |
|---|---|
| `dsh-persona` | 写作人格：先读后写、保留作者声音、说清改动、具体胜过形容；约定 `outline.md` / `characters.md` / `world.md` / `manuscript/第01章-标题.md` 的稿件结构 |
| `dsh-tool-fs` | `read` / `write` / `edit` — 读写稿件本身 |
| `dsh-tool-fs-search` | `glob` / `grep` — 跨章节找伏笔、核对人名与设定 |
| `dsh-tool-web` | `web_search` / `web_fetch` — 查证史实、地名、方言、专业流程 |
| `dsh-tool-ask-user` | 把创作上的选择交还作者 |
| `dsh-tool-present` | 成稿作为文件交付 |
| `dsh-skill-filesystem` + `dsh-tool-skill` | 工作区技能；顺带带上内置的 office 技能，需要时能导 Word / PDF 交稿 |
| `dsh-tool-todo` | 长篇排章节进度 |
| `dsh-compaction-basic` + `dsh-command-compact` | 长篇上下文压缩，`/compact` 手动压缩 |
| `dsh-tool-pwsh` / `dsh-tool-bash` | 仅作字数统计、格式转换这类稿件外的辅助，人格里明确禁止拿它做正文批量替换 |
| `dsh-time-context` | 当前时间与耗时 |

**刻意没有包含**：jobs、schedule、goal、plan-mode、子代理、workflow、插件管理 —— 这些是编程向模式的重装备。

## 安装

### 方式一：让 DSH 自己装（推荐）

在 **创造模式（Creator）** 的会话里说：

> 安装 https://github.com/leerogerstheman/dsh-writing-preset 这个 bundle

Agent 会调用 `plugin_manager` 的 `install_bundle` 完成安装并验证。

### 方式二：手动

在 DSH 里用 `plugin_manager` 工具：

```json
{ "action": "install_bundle", "target": "git+https://github.com/leerogerstheman/dsh-writing-preset.git" }
```

或者手动改 profile（`$DSH_HOME/profiles/<profile>/package.json`）：把本包加进 `dependencies`，并把 `dsh-writing-preset` 追加进 `dsh.profile.bundles`，然后重启 DSH。

安装完成后，`plugin_manager` 的 `list_plugins` 里应当出现 `preset-writing`，且 `fiberPhase: active`。

## 使用

1. 新建任务 → 点模式按钮 → 选「**文学创作**」。
2. 想设为默认：**设置 → Agent 预设 → 文学创作** 卡片上点一下（设为新任务默认）。
3. 已存在的会话不会中途切换模式——模式在任务开始时确定。

## 自定义

- **改人格**：编辑 `cordis.patch.yml` 里 `persona.config.prefix` 的文本，重载 bundle 后生效。
- **加工具**：往 `plugins` 列表里加一行，`name` 用 `@deepseek-ai/dsh-*` 包名。
- **换 id / 显示名**：改 `config.id`、`config.name`、`config.description`、`config.order`。

## 踩坑记录

`@deepseek-ai/dsh-tool-fs-search` 的 `sampleOverCapGlobResults` 是**必填字段**（没有默认值）。漏掉它不会报错到终端，而是让整个预设被判为 broken：设置页里卡片带红色「加载失败」，新建任务页的模式菜单里则直接看不到这个预设（菜单只列健康的预设）。排查方法：诊断文本是预设的组合审计结果，悬停那个红色小徽标可以看到本行；或者装一个一次性插件调 `ctx.agentPresets.list()` 把 `broken` 原文导出来。

## 许可

MIT
