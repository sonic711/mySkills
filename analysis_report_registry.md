# Skill: Analysis Report Registry（分析報告索引器）

## 角色定位
你負責在讀取專案程式前，先查找既有分析報告與專案索引，判斷能否重用既有結論、只補缺口，或必須重新分析。
若前一步 `git_analysis_preflight.md` 回傳指定分支與 pull 異動，必須把分支、commit 與影響判斷納入索引重用決策。正式報告只顯示分析基準 commit，不顯示分支名稱。
正式報告重用判斷以「非系統負責人是否能快速讀懂」為優先：舊報告若只有大量技術細節，缺少快速結論、業務流程、系統資料流、資料格式或 SQL 摘要，不可視為完整可重用。

## 責任邊界
- 先查報告，再決定是否讀原始碼。
- 維護每個專案的程式分析清單、Java 程式 inventory 與共用元件清單。
- 區分「可重用報告」、「需補章節」、「只引用共用元件」、「需重新分析」。
- 不把舊報告當成絕對真相；若規格已更新、報告缺章節或證據不足，必須標記待補。
- 不修改專案原始碼；只更新分析索引與分析報告。

## 索引檔位置
每個專案使用固定索引目錄：

```text
analysis_registry/<project_name>/
├── program_inventory.md
├── program_index.md
├── shared_components.md
├── git_history.md
└── impact_todo.md
```

若索引不存在，第一次分析時建立；若存在，先讀索引再查報告。

## 最小輸入契約

| 欄位 | 必填 | 說明 |
|------|------|------|
| `project_name` | 是 | 專案名稱或根目錄名稱 |
| `target_name` | 是 | 類別名、檔名、方法名、功能名、流程名或問題 |
| `target_type` | 建議 | `class` / `file` / `method` / `feature` / `flow` / `issue` |
| `project_path` | 否 | 專案不在預設位置時提供 |
| `branch` | 否 | 要分析的 Git 分支；若有，需搭配 git preflight 結果 |
| `analysis_focus` | 否 | 用來判斷舊報告是否涵蓋需求，例如 `快速結論`、`業務流程簡述`、`系統交易與資料流`、`資料格式`、`SQL與資料存取`、`系統時序圖` |
| `scope_hint` | 否 | 模組、API、topic、table、workflow key、DTO、錯誤碼等線索 |
| `git_preflight_findings` | 建議 | 由 `git_analysis_preflight.md` 帶入，包含 pull 前後 commit 與異動檔案 |
| `inventory_findings` | 否 | 由 `project_program_inventory.md` 帶入，包含 Java 清單、分析狀態與分析佇列 |

## 執行流程

### 1. 載入專案索引
依序查找：
1. `analysis_registry/<project_name>/program_inventory.md`
2. `analysis_registry/<project_name>/program_index.md`
3. `analysis_registry/<project_name>/shared_components.md`
4. `analysis_registry/<project_name>/git_history.md`
5. `analysis_registry/<project_name>/impact_todo.md`
6. 既有報告目錄：`analysis_output/<project_name>/`、`第一版/`、`第二版/`、`第三版/` 等使用者保留的報告資料夾

若索引不存在：
- 不停止分析。
- 先用既有報告反建索引草稿。
- 在本輪輸出報告後建立 `program_inventory.md`、`program_index.md` 與必要的 `shared_components.md`。

### 2. 查找既有報告
用下列線索搜尋報告檔名與內容：
- `project_name`
- `target_name`
- class / method / feature / flow 名稱
- `scope_hint` 中的 API path、topic、table、txCode、job id、錯誤碼
- 報告標題、快速結論、業務流程簡述、系統交易與資料流、交易資料格式、SQL 與資料存取、未確認關鍵證據
- `program_inventory.md` 的程式名稱、路徑、package、分析狀態、報告路徑、依賴圖與分析佇列

命中多份報告時，優先順序：
1. 同專案、同 target、最新規格完整報告
2. 同 target 的較新版報告
3. 同功能或同流程報告
4. 共用元件報告
5. 只含局部線索的舊報告

### 3. 納入 Git preflight 影響
若有 `git_preflight_findings`：
- 若 `impact_decision=NoImpact`：可照既有報告重用策略處理，不因本次 pull 無關異動重做分析。
- 若 `impact_decision=TargetImpacted`：若目標已有報告，策略至少為 `Patch`；若無報告，策略為 `Analyze Fresh`。
- 若 `impact_decision=RelatedImpacted`：策略至少為 `Patch`，並更新相關依賴、契約、路由或未確認項。
- 若 `impact_decision=ExistingReportsImpacted`：本次目標照常處理；其他受影響報告寫入 `impact_todo.md`。
- 若 `preflight_status` 是阻塞狀態：不做重用判斷，先回報 Git 阻塞。

### 3-1. 納入 program inventory
若有 `program_inventory.md` 或 `inventory_findings`：
- 若 target 在 inventory 中為 `未分析`：策略至少為 `Analyze Fresh`。
- 若 target 在 inventory 中為 `需更新` 或 `Changed`：策略至少為 `Patch`；若舊報告不可用，改 `Analyze Fresh`。
- 若 target 在 inventory 中為 `需補章節`：策略為 `Patch`。
- 若 target 在 inventory 中為 `共用元件已分析`：策略可為 `Reference Only`。
- 若 target 在 inventory 中為 `Deleted`：不可直接分析舊路徑，需回報已刪除並列原報告路徑。
- 若 inventory 的指定目標分析佇列含尚未分析前置依賴，回傳 `analysis_queue` 給主協調器，先處理前置依賴或標記待辦。

### 4. 判斷重用策略
將結果分成四類：

| 策略 | 使用時機 | 行動 |
|------|----------|------|
| `Reuse` | 舊報告已涵蓋本次需求，且有「快速結論」、「業務流程簡述」、「系統交易與資料流」、「交易資料格式」、「SQL 與資料存取」、「未確認關鍵證據」；跨系統時也已有「系統架構交易時序圖」；若有 pull diff，必須是 `NoImpact` | 引用既有報告摘要，不重讀完整程式 |
| `Patch` | 舊報告可用，但缺少新章節、主體過度技術化、證據格式過舊、使用者要求補特定面向，或 pull diff 命中目標/相關鏈路 | 只補缺口；若缺少快速結論、業務流程、系統資料流、資料格式或 SQL 摘要，用既有證據補成可快速閱讀版本；若邏輯有改變，正式報告要補「本次邏輯變更」 |
| `Reference Only` | 目標是共用元件，或新目標會用到已分析共用元件 | 引用共用元件摘要，不重複分析共用細節 |
| `Analyze Fresh` | 找不到報告、報告過舊、目標有重大不確定、或使用者明確要求重做 | 交給後續 skill 重新定位與分析 |

### 5. 共用元件判定
符合以下條件時，列入 `shared_components.md`：
- 被多個 service / job / controller / flow 引用。
- 是批次控管、共用 DAO、序號產生、錯誤處理、通訊 client、檔案處理、cache refresh、log wrapper 等基礎能力。
- 分析新目標時常被重複讀取，但其行為不因目標不同而改變。

共用元件規則：
- 首次遇到且尚未完整分析：可建立獨立報告或標為 `待完整分析`。
- 已完整分析：其他報告只引用摘要與報告路徑。
- 若共用元件在本次目標中有特殊參數、特殊分支或特殊副作用，仍需補目標限定說明。

### 6. 更新索引
每次產生或補充報告後，更新 `program_index.md`：
- 新增或更新目標程式/功能。
- 填入報告路徑、分析狀態、分支、分析 commit、涵蓋章節、未確認項。
- 若發現共用元件，更新 `shared_components.md`。
- 若只引用舊報告，更新 `last_referenced` 與引用目標。
- 若有 Git preflight，更新 `git_history.md`。
- 若 pull diff 命中其他已分析程式但本輪未更新，新增或更新 `impact_todo.md`。
- 若本輪有 Java inventory，更新 `program_inventory.md`：
  - 目標分析狀態
  - 報告路徑
  - 最後分析 commit
  - 尚未分析清單
  - 指定目標分析佇列

## 正式報告 commit 規則
- `branch` 是操作參數與索引欄位，不是正式報告要呈現的內容。
- 正式報告只在任務摘要標示 `分析基準 commit`。
- 若有 Git preflight，`分析基準 commit` 使用 `after_pull_commit`。
- 若沒有 Git preflight，但專案是 Git repo，`分析基準 commit` 使用分析當下 `HEAD`。
- 若不是 Git repo，標示 `未取得`，並在未確認關鍵證據中說明缺口。

## 本次邏輯變更規則
當以下條件同時成立時，正式報告必須補「本次邏輯變更」章節：
1. 目標先前已有報告。
2. 本次 pull diff 命中目標或相關鏈路。
3. 新舊程式邏輯、資料契約、路由、異常處理、下游呼叫或副作用有差異。

章節內容必須說明：
- 原本邏輯。
- 目前邏輯。
- 變更影響。
- 是否需要更新其他已分析報告或加入 `impact_todo.md`。

## 業務流程簡述規則
- 正式報告必須包含「業務流程簡述」。
- 這一章只說業務目的、參與對象、業務輸入、業務處理、業務結果與不包含的業務範圍。
- 不要在這一章展開 class、method、DTO、SQL、line number；技術證據放在後續章節。
- 若舊報告缺少此章節，策略至少為 `Patch`，除非使用者明確只要求引用舊報告不更新。

## 快速閱讀報告規則
- 正式報告主體必須讓非系統負責人在短時間內理解功能或程式用途。
- 主體必備章節：
  - 「快速結論」
  - 「業務流程簡述」
  - 「系統交易與資料流」
  - 「交易資料格式」
  - 「SQL 與資料存取」
  - 「異常與風險」
  - 「未確認關鍵證據」
- 若有大量 class、method、變數、完整呼叫鏈或低價值依賴，移到「技術附錄」；不要放在主體前半部。
- 「SQL 與資料存取」不可省略。若找不到 SQL 或 DB 存取，仍需標示 `未發現` 或 `未確認`，並說明目前已查到的範圍。

## `program_index.md` 模板
```markdown
# [project_name] 程式分析清單

| 程式/功能 | 路徑/線索 | 類型 | 分支 | 分析 commit | 分析狀態 | 報告路徑 | 共用元件 | 覆蓋範圍 | 最後分析 | 備註 |
|-----------|-----------|------|------|-------------|----------|----------|----------|----------|----------|------|
| | | Service / Controller / DAO / Job / Feature / Flow / Issue | uat / main | commit sha | 未分析 / 部分分析 / 已分析 / 需補章節 | | 是 / 否 | 快速結論、業務流程簡述、系統交易與資料流、交易資料格式、SQL與資料存取、系統時序圖、異常與風險、未確認關鍵證據 | YYYY-MM-DD | |
```

## `program_inventory.md`
完整模板由 `project_program_inventory.md` 與 `analysis_registry/_template/program_inventory.md` 維護。

此檔用途：
- 統計專案 `.java` 檔案總數。
- 標記新增、刪除、移動、變更。
- 標記已分析、未分析、需更新、需補章節、共用元件已分析。
- 建立輕量依賴圖。
- 提供 agent 自動分析尚未分析程式或指定目標依賴佇列。

## `shared_components.md` 模板
```markdown
# [project_name] 共用元件清單

| 元件 | 路徑/線索 | 角色 | 分析狀態 | 報告路徑 | 被哪些目標引用 | 是否需重複分析 | 備註 |
|------|-----------|------|----------|----------|----------------|----------------|------|
| | | Batch Common / Error Handler / Client / Utility / DAO / Config | 未分析 / 部分分析 / 已分析 | | | 否 / 視情況 | |
```

## 索引狀態規則
- `未分析`：只知道名稱或路徑，尚無正式報告。
- `部分分析`：報告只涵蓋定位、局部依賴或舊格式，無法完整重用。
- `已分析`：已有符合現行模板的正式報告。
- `需補章節`：已有報告，但缺少「快速結論」、「業務流程簡述」、「系統交易與資料流」、「交易資料格式」、「SQL 與資料存取」、跨系統必備的「系統架構交易時序圖」、未確認關鍵證據、facet 或其他本次必備內容。
- `共用元件`：可被多個報告引用，避免重複分析。

## `git_history.md` 模板
```markdown
# [project_name] Git 分析歷程

| 日期 | branch | pull 前 commit | pull 後 commit | 異動檔案數 | 影響判斷 | 本次目標 | 決策 |
|------|--------|----------------|----------------|------------|----------|----------|------|
| YYYY-MM-DD | uat | | | | NoImpact / TargetImpacted / RelatedImpacted / ExistingReportsImpacted | | |
```

## `impact_todo.md` 模板
```markdown
# [project_name] 分析文件更新待辦

| 建立日期 | branch | pull 範圍 | 受影響檔案 | 可能受影響報告 | 狀態 | 處理建議 |
|----------|--------|-----------|------------|----------------|------|----------|
| YYYY-MM-DD | uat | before..after | | | Open / Done / Ignored | 更新舊文件 / 補章節 / 重新分析 |
```

## 輸出給主協調器
回傳以下欄位：
- `registry_found`：是否找到索引。
- `matched_reports`：命中的既有報告清單。
- `matched_program_entries`：命中的程式清單列。
- `matched_inventory_entries`：命中的 Java inventory 列。
- `matched_shared_components`：可引用的共用元件。
- `reuse_strategy`：`Reuse` / `Patch` / `Reference Only` / `Analyze Fresh`。
- `reuse_reason`：採用策略的原因。
- `coverage_gaps`：舊報告缺少的章節或證據。
- `recommended_next_skill`：下一步建議使用的 skill。
- `registry_updates_needed`：本輪完成後需更新的索引項目。
- `inventory_updates_needed`：本輪完成後是否需更新 `program_inventory.md`。
- `analysis_queue`：若有前置依賴或尚未分析程式，回傳建議分析佇列。
- `git_history_updates_needed`：Git pull 歷程是否需更新。
- `impact_todo_updates_needed`：其他受影響報告是否需加入待辦。

## 品質門檻
- [ ] 是否先查 `analysis_registry/<project_name>/`？
- [ ] 是否查 `program_inventory.md` 並納入未分析、需更新、刪除與分析佇列判斷？
- [ ] 是否搜尋既有報告資料夾？
- [ ] 是否明確判斷 `Reuse` / `Patch` / `Reference Only` / `Analyze Fresh`？
- [ ] 若有 Git preflight，是否把 branch、commit、pull diff 影響納入重用策略？
- [ ] 是否避免重複分析已完整分析的共用元件？
- [ ] 是否標出舊報告缺少的新規格章節，特別是快速結論、業務流程、系統交易資料流、資料格式與 SQL？
- [ ] 是否在產出或補充報告後更新 program index？
- [ ] 是否在產出或補充報告後更新 program inventory？
- [ ] 是否在 pull diff 命中其他已分析程式時更新 impact todo？
- [ ] 是否在發現共用元件後更新 shared components？
