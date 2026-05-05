# Skill: Analysis Report Registry（分析報告索引器）

## 角色定位
你負責在讀取專案程式前，先查找既有分析報告與專案索引，判斷能否重用既有結論、只補缺口，或必須重新分析。

## 責任邊界
- 先查報告，再決定是否讀原始碼。
- 維護每個專案的程式分析清單與共用元件清單。
- 區分「可重用報告」、「需補章節」、「只引用共用元件」、「需重新分析」。
- 不把舊報告當成絕對真相；若規格已更新、報告缺章節或證據不足，必須標記待補。
- 不修改專案原始碼；只更新分析索引與分析報告。

## 索引檔位置
每個專案使用固定索引目錄：

```text
analysis_registry/<project_name>/
├── program_index.md
└── shared_components.md
```

若索引不存在，第一次分析時建立；若存在，先讀索引再查報告。

## 最小輸入契約

| 欄位 | 必填 | 說明 |
|------|------|------|
| `project_name` | 是 | 專案名稱或根目錄名稱 |
| `target_name` | 是 | 類別名、檔名、方法名、功能名、流程名或問題 |
| `target_type` | 建議 | `class` / `file` / `method` / `feature` / `flow` / `issue` |
| `project_path` | 否 | 專案不在預設位置時提供 |
| `analysis_focus` | 否 | 用來判斷舊報告是否涵蓋需求 |
| `scope_hint` | 否 | 模組、API、topic、table、workflow key、DTO、錯誤碼等線索 |

## 執行流程

### 1. 載入專案索引
依序查找：
1. `analysis_registry/<project_name>/program_index.md`
2. `analysis_registry/<project_name>/shared_components.md`
3. 既有報告目錄：`analysis_output/<project_name>/`、`第一版/`、`第二版/`、`第三版/` 等使用者保留的報告資料夾

若索引不存在：
- 不停止分析。
- 先用既有報告反建索引草稿。
- 在本輪輸出報告後建立 `program_index.md` 與必要的 `shared_components.md`。

### 2. 查找既有報告
用下列線索搜尋報告檔名與內容：
- `project_name`
- `target_name`
- class / method / feature / flow 名稱
- `scope_hint` 中的 API path、topic、table、txCode、job id、錯誤碼
- 報告標題、目標定位、主要用途與角色、未確認關鍵證據

命中多份報告時，優先順序：
1. 同專案、同 target、最新規格完整報告
2. 同 target 的較新版報告
3. 同功能或同流程報告
4. 共用元件報告
5. 只含局部線索的舊報告

### 3. 判斷重用策略
將結果分成四類：

| 策略 | 使用時機 | 行動 |
|------|----------|------|
| `Reuse` | 舊報告已涵蓋本次需求，且有「請求到回應完整說明」、「未確認關鍵證據」；跨系統時也已有「系統架構交易時序圖」 | 引用既有報告摘要，不重讀完整程式 |
| `Patch` | 舊報告可用，但缺少新章節、證據格式過舊、或使用者要求補特定面向 | 只補缺口，避免重做全量分析 |
| `Reference Only` | 目標是共用元件，或新目標會用到已分析共用元件 | 引用共用元件摘要，不重複分析共用細節 |
| `Analyze Fresh` | 找不到報告、報告過舊、目標有重大不確定、或使用者明確要求重做 | 交給後續 skill 重新定位與分析 |

### 4. 共用元件判定
符合以下條件時，列入 `shared_components.md`：
- 被多個 service / job / controller / flow 引用。
- 是批次控管、共用 DAO、序號產生、錯誤處理、通訊 client、檔案處理、cache refresh、log wrapper 等基礎能力。
- 分析新目標時常被重複讀取，但其行為不因目標不同而改變。

共用元件規則：
- 首次遇到且尚未完整分析：可建立獨立報告或標為 `待完整分析`。
- 已完整分析：其他報告只引用摘要與報告路徑。
- 若共用元件在本次目標中有特殊參數、特殊分支或特殊副作用，仍需補目標限定說明。

### 5. 更新索引
每次產生或補充報告後，更新 `program_index.md`：
- 新增或更新目標程式/功能。
- 填入報告路徑、分析狀態、涵蓋章節、未確認項。
- 若發現共用元件，更新 `shared_components.md`。
- 若只引用舊報告，更新 `last_referenced` 與引用目標。

## `program_index.md` 模板
```markdown
# [project_name] 程式分析清單

| 程式/功能 | 路徑/線索 | 類型 | 分析狀態 | 報告路徑 | 共用元件 | 覆蓋範圍 | 最後分析 | 備註 |
|-----------|-----------|------|----------|----------|----------|----------|----------|------|
| | | Service / Controller / DAO / Job / Feature / Flow / Issue | 未分析 / 部分分析 / 已分析 / 需補章節 | | 是 / 否 | 用途、上下游、請求到回應、系統時序圖、異常流、依賴、未確認關鍵證據 | YYYY-MM-DD | |
```

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
- `需補章節`：已有報告，但缺少「請求到回應完整說明」、跨系統必備的「系統架構交易時序圖」、未確認關鍵證據、facet 或其他本次必備內容。
- `共用元件`：可被多個報告引用，避免重複分析。

## 輸出給主協調器
回傳以下欄位：
- `registry_found`：是否找到索引。
- `matched_reports`：命中的既有報告清單。
- `matched_program_entries`：命中的程式清單列。
- `matched_shared_components`：可引用的共用元件。
- `reuse_strategy`：`Reuse` / `Patch` / `Reference Only` / `Analyze Fresh`。
- `reuse_reason`：採用策略的原因。
- `coverage_gaps`：舊報告缺少的章節或證據。
- `recommended_next_skill`：下一步建議使用的 skill。
- `registry_updates_needed`：本輪完成後需更新的索引項目。

## 品質門檻
- [ ] 是否先查 `analysis_registry/<project_name>/`？
- [ ] 是否搜尋既有報告資料夾？
- [ ] 是否明確判斷 `Reuse` / `Patch` / `Reference Only` / `Analyze Fresh`？
- [ ] 是否避免重複分析已完整分析的共用元件？
- [ ] 是否標出舊報告缺少的新規格章節？
- [ ] 是否在產出或補充報告後更新 program index？
- [ ] 是否在發現共用元件後更新 shared components？
