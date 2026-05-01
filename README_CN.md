# Token Cost Calculator（Token 费用计算器）

一个 [CloudCLI](https://cloudcli.ai) 插件，用于通过五种模式计算 API Token 费用——手动输入、单会话自动检测、项目汇总、全局总计和每日明细。内置 8 种模型预设定价（Anthropic、OpenAI、Google、DeepSeek），同时支持自定义价格。

## 功能特性

- **5 种计算模式：**
  - **Manual（手动）** — 手动输入 token 数量，支持快捷添加按钮
  - **Auto-detect（自动检测）** — 从当前 Claude 会话的 JSONL 转录文件中读取 token 使用量
  - **Project（项目）** — 汇总单个项目文件夹下所有会话的 token 使用量，含逐会话明细
  - **All-time（全局）** — 统计自安装以来的所有项目全部会话，含逐项目彩色进度条
  - **Daily（每日）** — 按天统计 token 使用量和费用，支持日期切换（默认显示当天）
- **预设定价**，覆盖 8 款模型：Claude Opus 4.7、Sonnet 4.6、Haiku 4.5、GPT-4o、GPT-4o-mini、Gemini 2.5 Pro/Flash、DeepSeek V4 Pro
- **自定义价格** — 可输入任意 input / output / cached input 的每百万 token 价格（美元）
- **自动检测**会话 token 用量，读取 `~/.claude/projects/*.jsonl` 转录文件（含子代理会话）
- **深色 / 浅色主题**支持，跟随 CloudCLI 宿主主题
- **状态持久化** — 模式、价格和手动 token 数量保存到 localStorage
- **去重处理** — 正确处理推理模型（如 DeepSeek）的多块 API 响应（thinking + text + tool_use）

## 安装

```bash
cd ~/.claude-code-ui/plugins
git clone https://github.com/YOUR_USERNAME/cloudcli-plugin-token-cost-calculator.git token-cost-calculator
cd token-cost-calculator
npm install
npm run build
```

然后重启 CloudCLI 或刷新插件面板。

## 使用方法

1. 打开 CloudCLI，切换到 **Token Cost Calculator** 标签页。
2. 从 **Model Preset** 下拉菜单选择模型，或输入自定义价格。
3. 选择一种 **Mode（模式）**：

| 模式 | 说明 |
|---|---|
| Manual | 手动输入 token 数量。使用 **Quick Add** 按钮（+1K、+10K、+100K、+1M、+10M）快速输入。 |
| Auto-detect | 从当前会话读取 token。若无活跃会话，可手动输入项目路径和会话 ID。 |
| Project | 汇总单个项目下所有会话。输入项目路径后点击 **Fetch**。 |
| All-time | 统计所有项目的全部会话。以彩色进度条展示逐项目明细。 |
| Daily | 按天分组统计。使用 ◀/▶ 按钮切换日期，或点击明细列表中的任意日期。默认显示当天。 |

4. 查看 **Cost Breakdown** 区域，了解 input、output、cached-input 及总费用。

## 工作原理

插件后端读取 Claude Code 存储在 `~/.claude/projects/<encoded-project-path>/<session-id>.jsonl` 的会话转录文件。每条 `"type": "assistant"` 的 JSONL 行包含 `message.usage` 对象，内含 token 计数：

- `input_tokens` — 提示 token，按 input 价格计费
- `output_tokens` — 输出 token，按 output 价格计费
- `cache_creation_input_tokens` + `cache_read_input_tokens` — 缓存 token，按缓存价格计费
- `timestamp` — 用于按天汇总

子代理会话（来自 `Task` 工具调用）也会扫描 `<session-id>/subagents/*.jsonl` 目录。

## 项目结构

```
token-cost-calculator/
├── manifest.json        # 插件元数据（CloudCLI）
├── package.json
├── tsconfig.json
├── icon.svg
└── src/
    ├── index.ts         # 前端入口 — UI 渲染、事件处理、RPC 调用
    ├── server.ts        # 后端服务 — HTTP API、JSONL 解析、token 汇总
    └── types.ts         # 插件 API 类型定义
```

## API 接口

| 接口 | 说明 |
|---|---|
| `GET /presets` | 返回内置模型预设价格 |
| `GET /session-usage?projectPath=...&sessionId=...` | 单个会话的 token 用量 |
| `POST /calculate` | 服务端费用计算 |
| `GET /project-usage?projectPath=...` | 某项目下全部会话的汇总用量 |
| `GET /all-usage` | 所有项目的汇总用量 |
| `GET /daily-usage` | 按天分组的 token 用量 |

## 开发

```bash
npm install
npm run dev        # watch 模式
npm run build      # 单次构建
```

本插件遵循 [CloudCLI 插件架构](https://cloudcli.ai/docs/plugins/plugin-overview)：
- `manifest.json` 声明插件元数据、入口点和槽位类型
- `src/index.ts` 导出 `mount(container, api)` 和 `unmount(container)` 生命周期钩子
- `src/server.ts` 作为 Node.js HTTP 子进程运行，通过 stdout JSON 通知就绪状态

## 许可证

MIT
