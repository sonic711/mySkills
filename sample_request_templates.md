# Sample Request Templates（請求模板）

## 固定輸出規格
- 正式報告必須用繁體中文。
- 正式報告必須輸出到 `analysis_output/<project_name>/`。
- 正式報告必須是 `.md`。
- 分析前先查既有報告與 `analysis_registry/<project_name>/`；可重用時只補缺口，不重複讀完整程式。
- 若描述程式流程、交易流程、路由鏈或資料流，應補 Mermaid 流程圖。
- 若問題跨到不同系統、服務、外部 API、MQ、gRPC callback 或跨專案，正式報告必須補 Mermaid `sequenceDiagram` 系統架構交易時序圖。
- 若未特別指定，預設套用第十人原則審查；審查只供分析時使用，不輸出成正式報告章節。
- 正式報告最後的「未確認關鍵證據」只列 `Inferred`、`Unknown` 或其他未確認/待補查項；`Confirmed` 證據放在正文中。
- 正式報告要先有「快速結論」，讓非系統負責人可以快速知道功能或程式的作用、主要輸入、主要輸出、是否跨系統、是否執行 SQL。
- 正式報告要有「業務流程簡述」章節，從業務面向說明目的、參與對象、輸入、處理與結果，不在該章節堆技術細節。
- 正式報告要有「系統交易與資料流」與「交易資料格式」章節，標明系統之間資料來源、目的地、傳輸方式、格式、主要欄位與結果。
- 正式報告要有「SQL 與資料存取」章節；有 SQL/Mapper/Repository/SP/table 時必須列出，沒有找到也要標示 `未發現` 或 `未確認`。
- 技術細節、class、method、變數、完整 call tree 應放技術附錄，不要讓主體報告過長。

## 共用欄位
| 欄位 | 說明 |
|------|------|
| `project_name` | 專案名稱 |
| `project_path` | 專案實際路徑；不在預設位置時提供 |
| `branch` | 要分析的 Git 分支，例如 `uat`；提供時需先切分支、pull，並檢查 pull diff 影響 |
| `target_name` | 類別名、方法名、功能名或流程問題 |
| `issue_description` | 問題現象；問題導向調查時填 |
| `target_type` | `class` / `file` / `method` / `feature` / `flow` / `issue` |
| `analysis_focus` | `用途` / `業務流程簡述` / `系統交易與資料流` / `資料格式` / `SQL與資料存取` / `上下游` / `交易細節` / `依賴影響` / `跨專案比較` / `路由鏈` / `資料契約` / `異常流` / `流程圖` / `系統時序圖` / `實作細節` / `變數分析` / `方法分析` / `物件結構` / `完整流程` / `問題原因` / `寫入點` / `套件引用` / `邏輯分支` / `驗證方式` / `反證審查` / `精確度檢查` |
| `maintenance_facets` | `batch_scheduler` / `db_write` / `broadcast_event` / `external_contract` / `manual_rerun` / `cache_sync` |
| `scope_hint` | 模組、API、topic、table、workflow key、DTO、錯誤碼等線索 |
| `output_requirements` | 建議固定：`繁體中文, analysis_output/<project_name>/, md` |

## 報告重用原則
- 若提供 `branch`，先執行 Git preflight：切到分支、`git pull --ff-only`、比對 pull 前後 diff。
- 若 pull diff 不影響本次目標，忽略無關異動。
- 若 pull diff 影響本次目標且已有報告，更新舊文件。
- 若 pull diff 影響其他已分析程式，加入 `analysis_registry/<project_name>/impact_todo.md` 或更新對應舊文件。
- 正式報告只顯示分析基準 commit，不顯示分支名稱。
- 若更新舊報告且邏輯有變，正式報告要補「本次邏輯變更」章節。
- 先查 `analysis_registry/<project_name>/program_index.md` 與 `shared_components.md`。
- 若已有完整報告，採 `Reuse`。
- 若舊報告缺少新規格章節，例如「快速結論」、「業務流程簡述」、「系統交易與資料流」、「交易資料格式」或「SQL 與資料存取」，採 `Patch`。
- 若命中共用元件，採 `Reference Only`。
- 若沒有可用報告，採 `Analyze Fresh`。

## Facet 原則
- `analysis_focus` 決定你想看什麼。
- `maintenance_facets` 決定只在符合特徵時要補哪些維護附錄。
- 若不想讓報告偏向某種系統型態，優先用 `maintenance_facets`，不要把所有維護需求塞進固定模板。

## 第十人原則
若要提高精確度，請在 `analysis_focus` 補：`反證審查, 精確度檢查`。這只會影響分析與降級，不會新增正式報告章節。

## Templates
### Template 1：已知 class / service
```text
project_name: [專案名稱]
branch: uat
target_name: [類別名，例如 G0126RIM01Service]
target_type: class
analysis_focus: 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 異常流, 流程圖
scope_hint: 請先用業務角度說明這支 service 解決什麼事，再說明系統間交易資料格式與流向；若有 SQL/Mapper/Repository/SP/table 請標示出來
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 1B：跨系統交易時序圖
```text
project_name: [專案名稱]
branch: uat
target_name: [交易 / flow / service 名稱]
target_type: flow
analysis_focus: 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 異常流, 流程圖, 系統時序圖
scope_hint: 此流程會跨系統，請輸出系統架構交易時序圖，標出呼叫端、目前服務、DB/MQ、下游系統、callback 順序、資料格式與主要欄位
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 1A：先查既有報告再分析
```text
project_name: [專案名稱]
branch: uat
target_name: [類別名 / 功能名]
target_type: class
analysis_focus: 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 異常流
scope_hint: 請先查 analysis_registry 與既有報告；若已有完整報告就重用，若缺少快速結論、業務流程、資料流、資料格式或 SQL 章節只補缺口
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 1C：指定分支分析
```text
project_name: [專案名稱]
project_path: [專案路徑]
branch: uat
target_name: [類別名 / 功能名 / flow]
target_type: class
analysis_focus: 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 異常流, 流程圖, 系統時序圖
scope_hint: 請先切到 uat 並 git pull；報告只標明最新 commit，不顯示分支。檢查本次 pull 下來的 diff 是否影響此目標；若舊報告已存在且邏輯有變，請補「本次邏輯變更」。若 diff 影響已分析過的其他程式，請更新舊文件或加入 impact_todo.md
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 2：已知 method
```text
project_name: [專案名稱]
target_name: [方法名，例如 receive]
target_type: method
analysis_focus: 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 異常流, 流程圖
scope_hint: 請聚焦此 method 的實際入口、分流條件、資料格式、下游呼叫、SQL/資料存取與錯誤處理
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 3：單一程式深度解剖
```text
project_name: [專案名稱]
target_name: [類別名或檔名]
target_type: class
analysis_focus: 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 實作細節, 流程圖, 反證審查, 精確度檢查
scope_hint: 我不想看原始碼，請先用業務角度簡述它負責的流程，再說明資料格式、資料流向與 SQL；class/method/變數細節放技術附錄
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 4：功能流程分析
```text
project_name: [專案名稱]
target_name: [功能名，例如 開戶流程]
target_type: flow
analysis_focus: 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 異常流, 流程圖
scope_hint: 請先用業務角度說明整體流程的目的、參與對象、輸入與結果，再分析系統之間交易資料格式、流向、SQL 與錯誤處理
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 5：功能反查
```text
project_name: [專案名稱]
target_name: [功能問題，例如 log 集中化如何運作]
target_type: feature
analysis_focus: 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 依賴影響, 異常流, 流程圖
scope_hint: [關鍵字，例如 logback, appender, fluentd, elk, tracing]
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 6：API 反查內部實作
```text
project_name: [專案名稱]
target_name: [API path 或 API 名稱]
target_type: feature
analysis_focus: 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 異常流, 流程圖
scope_hint: [例如 /api/order/create, controller, request dto, response dto]
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 7：MQ / 事件反查
```text
project_name: [專案名稱]
target_name: [topic / event 名稱]
target_type: feature
analysis_focus: 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 異常流, 流程圖
scope_hint: [producer, consumer, payload, listener, retry, dead letter]
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 8：資料表 / SP 反查
```text
project_name: [專案名稱]
target_name: [table / stored procedure 名稱]
target_type: feature
analysis_focus: 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 依賴影響, 流程圖
scope_hint: [dao, repository, mapper xml, query method, service]
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 9：跨專案比較
```text
project_name: [專案A, 專案B]
target_name: [共同功能或共同 service 名稱]
target_type: feature
analysis_focus: 跨專案比較, 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 依賴影響, 流程圖
scope_hint: 請比較責任切分、系統資料流差異、資料契約差異、SQL/資料存取差異、風險差異
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 10：高精確度審查
```text
project_name: [專案名稱]
target_name: [類別名 / 功能名]
target_type: class
analysis_focus: 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 異常流, 流程圖, 反證審查, 精確度檢查
scope_hint: 請套用第十人原則，主動挑戰所有核心結論，降低任何過度自信敘述
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 11：條件式維護分析
```text
project_name: [專案名稱]
target_name: [類別名 / 功能名]
target_type: class
analysis_focus: 用途, 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 異常流, 流程圖, 反證審查, 精確度檢查
maintenance_facets: [依特徵選填，例如 batch_scheduler, db_write, external_contract]
scope_hint: 我的目標是靠文件做系統維護，請依程式特徵補對應 facet，不要把不相關的維護段落硬塞進報告
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 12：程式問題原因調查
```text
project_name: [專案名稱]
target_name: [選填：已知類別名 / 表名 / 交易名]
target_type: issue
issue_description: 同一個錯誤交易，error_log 的 msg 寫入值與 sys_posteifmsg 的 msg 值不一樣，請分析可能原因
analysis_focus: 業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 問題原因, 資料流, 寫入點, 套件引用, 邏輯分支, 驗證方式, 反證審查, 精確度檢查
scope_hint: error_log, sys_posteifmsg, msg, error code, transaction id；請追到兩邊 msg 的最後 setMsg、save/send 前轉換、utility 截斷/格式化與被呼叫 service 是否改值
maintenance_facets: db_write, external_contract
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

## 最短可用格式
```text
project_name: [專案名稱]
target_name: [目標名稱]
```

## 推薦格式
```text
project_name: [專案名稱]
project_path: [選填]
target_name: [目標名稱]
target_type: [class / file / method / feature / flow]
analysis_focus: [逗號分隔]
scope_hint: [選填]
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

## 使用建議
- 想減少重複讀程式：要求先查 `analysis_registry` 與既有報告。
- 想分析特定分支當下狀態：填 `branch: uat`，agent 會先 pull 再分析 diff 影響。
- 已知程式名：`target_type` 用 `class`、`file` 或 `method`。
- 已知功能、不知道程式：`target_type` 用 `feature`。
- 已知異常現象、不知道原因：`target_type` 用 `issue`，並填 `issue_description`。
- 欄位值不一致問題：要求追到兩邊最後賦值點，並做雙路徑對照。
- 想完整拆解單一程式：`analysis_focus` 加 `業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取, 實作細節`；技術細節會放附錄。
- 想讓不懂系統的人快速理解：`analysis_focus` 加 `業務流程簡述, 系統交易與資料流, 資料格式, SQL與資料存取`。
- 想讓流程更容易讀：`analysis_focus` 加 `流程圖`。
- 問題跨系統或跨服務：`analysis_focus` 加 `系統時序圖`，報告會輸出 Mermaid `sequenceDiagram`。
- 想讓文件可支撐維護：用 `maintenance_facets` 指定要補哪些條件附錄。
- 想提高精確度：`analysis_focus` 再加 `反證審查, 精確度檢查`。
- 想建立專案知識庫：確認每次分析後更新 `program_index.md` 與 `shared_components.md`。
