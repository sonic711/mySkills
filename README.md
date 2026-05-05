# mySkills

此 repo 用來管理多專案程式分析規格。內容只放 `.md` 文件，不放專案程式。

## 核心原則
- 分析任務採唯讀模式：只讀專案與 skill 文件，只輸出報告。
- 分析前先查既有報告與專案索引；若已有完整報告或共用元件報告，優先重用，只補缺口。
- 正式報告必須用繁體中文、Markdown、`.md` 副檔名。
- 正式報告固定輸出到 `analysis_output/<project_name>/`。
- 每個專案的分析索引固定輸出到 `analysis_registry/<project_name>/`。
- 描述程式流程、交易流程、路由鏈或資料流時，優先補 Mermaid 流程圖。
- 若問題跨到不同系統、服務、外部 API、MQ、gRPC callback 或跨專案，正式報告必須補 Mermaid `sequenceDiagram` 系統架構交易時序圖。
- 每個結論都要能區分 `Confirmed`、`Inferred`、`Unknown`；已確認證據放在主體段落，最後「未確認關鍵證據」只列未確認或待補查項。
- 正式報告輸出前，必須經過第十人原則審查；審查只供分析時使用，不輸出成正式報告章節。
- 正式報告要有「請求到回應完整說明」章節，白話說明從收到請求到回應結果中間做了什麼。
- 若目標是系統維護，採 facet 機制補強，只在符合特徵時追加對應附錄。

## 主要文件
- `main_orchestrator.md`：主路由、流程、輸入輸出規格。
- `analysis_report_registry.md`：查找既有報告、維護程式分析清單與共用元件清單。
- `conditional_maintenance_facets.md`：條件式維護附錄與觸發規則。
- `code_issue_investigator.md`：從異常現象反查功能、套件、資料流、寫入點與可能原因。
- `project_navigator.md`：定位專案、模組、目標與入口線索。
- `dependency_mapper.md`：分析入站、出站、build/config、DB 依賴。
- `inter_service_communication.md`：分析上游、下游、路由鏈、資料契約、異常流。
- `feature_capability_mapper.md`：由功能反查對應程式與元件群。
- `implementation_deep_dive.md`：拆解單一程式的變數、方法、流程與資料結構。
- `roleIdentity_synthesizer.md`：整合定位、依賴、通訊，輸出角色與風險。
- `tenth_man_auditor.md`：反證、挑錯、降級、精確度審查。
- `sample_request_templates.md`：常用請求模板。

## 支援的分析模式
1. 報告重用模式：先查既有報告與專案索引，判斷 `Reuse` / `Patch` / `Reference Only` / `Analyze Fresh`。
2. 已知程式：分析用途、上下游、交易細節、依賴影響。
3. 已知程式：做深度實作解剖，不看原始碼也能理解內容。
4. 已知功能：反查對應程式、設定、資料流，再做完整分析。
5. 高精確度模式：在正式輸出前加做第十人原則審查，並把審查結果整合成正文保留語句或未確認項。
6. 維護模式：用 facet 補強排查、重跑、修資料與回歸驗證，但不把所有報告寫成同一型。
7. 問題調查模式：由異常現象反查相關程式、套件引用、寫入點與原因假說。

## 分析索引
- `analysis_registry/<project_name>/program_index.md`：記錄哪些程式、功能或流程已產生報告。
- `analysis_registry/<project_name>/shared_components.md`：記錄哪些元件是共用元件，後續報告只引用摘要，不重複分析。
- `analysis_registry/_template/`：新專案索引模板。

## 輸入最小格式
```text
project_name: 專案名稱
target_name: 類別名 / 方法名 / 功能名
target_type: class / file / method / feature / flow
analysis_focus: 用途, 上下游, 交易細節, 路由鏈, 資料契約, 請求到回應, 異常流, 流程圖, 系統時序圖
scope_hint: 其他線索
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

## 維護 Facets
- `batch_scheduler`：批次、排程、手動觸發、重跑風險
- `db_write`：資料寫入矩陣、key、修復注意事項
- `broadcast_event`：廣播/事件通知矩陣
- `external_contract`：外部 request/response 契約與成功條件
- `manual_rerun`：補跑、重送、人工重跑
- `cache_sync`：快取/同步刷新驗證

## 常見提問
### 1. 已知程式
```text
project_name: billing-core
target_name: G0126RIM01Service
target_type: class
analysis_focus: 用途, 上下游, 交易細節, 路由鏈, 資料契約, 請求到回應, 異常流, 流程圖, 系統時序圖
```

### 2. 已知功能
```text
project_name: payment-platform
target_name: log 集中化如何運作
target_type: feature
analysis_focus: 用途, 上下游, 交易細節, 依賴影響, 路由鏈, 資料契約, 請求到回應, 異常流, 流程圖, 系統時序圖
scope_hint: logback, appender, fluentd, elk, tracing
```

### 3. 單一程式深度解剖
```text
project_name: account-service
target_name: UserSyncService.java
target_type: file
analysis_focus: 實作細節, 變數分析, 方法分析, 物件結構, 完整流程, 請求到回應, 流程圖
scope_hint: 我不想看原始碼，請完整拆解
```

### 4. 程式問題原因調查
```text
project_name: payment-core
target_type: issue
issue_description: 同一個錯誤交易，error_log.msg 與 sys_posteifmsg.msg 不一致，請分析可能原因
analysis_focus: 問題原因, 資料流, 寫入點, 套件引用, 邏輯分支, 驗證方式, 請求到回應
scope_hint: error_log, sys_posteifmsg, msg, transaction id；請追到最後 setMsg、save/send 前轉換與 utility 截斷/格式化
maintenance_facets: db_write, external_contract
```

### 5. 先查既有報告與索引
```text
project_name: fsap-common-service
target_name: BT908Service
target_type: class
analysis_focus: 用途, 上下游, 請求到回應, 異常流, 系統時序圖
scope_hint: 請先查既有報告與 program_index；若已有完整報告就重用，只補缺少的章節
```
