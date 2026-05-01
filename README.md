# Token Cost Calculator

A [CloudCLI](https://cloudcli.ai) plugin that calculates API token costs across five modes — manual entry, per-session auto-detect, per-project totals, all-time aggregates, and daily breakdowns. Supports 8 built-in model pricing presets (Anthropic, OpenAI, Google, DeepSeek) plus custom pricing.

## Features

- **5 calculation modes:**
  - **Manual** — enter token counts by hand with quick-add buttons
  - **Auto-detect** — reads token usage from the active Claude session transcript (JSONL)
  - **Project** — sums all sessions under a single project folder, with per-session breakdown
  - **All-time** — aggregates every session across all projects since install, with per-project colored bars
  - **Daily** — per-day token usage and cost, with a date navigator (defaults to today)
- **Preset pricing** for 8 models: Claude Opus 4.7, Sonnet 4.6, Haiku 4.5, GPT-4o, GPT-4o-mini, Gemini 2.5 Pro/Flash, DeepSeek V4 Pro
- **Custom pricing** — enter any input / output / cached-input price per 1M tokens (USD)
- **Auto-detect** session token usage from `~/.claude/projects/*.jsonl` transcript files (includes sub-agent sessions)
- **Dark / light theme** support, follows the host CloudCLI theme
- **Persistent state** — mode, prices, and manual token counts saved to localStorage
- **Deduplication** — correctly handles multi-block API responses (thinking + text + tool_use) from reasoning models like DeepSeek

## Installation

```bash
cd ~/.claude-code-ui/plugins
git clone https://github.com/YOUR_USERNAME/cloudcli-plugin-token-cost-calculator.git token-cost-calculator
cd token-cost-calculator
npm install
npm run build
```

Then restart CloudCLI or reload the plugins panel.

## Usage

1. Open CloudCLI and navigate to the **Token Cost Calculator** tab.
2. Select a model from the **Model Preset** dropdown, or enter custom prices.
3. Choose a **Mode**:

| Mode | What it does |
|---|---|
| Manual | Enter token counts manually. Use **Quick Add** buttons (+1K, +10K, +100K, +1M, +10M) for rapid input. |
| Auto-detect | Reads tokens from the active session. If no session is active, enter the project path + session ID manually. |
| Project | Sums all sessions under one project folder. Enter the project path and click **Fetch**. |
| All-time | Aggregates every session across all projects. Shows a per-project breakdown with colored progress bars. |
| Daily | Groups token usage by day. Navigate with ◀/▶ buttons or click any day in the breakdown list. Defaults to today. |

4. View the **Cost Breakdown** section for input, output, cached-input, and total cost.

## How it works

The plugin backend reads Claude Code's session transcript files stored in `~/.claude/projects/<encoded-project-path>/<session-id>.jsonl`. Each JSONL line with `"type": "assistant"` contains a `message.usage` object with token counts:

- `input_tokens` — prompt tokens charged at the input price
- `output_tokens` — completion tokens charged at the output price
- `cache_creation_input_tokens` + `cache_read_input_tokens` — cached tokens charged at the cache price
- `timestamp` — used for daily aggregation

Sub-agent sessions (from `Task` tool invocations) are also scanned under `<session-id>/subagents/*.jsonl`.

## Project structure

```
token-cost-calculator/
├── manifest.json        # Plugin metadata (CloudCLI)
├── package.json
├── tsconfig.json
├── icon.svg
└── src/
    ├── index.ts         # Frontend entry — UI rendering, event handling, RPC calls
    ├── server.ts        # Backend server — HTTP API, JSONL parsing, token aggregation
    └── types.ts         # Plugin API type definitions
```

## API endpoints

| Endpoint | Description |
|---|---|
| `GET /presets` | Returns built-in model pricing presets |
| `GET /session-usage?projectPath=...&sessionId=...` | Token usage for a single session |
| `POST /calculate` | Server-side cost calculation |
| `GET /project-usage?projectPath=...` | Aggregated usage for all sessions in a project |
| `GET /all-usage` | Aggregated usage across all projects |
| `GET /daily-usage` | Token usage grouped by day across all projects |

## Development

```bash
npm install
npm run dev        # watch mode
npm run build      # single build
```

The plugin follows the [CloudCLI Plugin Architecture](https://cloudcli.ai/docs/plugins/plugin-overview):
- `manifest.json` declares the plugin metadata, entry points, and slot type
- `src/index.ts` exports `mount(container, api)` and `unmount(container)` lifecycle hooks
- `src/server.ts` runs as a Node.js HTTP subprocess, signaling readiness via stdout JSON

## License

MIT
