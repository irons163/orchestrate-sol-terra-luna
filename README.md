# Sol–Terra–Luna Orchestrator

> 本 README 以中文為主，英文版請見文末。

一個 Codex skill，讓 GPT-5.6 Sol 留在主線程擔任總指揮，並依工作性質把獨立子任務交給 Terra Max 與 Luna Max。

## 角色分工

| 角色 | 責任 |
| --- | --- |
| GPT-5.6 Sol | 理解目標、拆分任務、架構決策、檢查結果與整合輸出 |
| GPT-5.6 Terra Max | 困難但邊界清楚的分析、實作、深度 code review 與複雜 debugging |
| GPT-5.6 Luna Max | 清楚、可重複且容易驗證的搜尋、測試、重現、機械式修改與摘要 |

## 主要行為

- 執行實質工作前先確認 Sol、Terra Max 與 Luna Max 是否可用。
- 任一必要模型或 Max reasoning 不可用時，完整停止工作，不以其他模型或較低 reasoning 替代。
- 能力齊全後，使用明確的目標、範圍、完成條件與驗證方式委派子任務。
- 由 Sol 檢查重要 diff、測試與證據，並負責最終整合。

## 必要條件

- Codex 支援 skills、custom agents 與 subagents。
- 主線程可使用 `gpt-5.6-sol`。
- subagent 可使用 `gpt-5.6-terra` 和 `gpt-5.6-luna`，且兩者支援 `max` reasoning effort。

Codex App 的 model picker 是否勾選 Max 只影響 composer 中是否顯示該選項，不影響 custom agent 檔案明確設定 `model_reasoning_effort = "max"`。實際 subagent 啟動結果才是能力判準。

## 安裝

在 macOS 或 Linux 執行：

```bash
git clone https://github.com/irons163/orchestrate-sol-terra-luna.git "${CODEX_HOME:-$HOME/.codex}/skills/orchestrate-sol-terra-luna"
```

若目前 task 沒有重新載入 skill，請建立新 task；仍未出現時再重新啟動 Codex App。

## 使用

在 Codex prompt 明確啟用：

```text
$orchestrate-sol-terra-luna
```

也可以直接描述工作，例如：

```text
使用 Sol–Terra–Luna Orchestrator 檢查這個專案：Sol 負責整合，Terra Max 做架構與安全審查，Luna Max 執行測試與整理錯誤。
```

若 Luna Max 尚未設定，可以要求：

```text
請 AI 幫我設定 Luna Max
```

設定完成但目前 task 尚未載入時，可以要求：

```text
請 AI 複製目前 task 並測試 Luna Max
```

## Repository 結構

```text
.
├── README.md
├── SKILL.md
└── agents/
    └── openai.yaml
```

個人的 `luna-max.toml` 或 `terra-max.toml` 不包含在 repository 中；skill 會依使用者環境提供建立與驗證指引。

---

## English Version

### Sol–Terra–Luna Orchestrator

A Codex skill that keeps GPT-5.6 Sol in the main thread as the lead orchestrator, while delegating independent subtasks to Terra Max and Luna Max based on the nature of the work.

### Role assignments

| Role | Responsibilities |
| --- | --- |
| GPT-5.6 Sol | Understand the objective, break down tasks, make architectural decisions, review results, and integrate the final output |
| GPT-5.6 Terra Max | Handle difficult but clearly bounded analysis and implementation, in-depth code review, and complex debugging |
| GPT-5.6 Luna Max | Perform clear, repeatable, and easily verifiable searches, tests, reproductions, mechanical edits, and summarization |

### Core behavior

- Verify that Sol, Terra Max, and Luna Max are available before starting substantive work.
- If any required model or Max reasoning capability is unavailable, stop completely instead of substituting another model or using a lower reasoning level.
- Once all capabilities are available, delegate subtasks with explicit objectives, scope, completion criteria, and validation methods.
- Sol reviews important diffs, tests, and evidence, and is responsible for the final integration.

### Requirements

- Codex supports skills, custom agents, and subagents.
- The main thread can use `gpt-5.6-sol`.
- Subagents can use `gpt-5.6-terra` and `gpt-5.6-luna`, and both support `max` reasoning effort.

Whether Max is selected in the Codex App model picker only affects whether the option is displayed in the composer. It does not affect the explicit `model_reasoning_effort = "max"` setting in custom agent files. The actual subagent startup result is the criterion for determining whether the capability is available.

### Installation

Run the following on macOS or Linux:

```bash
git clone https://github.com/irons163/orchestrate-sol-terra-luna.git "${CODEX_HOME:-$HOME/.codex}/skills/orchestrate-sol-terra-luna"
```

If the current task does not reload the skill, create a new task. If it still does not appear, restart the Codex App.

### Usage

Explicitly enable the skill in a Codex prompt:

```text
$orchestrate-sol-terra-luna
```

You can also describe the work directly, for example:

```text
Use the Sol–Terra–Luna Orchestrator to review this project: Sol handles integration, Terra Max performs the architecture and security review, and Luna Max runs tests and organizes the errors.
```

If Luna Max has not been configured, you can ask:

```text
Please help me configure Luna Max.
```

After configuration, if the current task has not loaded the skill yet, you can ask:

```text
Please duplicate the current task and test Luna Max.
```

### Repository structure

```text
.
├── README.md
├── SKILL.md
└── agents/
    └── openai.yaml
```

Personal `luna-max.toml` and `terra-max.toml` files are not included in the repository. The skill provides instructions for creating and validating them based on the user's environment.
