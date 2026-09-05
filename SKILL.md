---
name: three-tier-agent-orchestrator
description: "讓 GPT-6 Astra 留在主線程（main thread）擔任總指揮，將困難但邊界清楚的工作交給 GPT-5.6 Sol Mid，將清楚、可重複的工作交給 GPT-5.6 Luna Max。當使用者要求 Astra 總指揮、Sol Mid 或 Luna Max subagent、分層 multi-agent coding、平行 code review、模組分析、獨立功能實作、測試、debugging 或整合結果時使用；開始前檢查模型與 reasoning 是否可用，若任一必要模型或 reasoning 組合不可用，完整停止所有工作並告知如何開啟，確認可用後才執行。"
---

# Three-Tier Agent Orchestration

讓 Astra 保留主線程，負責理解目標、拆分任務、架構判斷、驗收與整合。把困難但邊界明確的工作交給 Sol Mid，把清楚、可重複且容易驗證的工作交給 Luna Max。

## 1. 先做能力預檢（唯一允許的前置工作）

在理解、拆解或執行使用者的實質任務前，先檢查目前執行環境實際暴露的能力，不要依賴模型記憶或假設：

1. 在可觀察時，確認主線程使用 `gpt-6-astra`。若已知不是 Astra，套用完整 hard stop 並請使用者切換；若無法觀察，不要僅因此阻塞。
2. 確認 subagent 工具可直接指定以下組合：
   - Sol Mid：`model = "gpt-5.6-sol"`、`reasoning_effort = "medium"`。
   - Luna Max：`model = "gpt-5.6-luna"`、`reasoning_effort = "max"`。
3. 若工具宣告支援、但實際啟動遭模型或 reasoning 相容性拒絕，將該能力視為不可用。
4. 只有主線程 Astra（在可觀察時）、Sol Mid 與 Luna Max 都確認可用後，才能進入第 2 節。

每次啟動 Sol Mid 或 Luna Max 時，直接傳入上述 `model` 與 `reasoning_effort`，不要依賴額外的 custom agent 設定檔。不要把 App 的 model picker 是否顯示 Max 當成能力預檢條件；以 subagent 工具宣告與實際啟動結果為準。

### 任一必要能力不可用：完整 hard stop

若缺少主線程 Astra、`Sol Mid`、`Luna Max` 或其中任一必要模型／reasoning 組合，立即停止整個工作流，不只是停止委派。除確認能力與提供開啟說明外，不得執行任何工具或推進實質任務，包括：

- 不讀取或盤點專案、程式碼、文件與外部資料。
- 不進行架構設計、法遵邊界、需求拆解、風險分析或測試規劃。
- 不建立任何替代子代理，也不讓 Astra 單獨先做可安全推進的主線工作。
- 不自行降級成其他模型／reasoning 設定。

不得使用「這不妨礙 Astra 主線程先做……」之類的例外。此 skill 啟用期間，缺少必要能力就是完整停止條件。只有使用者明確取消此 skill 或明確改變所需模型組合時，才能採用其他工作流。

唯一允許的修復例外：若使用者明確要求 AI 協助開啟必要能力，AI 可以指引 App 設定；若目前環境提供桌面控制能力，也可以直接協助檢查模型與 reasoning 選項。此例外不得延伸到讀取原專案、分析原任務或推進任何主線工作。

### 必要模型或 reasoning 不可用

準確指出缺少的是哪個模型或 reasoning 組合，並請使用者在 Codex App 的 model／reasoning 控制中確認：

- Sol Mid 需要 `gpt-5.6-sol` 與 `medium`。
- Luna Max 需要 `gpt-5.6-luna` 與 `max`。

若選項不存在，請使用者檢查帳號方案、工作區管理員模型政策與目前 provider。Skill 本身無法解鎖未提供的模型。能力重新可用前持續 hard stop。

## 2. 建立任務地圖

由 Astra 先定義整體目標、限制、驗收條件與依賴關係，再依工作性質路由：

| 角色 | 分派內容 |
| --- | --- |
| GPT-6 Astra 主線程 | 理解目標、拆分任務、架構決策、檢查結果與整合輸出 |
| GPT-5.6 Sol Mid | 困難但可封裝的模組分析、非平凡獨立實作、深度程式碼審查、安全或併發推理、複雜根因排查 |
| GPT-5.6 Luna Max | 程式碼搜尋與事實整理、測試執行與錯誤重現、日誌分類、機械式修改、規格非常明確的小功能與結構化摘要 |

不要為了使用子代理而委派微小工作。核心需求仍不清楚時，先由 Astra 解決，不要只靠提高子代理 reasoning effort。

## 3. 用明確契約委派

每個子代理 prompt 都要包含：

- **目標**：只描述一個可獨立完成的成果。
- **背景與輸入**：提供必要檔案、符號、錯誤或規格。
- **範圍**：列出允許讀寫的模組或檔案。
- **禁止事項**：指出不可變更的介面、行為與相鄰工作。
- **完成條件**：定義可檢查的 acceptance criteria。
- **驗證方式**：指定測試、型別檢查、lint、重現步驟或證據。
- **回報格式**：要求摘要、變更檔案、驗證結果、風險與未解問題。

預設採星狀協作：Astra 直接分派給 Sol Mid 與 Luna Max，兩者直接回報 Astra。不要讓 Sol Mid 再管理 Luna Max，除非 Sol Mid 被授權完整承包一個獨立子系統。

## 4. 控制並行寫入

只把互不依賴的工作平行化。多個代理共用工作目錄時，為每個寫入任務指定互斥的檔案或模組所有權；範圍重疊時改成循序執行或使用隔離 worktree。

適合的交叉檢查模式：

- Sol Mid 實作複雜功能，Luna Max 建立或執行針對性測試，Astra 最終整合。
- Luna Max 完成明確修改，Sol Mid 做深度審查，Astra 判定是否接受。
- Luna Max 收集重現與日誌證據，Sol Mid 分析根因，Astra 決定跨模組修正方案。

## 5. 驗收並整合

等待所有必要結果後，由 Astra：

1. 檢查子代理是否遵守範圍與完成條件。
2. 直接查看重要 diff、檔案引用、測試與重現證據，不只接受摘要。
3. 以程式碼與驗證結果解決互相矛盾的結論。
4. 執行適合風險程度的整合驗證。
5. 在最終輸出說明完成內容、驗證結果、剩餘風險，以及實際使用的 Astra、Sol Mid 與 Luna Max 分工。

不要把同一代理的「已完成」當成最終批准；最終責任留在 Astra 主線程。
