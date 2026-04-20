# mySkills

此 repo 用來管理多專案程式分析規格。內容只放 `.md` 文件，不放專案程式。

## 核心原則
- 分析任務採唯讀模式：只讀專案與 skill 文件，只輸出報告。
- 正式報告必須用繁體中文、Markdown、`.md` 副檔名。
- 正式報告固定輸出到 `analysis_output/<project_name>/`。
- 每個結論都要標示 `Confirmed`、`Inferred`、`Unknown`。
- 正式報告輸出前，必須經過第十人原則審查。

## 主要文件
- `main_orchestrator.md`：主路由、流程、輸入輸出規格。
- `project_navigator.md`：定位專案、模組、目標與入口線索。
- `dependency_mapper.md`：分析入站、出站、build/config、DB 依賴。
- `inter_service_communication.md`：分析上游、下游、路由鏈、資料契約、異常流。
- `feature_capability_mapper.md`：由功能反查對應程式與元件群。
- `implementation_deep_dive.md`：拆解單一程式的變數、方法、流程與資料結構。
- `roleIdentity_synthesizer.md`：整合定位、依賴、通訊，輸出角色與風險。
- `tenth_man_auditor.md`：反證、挑錯、降級、精確度審查。
- `sample_request_templates.md`：常用請求模板。

## 支援的分析模式
1. 已知程式：分析用途、上下游、交易細節、依賴影響。
2. 已知程式：做深度實作解剖，不看原始碼也能理解內容。
3. 已知功能：反查對應程式、設定、資料流，再做完整分析。
4. 高精確度模式：在正式輸出前加做第十人原則審查。

## 輸入最小格式
```text
project_name: 專案名稱
target_name: 類別名 / 方法名 / 功能名
target_type: class / file / method / feature / flow
analysis_focus: 用途, 上下游, 交易細節, 路由鏈, 資料契約, 異常流
scope_hint: 其他線索
output_requirements: 繁體中文, analysis_output/<project_name>/, md
```

## 常見提問
### 1. 已知程式
```text
project_name: billing-core
target_name: G0126RIM01Service
target_type: class
analysis_focus: 用途, 上下游, 交易細節, 路由鏈, 資料契約, 異常流
```

### 2. 已知功能
```text
project_name: payment-platform
target_name: log 集中化如何運作
target_type: feature
analysis_focus: 用途, 上下游, 交易細節, 依賴影響, 路由鏈, 資料契約, 異常流
scope_hint: logback, appender, fluentd, elk, tracing
```

### 3. 單一程式深度解剖
```text
project_name: account-service
target_name: UserSyncService.java
target_type: file
analysis_focus: 實作細節, 變數分析, 方法分析, 物件結構, 完整流程
scope_hint: 我不想看原始碼，請完整拆解
```
