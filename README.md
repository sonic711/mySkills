# mySkills

這個 repo 放的是「專案程式分析用的 skill 規格文件」。用途是讓 AI agent 在分析 Java / Spring / 多模組 / 跨系統專案時，有固定流程可以遵循，產出一致、可維護、可重用的繁體中文分析報告。

## 安裝到要分析的專案

這些 skill 是純 Markdown 規格，任何可以讀檔、搜尋檔案、輸出 Markdown 的 AI agent 都可以使用，不限定 Codex。

建議安裝方式有兩種：

### 方式 A：放在共同 workspace 底下

適合 agent 可以讀取同一個 workspace 內多個資料夾的情境。把 `mySkills` 放在「要分析的專案旁邊」，不要混進被分析專案的原始碼目錄。

範例目錄：

```text
development/
├── mySkills/
└── fsap-adm/
```

如果尚未下載：

```bash
cd ~/Desktop/development
git clone https://github.com/sonic711/mySkills.git
```

如果已經有 repo，只要更新：

```bash
cd ~/Desktop/development/mySkills
git pull
```

使用時，agent 的工作目錄或可讀範圍必須包含：

- `mySkills/`
- 被分析專案，例如 `fsap-adm/`

如果 agent 有 sandbox 限制，建議把 workspace 開在共同上層：

```bash
cd ~/Desktop/development
```

### 方式 B：複製到被分析專案內

適合 agent 只能讀取單一專案目錄的情境。把 `mySkills` 複製或 clone 到被分析專案底下，例如：

```text
fsap-adm/
├── .analysis-skills/
│   └── mySkills/
└── ...
```

範例：

```bash
cd /path/to/fsap-adm
mkdir -p .analysis-skills
git clone https://github.com/sonic711/mySkills.git .analysis-skills/mySkills
```

這種方式的 prompt 要改用專案內路徑：

```text
請使用 .analysis-skills/mySkills/main_orchestrator.md 的規則分析目前專案。
```

## 權限與 sandbox 注意事項

不同 agent 的檔案權限模型不同。使用前請確認 agent 能同時：

- 讀取 skill 文件，例如 `mySkills/main_orchestrator.md`
- 讀取被分析專案原始碼
- 寫入報告輸出目錄，例如 `analysis_output/<project_name>/`
- 寫入分析索引，例如 `analysis_registry/<project_name>/`

如果 agent 無法跨目錄讀取，請使用「方式 B：複製到被分析專案內」。

如果 agent 無法寫入 repo 外部目錄，請把 `analysis_output/` 與 `analysis_registry/` 放在 agent 可寫的專案目錄內。

## 如何讓 AI agent 使用這些 skills

在任何 AI agent 對話中，直接指定 skill 文件路徑與要分析的專案路徑即可。

最短格式：

```text
請使用 /path/to/mySkills/main_orchestrator.md 的規則分析：

project_name: fsap-common-service
project_path: /path/to/fsap-adm/fsap-common-service
branch: uat
target_name: BT908Service
target_type: class
analysis_focus: 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 異常流, 流程圖, 系統時序圖
```

如果提供 `branch`，agent 需要先在 `project_path` 執行 Git 前置檢查：切到該分支、執行 `git pull --ff-only`，再用 pull 前後 commit diff 判斷這次拉下來的異動是否影響本次目標或既有分析文件。正式報告不需要顯示分支名稱，只需要標明分析當下最新的 commit。

如果不確定程式在哪裡，可以只給功能描述：

```text
請使用 /path/to/mySkills/main_orchestrator.md 的規則分析：

project_name: fsap-adm
project_path: /path/to/fsap-adm
target_name: log 集中化如何運作
target_type: feature
analysis_focus: 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 異常流, 流程圖, 系統時序圖
```

如果 agent 不支援自動載入外部文件，請先把 `main_orchestrator.md` 內容貼給 agent，並要求它依文件中列出的其他 skill 檔案逐步讀取。

## 建議輸入欄位

```text
project_name: 專案名稱
project_path: 專案實際路徑
branch: 要分析的 Git 分支，例如 uat
inventory_mode: scan / diff / plan_analysis / next_unanalyzed
target_name: 類別名 / 方法名 / 功能名 / 流程名 / 問題描述
target_type: class / file / method / feature / flow / issue
analysis_focus: 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 異常流, 流程圖, 系統時序圖
scope_hint: 額外線索，例如 API path、txCode、table、topic、錯誤碼
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

## 常用分析方式

### 已知程式

```text
project_name: billing-core
project_path: /path/to/billing-core
branch: uat
target_name: G0126RIM01Service
target_type: class
analysis_focus: 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 異常流, 流程圖, 系統時序圖
```

### 已知功能，不知道程式

```text
project_name: payment-platform
project_path: /path/to/payment-platform
target_name: log 集中化如何運作
target_type: feature
analysis_focus: 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 依賴影響, 異常流, 流程圖, 系統時序圖
scope_hint: logback, appender, tracing, log server
```

### 單一程式深度解剖

```text
project_name: account-service
project_path: /path/to/account-service
target_name: UserSyncService.java
target_type: file
analysis_focus: 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 實作細節, 流程圖
scope_hint: 我不想看原始碼，請先說明業務作用、資料流、資料格式與 SQL；技術細節放附錄
```

### 程式問題原因調查

```text
project_name: payment-core
project_path: /path/to/payment-core
target_type: issue
issue_description: 同一個錯誤交易，error_log.msg 與 sys_posteifmsg.msg 不一致，請分析可能原因
analysis_focus: 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 問題原因, 資料流, 寫入點, 套件引用, 邏輯分支, 驗證方式
scope_hint: error_log, sys_posteifmsg, msg, transaction id；請追到最後 setMsg、save/send 前轉換
maintenance_facets: db_write, external_contract
```

### 先查既有報告再分析

```text
project_name: fsap-common-service
project_path: /path/to/fsap-common-service
branch: uat
target_name: BT908Service
target_type: class
analysis_focus: 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 異常流, 系統時序圖
scope_hint: 請先查既有報告與 program_index；若已有完整報告就重用，只補缺少的章節，例如快速結論、業務流程、資料流、資料格式或 SQL
```

## 報告輸出位置

正式報告固定輸出到：

```text
analysis_output/<project_name>/
```

建議檔名：

```text
analysis_output/<project_name>/<project_name>__<target_name>__analysis.md
```

每個專案的分析索引固定輸出到：

```text
analysis_registry/<project_name>/program_inventory.md
analysis_registry/<project_name>/program_index.md
analysis_registry/<project_name>/shared_components.md
analysis_registry/<project_name>/git_history.md
analysis_registry/<project_name>/impact_todo.md
```

`program_inventory.md` 是專案 Java 程式清單，只統計 `.java`。它用來記錄 Java 檔案總數、已分析、未分析、新增、刪除、移動、需更新，以及輕量依賴圖與分析佇列。當你要 agent 自動分析尚未分析程式，或分析指定程式前先按依賴順序處理 B、C 等前置程式時，就使用這個檔案。

### 建立或更新 Java 程式清單

```text
請使用 /path/to/mySkills/main_orchestrator.md 的規則：

project_name: fsap-adm
project_path: /path/to/fsap-adm
inventory_mode: scan
scope_hint: 請掃描專案內所有 .java，建立或更新 analysis_registry/<project_name>/program_inventory.md，統計 Java 總數、已分析、未分析、新增、刪除、移動與需更新
```

### 檢查新增或刪除程式

```text
請使用 /path/to/mySkills/main_orchestrator.md 的規則：

project_name: fsap-adm
project_path: /path/to/fsap-adm
inventory_mode: diff
scope_hint: 請比對 program_inventory.md 與目前專案 .java，列出 Added、Deleted、Moved、Changed，並更新尚未分析清單
```

### 依指定程式的依賴順序安排分析

```text
請使用 /path/to/mySkills/main_orchestrator.md 的規則：

project_name: fsap-adm
project_path: /path/to/fsap-adm
target_name: AService.java
target_type: class
inventory_mode: plan_analysis
analysis_focus: 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取
scope_hint: 請先用 program_inventory.md 檢查 A 依賴的 B、C 是否已分析；若 B、C 尚未分析，先排入分析佇列，再分析 A 並整合結果
```

## 主要 skill 文件

- `main_orchestrator.md`：主流程。所有分析任務建議先使用它。
- `project_program_inventory.md`：建立 `.java` 程式清單、檢查新增/刪除/移動、標記未分析並產生依賴分析佇列。
- `analysis_report_registry.md`：先查既有報告與專案索引，減少重複讀程式。
- `project_navigator.md`：定位專案、模組、目標與入口線索。
- `feature_capability_mapper.md`：從功能描述反查程式與元件。
- `dependency_mapper.md`：分析入站、出站、build/config、DB 依賴。
- `inter_service_communication.md`：分析跨服務、API、gRPC、MQ、callback、交易鏈。
- `implementation_deep_dive.md`：拆解單一程式的變數、方法、流程與資料結構。
- `code_issue_investigator.md`：從異常現象追資料流、寫入點、最後賦值點與可能原因。
- `conditional_maintenance_facets.md`：依特徵補維護面向，例如重跑、DB 寫入、外部契約。
- `roleIdentity_synthesizer.md`：整合角色、重要性、風險與驗證重點。
- `tenth_man_auditor.md`：正式輸出前的內部反證審查。
- `sample_request_templates.md`：更多請求模板。

## 產出報告的固定要求

- 使用繁體中文。
- 使用 Markdown。
- 若輸入包含 `branch`，必須先切到該分支並執行 `git pull --ff-only`，以該分支 pull 後狀態作為分析基準。
- 正式報告不顯示分支名稱，只標明分析基準 commit。
- 若 pull 下來的異動不包含本次目標，忽略無關異動；若包含已分析過的其他程式，更新舊文件或加入 `impact_todo.md`。
- 若本次目標先前已分析過，且 pull diff 顯示邏輯有變，正式報告必須補「本次邏輯變更」，說明原本邏輯改成目前邏輯。
- 先查既有報告與 `analysis_registry`，可重用時只補缺口。
- 需要盤點或自動依序分析時，先更新 `program_inventory.md`；它只統計 `.java`，並作為尚未分析與依賴排序的依據。
- 每個結論要能區分 `Confirmed` / `Inferred` / `Unknown`。
- 最後只列「未確認關鍵證據」，不要重複列已確認證據清單。
- 必須先有「快速結論」，讓非系統負責人快速知道功能或程式的作用、主要輸入、主要輸出、是否跨系統、是否執行 SQL。
- 必須有「業務流程簡述」，用業務面向說明目的、參與對象、輸入、處理與結果，不在該章節堆技術細節。
- 必須有「系統交易與資料流」與「交易資料格式」，標明系統之間資料來源、目的地、傳輸方式、格式、主要欄位與結果。
- 必須有「SQL 與資料存取」；有 SQL/Mapper/Repository/SP/table 時必須列出，沒有找到也要標示 `未發現` 或 `未確認`。
- class、method、變數、完整 call tree 等技術細節應放在附錄，避免主體報告過長。
- 若目標是純 VO/DTO/Request/Response/Entity，只需列出被哪些程式使用、屬性與型別；只有 VO/DTO 內含 validation、轉換、格式化、預設值、衍生欄位等邏輯時，才補內部邏輯說明。
- 若跨系統、跨服務、MQ、gRPC callback、外部 API 或跨專案，必須輸出 Mermaid `sequenceDiagram` 系統架構交易時序圖。
- 若流程超過 3 個節點，優先補 Mermaid 流程圖。
- 第十人原則只作為內部審查使用，不輸出成正式報告章節。

## 更新 skills

在 `mySkills` repo 內更新：

```bash
cd /Users/sonic711/Desktop/development/mySkills
git pull
```

若在不同機器或不同 agent 使用，先 clone 這個 repo，再在 prompt 中指定 `main_orchestrator.md` 的實際路徑。
