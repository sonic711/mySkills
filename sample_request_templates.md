# Sample Request Templates（請求模板）

## 固定輸出規格
- 正式報告必須用繁體中文。
- 正式報告必須輸出到 `analysis_output/<project_name>/`。
- 正式報告必須是 `.md`。
- 若描述程式流程、交易流程、路由鏈或資料流，應補 Mermaid 流程圖。
- 若未特別指定，預設套用第十人原則審查。

## 共用欄位
| 欄位 | 說明 |
|------|------|
| `project_name` | 專案名稱 |
| `project_path` | 專案實際路徑；不在預設位置時提供 |
| `target_name` | 類別名、方法名、功能名或流程問題 |
| `target_type` | `class` / `file` / `method` / `feature` / `flow` |
| `analysis_focus` | `用途` / `上下游` / `交易細節` / `依賴影響` / `跨專案比較` / `路由鏈` / `資料契約` / `異常流` / `流程圖` / `實作細節` / `變數分析` / `方法分析` / `物件結構` / `完整流程` / `反證審查` / `精確度檢查` |
| `maintenance_facets` | `batch_scheduler` / `db_write` / `broadcast_event` / `external_contract` / `manual_rerun` / `cache_sync` |
| `scope_hint` | 模組、API、topic、table、workflow key、DTO、錯誤碼等線索 |
| `output_requirements` | 建議固定：`繁體中文, analysis_output/<project_name>/, md` |

## Facet 原則
- `analysis_focus` 決定你想看什麼。
- `maintenance_facets` 決定只在符合特徵時要補哪些維護附錄。
- 若不想讓報告偏向某種系統型態，優先用 `maintenance_facets`，不要把所有維護需求塞進固定模板。

## 第十人原則
若要提高精確度，請在 `analysis_focus` 補：`反證審查, 精確度檢查`。

## Templates
### Template 1：已知 class / service
```text
project_name: [專案名稱]
target_name: [類別名，例如 G0126RIM01Service]
target_type: class
analysis_focus: 用途, 上下游, 交易細節, 路由鏈, 資料契約, 異常流, 流程圖
scope_hint: 請說明這支 service 的用途、誰呼叫它、它呼叫誰、交易資料怎麼流動
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 2：已知 method
```text
project_name: [專案名稱]
target_name: [方法名，例如 receive]
target_type: method
analysis_focus: 用途, 上下游, 交易細節, 路由鏈, 資料契約, 異常流, 流程圖
scope_hint: 請聚焦此 method 的實際入口、分流條件、下游呼叫與錯誤處理
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 3：單一程式深度解剖
```text
project_name: [專案名稱]
target_name: [類別名或檔名]
target_type: class
analysis_focus: 實作細節, 變數分析, 方法分析, 物件結構, 完整流程, 流程圖, 反證審查, 精確度檢查
scope_hint: 我不想看原始碼，請完整拆解每個成員變數、每個方法、關鍵局部變數、物件結構與完整流程
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 4：功能流程分析
```text
project_name: [專案名稱]
target_name: [功能名，例如 開戶流程]
target_type: flow
analysis_focus: 用途, 上下游, 交易細節, 路由鏈, 資料契約, 異常流, 流程圖
scope_hint: 請分析整體流程涉及哪些核心服務、資料流、交易節點與錯誤處理
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 5：功能反查
```text
project_name: [專案名稱]
target_name: [功能問題，例如 log 集中化如何運作]
target_type: feature
analysis_focus: 用途, 上下游, 交易細節, 依賴影響, 路由鏈, 資料契約, 異常流, 流程圖
scope_hint: [關鍵字，例如 logback, appender, fluentd, elk, tracing]
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 6：API 反查內部實作
```text
project_name: [專案名稱]
target_name: [API path 或 API 名稱]
target_type: feature
analysis_focus: 用途, 上下游, 交易細節, 路由鏈, 資料契約, 異常流, 流程圖
scope_hint: [例如 /api/order/create, controller, request dto, response dto]
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 7：MQ / 事件反查
```text
project_name: [專案名稱]
target_name: [topic / event 名稱]
target_type: feature
analysis_focus: 用途, 上下游, 交易細節, 路由鏈, 資料契約, 異常流, 流程圖
scope_hint: [producer, consumer, payload, listener, retry, dead letter]
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 8：資料表 / SP 反查
```text
project_name: [專案名稱]
target_name: [table / stored procedure 名稱]
target_type: feature
analysis_focus: 用途, 上下游, 交易細節, 依賴影響, 資料契約, 流程圖
scope_hint: [dao, repository, mapper xml, query method, service]
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 9：跨專案比較
```text
project_name: [專案A, 專案B]
target_name: [共同功能或共同 service 名稱]
target_type: feature
analysis_focus: 跨專案比較, 用途, 上下游, 交易細節, 依賴影響, 流程圖
scope_hint: 請比較責任切分、上下游差異、資料契約差異、風險差異
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 10：高精確度審查
```text
project_name: [專案名稱]
target_name: [類別名 / 功能名]
target_type: class
analysis_focus: 用途, 上下游, 交易細節, 路由鏈, 資料契約, 異常流, 流程圖, 反證審查, 精確度檢查
scope_hint: 請套用第十人原則，主動挑戰所有核心結論，降低任何過度自信敘述
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

### Template 11：條件式維護分析
```text
project_name: [專案名稱]
target_name: [類別名 / 功能名]
target_type: class
analysis_focus: 用途, 上下游, 交易細節, 路由鏈, 資料契約, 異常流, 流程圖, 反證審查, 精確度檢查
maintenance_facets: [依特徵選填，例如 batch_scheduler, db_write, external_contract]
scope_hint: 我的目標是靠文件做系統維護，請依程式特徵補對應 facet，不要把不相關的維護段落硬塞進報告
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
- 已知程式名：`target_type` 用 `class`、`file` 或 `method`。
- 已知功能、不知道程式：`target_type` 用 `feature`。
- 想完整拆解單一程式：`analysis_focus` 加 `實作細節, 變數分析, 方法分析, 物件結構, 完整流程`。
- 想讓流程更容易讀：`analysis_focus` 加 `流程圖`。
- 想讓文件可支撐維護：用 `maintenance_facets` 指定要補哪些條件附錄。
- 想提高精確度：`analysis_focus` 再加 `反證審查, 精確度檢查`。
