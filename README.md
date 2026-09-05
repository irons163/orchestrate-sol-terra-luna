# Three-Tier Agent Orchestrator

> 本 README 以中文為主，英文版請見文末。

一個 Codex skill，讓 GPT-6 Astra 留在主線程擔任總指揮：理解目標、拆分任務、做架構決策、檢查結果並整合輸出；並依工作性質把獨立子任務交給 GPT-5.6 Sol Mid 與 GPT-5.6 Luna Max。

## 角色分工

| 角色 | 責任 |
| --- | --- |
| GPT-6 Astra | 理解目標、拆分任務、架構決策、檢查結果與整合輸出 |
| GPT-5.6 Sol Mid | 困難但邊界清楚的分析、實作、深度 code review 與複雜 debugging |
| GPT-5.6 Luna Max | 清楚、可重複且容易驗證的搜尋、測試、重現、機械式修改與摘要 |

## 主要行為

- 執行實質工作前先確認 GPT-6 Astra、GPT-5.6 Sol Mid 與 GPT-5.6 Luna Max 是否可用。
- 任一必要模型或指定 reasoning 不可用時，完整停止工作，不以其他模型或較低 reasoning 替代。
- 能力齊全後，使用明確的目標、範圍、完成條件與驗證方式委派子任務。
- 由 GPT-6 Astra 檢查重要 diff、測試與證據，並負責最終整合。

## 必要條件

- Codex 支援 skills 與 subagents，且 subagent 工具可直接指定模型與 reasoning effort。
- 主線程可使用 `gpt-6-astra`。
- Sol Mid subagent 使用 `gpt-5.6-sol` 與 `medium` reasoning effort。
- Luna Max subagent 使用 `gpt-5.6-luna` 與 `max` reasoning effort。

不需要建立額外的 custom agent 設定檔。Skill 每次啟動 subagent 時都會直接傳入指定的模型與 reasoning effort；實際 subagent 啟動結果才是能力判準。

## 安裝

在 macOS 或 Linux 執行：

```bash
git clone https://github.com/irons163/three-tier-agent-orchestrator.git "${CODEX_HOME:-$HOME/.codex}/skills/three-tier-agent-orchestrator"
```

若目前 task 沒有重新載入 skill，請建立新 task；仍未出現時再重新啟動 Codex App。

## 使用

在 Codex prompt 明確啟用：

```text
$three-tier-agent-orchestrator
```

也可以直接描述工作，例如：

```text
使用 Three-Tier Agent Orchestrator 檢查這個專案：GPT-6 Astra 負責理解目標與整合，GPT-5.6 Sol Mid 做架構與安全審查，GPT-5.6 Luna Max 執行測試與整理錯誤。
```

## Repository 結構

```text
.
├── README.md
├── SKILL.md
└── agents/
    └── openai.yaml
```

不需要額外的 custom agent 設定檔。

---

## English Version

### Three-Tier Agent Orchestrator

A Codex skill that keeps GPT-6 Astra in the main thread as the lead orchestrator: understanding the objective, breaking down tasks, making architectural decisions, reviewing results, and integrating the final output while delegating independent subtasks to GPT-5.6 Sol Mid and GPT-5.6 Luna Max based on the nature of the work.

### Role assignments

| Role | Responsibilities |
| --- | --- |
| GPT-6 Astra | Understand the objective, break down tasks, make architectural decisions, review results, and integrate the final output |
| GPT-5.6 Sol Mid | Handle difficult but clearly bounded analysis and implementation, in-depth code review, and complex debugging |
| GPT-5.6 Luna Max | Perform clear, repeatable, and easily verifiable searches, tests, reproductions, mechanical edits, and summarization |

### Core behavior

- Verify that GPT-6 Astra, GPT-5.6 Sol Mid, and GPT-5.6 Luna Max are available before starting substantive work.
- If any required model or specified reasoning capability is unavailable, stop completely instead of substituting another model or using a lower reasoning level.
- Once all capabilities are available, delegate subtasks with explicit objectives, scope, completion criteria, and validation methods.
- GPT-6 Astra reviews important diffs, tests, and evidence, and is responsible for the final integration.

### Requirements

- Codex supports skills and subagents, and the subagent tool can specify the model and reasoning effort directly.
- The main thread can use `gpt-6-astra`.
- The Sol Mid subagent uses `gpt-5.6-sol` with `medium` reasoning effort.
- The Luna Max subagent uses `gpt-5.6-luna` with `max` reasoning effort.

No additional custom agent configuration files are required. The skill passes the specified model and reasoning effort directly whenever it launches a subagent. The actual subagent startup result determines whether the capability is available.

### Installation

Run the following on macOS or Linux:

```bash
git clone https://github.com/irons163/three-tier-agent-orchestrator.git "${CODEX_HOME:-$HOME/.codex}/skills/three-tier-agent-orchestrator"
```

If the current task does not reload the skill, create a new task. If it still does not appear, restart the Codex App.

### Usage

Explicitly enable the skill in a Codex prompt:

```text
$three-tier-agent-orchestrator
```

You can also describe the work directly, for example:

```text
Use the Three-Tier Agent Orchestrator to review this project: GPT-6 Astra handles objective understanding and integration, GPT-5.6 Sol Mid performs the architecture and security review, and GPT-5.6 Luna Max runs tests and organizes the errors.
```

### Repository structure

```text
.
├── README.md
├── SKILL.md
└── agents/
    └── openai.yaml
```

No additional custom agent configuration files are required.
