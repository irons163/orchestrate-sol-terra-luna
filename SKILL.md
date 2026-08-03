---
name: orchestrate-sol-terra-luna
description: "讓 GPT-5.6 Sol 留在主線程（main thread）擔任總指揮，將困難但邊界清楚的工作交給 GPT-5.6 Terra Max，將清楚、可重複的工作交給 GPT-5.6 Luna Max。當使用者要求 Sol 總指揮、Terra Max 或 Luna Max subagent、分層 multi-agent coding、平行 code review、模組分析、獨立功能實作、測試、debugging 或整合結果時使用；開始前檢查模型與 Max reasoning 是否可用，若任一必要模型或 Max 不可用，完整停止所有工作並告知如何開啟，確認可用後才執行。"
---

# Sol–Terra–Luna Orchestration

讓 Sol 保留主線程，負責理解目標、拆分任務、架構判斷、驗收與整合。把困難但邊界明確的工作交給 Terra Max，把清楚、可重複且容易驗證的工作交給 Luna Max。

## 1. 先做能力預檢（唯一允許的前置工作）

在理解、拆解或執行使用者的實質任務前，先檢查目前執行環境實際暴露的能力，不要依賴模型記憶或假設：

1. 在可觀察時，確認主線程使用 `gpt-5.6-sol`。若已知不是 Sol，套用完整 hard stop 並請使用者切換；若無法觀察，不要僅因此阻塞。
2. 確認子代理可明確指定 `gpt-5.6-terra` 與 `gpt-5.6-luna`。
3. 確認兩者都可明確指定 `max` reasoning effort。
4. 若工具宣告支援、但實際啟動遭模型或 reasoning 相容性拒絕，將該能力視為不可用。
5. 只有主線程 Sol（在可觀察時）、`Terra Max` 與 `Luna Max` 都確認可用後，才能進入第 2 節。

不要把 App 的 model picker 是否勾選 Max 當成能力預檢條件。該選項只控制 composer 中是否顯示 Max；custom agent 檔案中的 `model_reasoning_effort = "max"` 會直接指定 spawned session 的 reasoning。以 subagent 工具宣告與實際啟動結果作為判準。

### 任一必要能力不可用：完整 hard stop

若缺少主線程 Sol、`Terra Max`、`Luna Max` 或其中任一必要模型／reasoning 組合，立即停止整個工作流，不只是停止委派。除確認能力與提供開啟說明外，不得執行任何工具或推進實質任務，包括：

- 不讀取或盤點專案、程式碼、文件與外部資料。
- 不進行架構設計、法遵邊界、需求拆解、風險分析或測試規劃。
- 不建立任何替代子代理，也不讓 Sol 單獨先做可安全推進的主線工作。
- 不自行降級成 `high`、`xhigh` 或其他模型／reasoning 設定。

不得使用「這不妨礙 Sol 主線程先做……」之類的例外。此 skill 啟用期間，缺少必要能力就是完整停止條件。只有使用者明確取消此 skill 或明確改變所需模型組合時，才能採用其他工作流。

唯一允許的修復例外：若使用者明確要求 AI 協助開啟必要能力，AI 可以指引 App 設定；若目前環境提供桌面控制能力，也可以直接開啟 Codex App 設定並勾選必要選項。AI 也可以建立、檢查與驗證 `~/.codex/agents/`／`.codex/agents/` 中的 Luna Max 或 Terra Max custom agent 設定。此例外不得延伸到讀取原專案、分析原任務或推進任何主線工作。

### Luna Max 不可用時的必要回覆

準確指出缺少的是 `gpt-5.6-luna`、Luna 的 `max` reasoning，或兩者，然後提供以下步驟並結束回覆：

1. 在 Codex 桌面 App 找到輸入框下方的 model／reasoning 控制。
2. 開啟 **Advanced**，確認可以選擇 `gpt-5.6-luna`。若 Max 已顯示，可以在此手動確認；若 Max 被 model picker 隱藏，不要僅因此判定 Luna Max 不可用，繼續以 custom agent 設定、subagent 工具宣告與實際啟動結果確認。
3. 若為了確認而將主線程切到 Luna，確認完成後切回 `gpt-5.6-sol`；Luna 必須作為 subagent，不得取代 Sol 主線程。
4. 將 App 的 Max 顯示設定視為 optional。只有使用者想在主 task 的 composer 手動選擇 Max 時，才開啟 **Settings**（macOS：Cmd+,；Windows：Ctrl+,），在 **Configuration** 或模型／推理設定中勾選 Max。若使用者明確要求 AI 協助且目前環境提供桌面控制能力，可以直接替使用者勾選；若無法控制 App，再請使用者手動處理。Max 未勾選不得單獨觸發 hard stop，也不影響 custom agent 檔案明確指定的 `model_reasoning_effort = "max"`。
5. 若 Advanced 中沒有 Luna，檢查帳號方案、工作區管理員的模型政策，以及目前 provider 是否提供 `gpt-5.6-luna`。明確說明 skill 本身無法解鎖未提供的模型。
6. 若 Luna 與 Max 在帳號中可用，但尚未定義專用 subagent，指示使用者在 `~/.codex/agents/luna-max.toml`（個人）或 `.codex/agents/luna-max.toml`（專案）建立：

```toml
name = "luna_max"
description = "Dedicated Luna Max worker for clear, bounded, repeatable tasks."
model = "gpt-5.6-luna"
model_reasoning_effort = "max"
developer_instructions = """
Handle only clear, bounded, repeatable tasks. Stay within the assigned scope, verify the result, and return concise evidence to the Sol main thread.
"""
```

此設定只會指定 subagent 的模型與 reasoning，不會授予帳號原本沒有的模型權限。

7. 完成後回到此 task 重試；若能力清單沒有更新，開啟新 task 再觸發 `$orchestrate-sol-terra-luna`。

依設定狀態選擇結尾，不要無條件重複同一句：

- **尚未由 AI 建立或驗證設定**：最後一行寫成：

  > 完成後請回覆：**Luna Max 已開啟**。若希望 AI 協助設定，請回覆：**請 AI 幫我設定 Luna Max**。

- **AI 已建立並驗證 `luna-max.toml`，但目前 task 尚未重新載入能力**：不要要求使用者先在 App 勾選 Max，也不要再提示「請 AI 幫我設定」。最後一行改成：

  > Luna Max 設定檔已完成。若能力清單仍未更新，請回覆：**請 AI 複製目前 task 並測試 Luna Max**。

不得在已成功建立並驗證設定後，再次要求使用者回覆「請 AI 幫我設定 Luna Max」。

在目前執行環境重新確認 `gpt-5.6-luna` 與 `max` 都可用以前，持續 hard stop。

### Terra Max 不可用

採用相同 hard-stop 原則，將上述步驟中的 Luna 改為 `gpt-5.6-terra`。不要把「支援 Max」解讀成「一定支援 Terra 或 Luna」，也不要未經使用者取消此 skill 就替換模型。

## 2. 建立任務地圖

由 Sol 先定義整體目標、限制、驗收條件與依賴關係，再依工作性質路由：

| 角色 | 分派內容 |
| --- | --- |
| Sol 主線程 | 模糊需求、架構與跨模組決策、任務拆解、衝突仲裁、最終驗收與整合輸出 |
| Terra Max | 困難但可封裝的模組分析、非平凡獨立實作、深度程式碼審查、安全或併發推理、複雜根因排查 |
| Luna Max | 程式碼搜尋與事實整理、測試執行與錯誤重現、日誌分類、機械式修改、規格非常明確的小功能與結構化摘要 |

不要為了使用子代理而委派微小工作。核心需求仍不清楚時，先由 Sol 解決，不要只靠提高子代理 reasoning effort。

## 3. 用明確契約委派

每個子代理 prompt 都要包含：

- **目標**：只描述一個可獨立完成的成果。
- **背景與輸入**：提供必要檔案、符號、錯誤或規格。
- **範圍**：列出允許讀寫的模組或檔案。
- **禁止事項**：指出不可變更的介面、行為與相鄰工作。
- **完成條件**：定義可檢查的 acceptance criteria。
- **驗證方式**：指定測試、型別檢查、lint、重現步驟或證據。
- **回報格式**：要求摘要、變更檔案、驗證結果、風險與未解問題。

預設採星狀協作：Sol 直接分派給 Terra 與 Luna，兩者直接回報 Sol。不要讓 Terra 再管理 Luna，除非 Terra 被授權完整承包一個獨立子系統。

## 4. 控制並行寫入

只把互不依賴的工作平行化。多個代理共用工作目錄時，為每個寫入任務指定互斥的檔案或模組所有權；範圍重疊時改成循序執行或使用隔離 worktree。

適合的交叉檢查模式：

- Terra 實作複雜功能，Luna 建立或執行針對性測試，Sol 最終整合。
- Luna 完成明確修改，Terra 做深度審查，Sol 判定是否接受。
- Luna 收集重現與日誌證據，Terra 分析根因，Sol 決定跨模組修正方案。

## 5. 驗收並整合

等待所有必要結果後，由 Sol：

1. 檢查子代理是否遵守範圍與完成條件。
2. 直接查看重要 diff、檔案引用、測試與重現證據，不只接受摘要。
3. 以程式碼與驗證結果解決互相矛盾的結論。
4. 執行適合風險程度的整合驗證。
5. 在最終輸出說明完成內容、驗證結果、剩餘風險，以及實際使用的 Sol、Terra Max 與 Luna Max 分工。

不要把同一代理的「已完成」當成最終批准；最終責任留在 Sol 主線程。
