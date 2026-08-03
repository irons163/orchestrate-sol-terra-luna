# Sol–Terra–Luna Orchestrator

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
