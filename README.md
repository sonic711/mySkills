# mySkills

這個 repo 是一套以 Markdown 文件為核心的分析規格庫，用來協助我對不同專案做程式、功能、流程與系統能力分析。

這個 repo 的重點不是放程式碼，而是放分析方法、輸入模板、輸出格式與技能規則。未來我只要提供專案名稱、程式名稱、方法名稱，或直接描述一個系統功能，就可以依照這套規格產出正式分析報告。

## Repo 目的

- 提供一套可重複使用的分析流程，避免每次分析都重新定義格式與邏輯。
- 支援從「已知程式」切入，也支援從「已知功能，不知道程式在哪裡」切入。
- 統一分析報告的輸出格式、證據等級、語言與存放位置。
- 讓分析結果可以沉澱為正式文件，而不是只停留在對話或臨時筆記。

## 主要檔案

| 檔案 | 用途 |
|------|------|
| `main_orchestrator.md` | 主協調器，定義整體分析流程、任務分類、輸出規格與限制 |
| `project_navigator.md` | 專案導航，負責定位專案、模組、檔案、層級與路由線索 |
| `dependency_mapper.md` | 依賴分析，負責入站/出站依賴、DB 鏈、build/config 關聯 |
| `inter_service_communication.md` | 通訊分析，負責 API/gRPC/MQ/workflow/dispatcher 鏈路、契約與異常流 |
| `roleIdentity_synthesizer.md` | 角色綜合判斷，整理角色定位、風險、驗證重點 |
| `feature_capability_mapper.md` | 功能反查，支援從系統功能描述反查多支相關程式 |
| `implementation_deep_dive.md` | 深度實作解剖，支援單一程式的變數、方法、物件結構與完整流程分析 |
| `sample_request_templates.md` | 可直接複製貼上的請求模板 |

## 支援的分析模式

### 1. 單一程式/類別分析
適合你已經知道要分析哪一支程式，例如：

- `OrderService.java`
- `G0126RIM01Service`
- `UserController.login`

這種模式會分析：

- 用途
- 上游來源
- 下游去向
- 路由鏈
- 資料契約
- 正常流與異常流
- 風險與影響

### 2. 單一程式深度實作分析
適合你已經知道某支程式，但不想自己看原始碼，希望直接知道：

- 每個成員變數做什麼
- 每個方法做什麼
- 方法內部流程如何推進
- 關鍵局部變數與中間物件是什麼
- 涉及哪些 DTO / VO / Entity / payload
- 完整功能流程與錯誤處理

這個模式對應 `implementation_deep_dive.md`。

### 3. 功能/流程分析
適合你知道一條業務流程，例如：

- 建立訂單流程
- 支付流程
- 退款流程

這種模式會從入口一路追到 DB、事件、外部系統與回應契約。

### 4. 功能反查/能力分析
適合你知道的是一個系統功能，但不知道對應哪些程式，例如：

- `log 集中化如何運作`
- `權限驗證如何運作`
- `審計軌跡如何落地`
- `配置中心如何生效`

這種模式會先反查多支相關程式與設定，再做完整分析。

## 正式輸出規格

所有正式分析報告都必須遵守以下規則：

- 使用繁體中文撰寫
- 以 Markdown `.md` 格式輸出
- 存放在 `analysis_output/<project_name>/`
- 建議檔名格式：

```text
analysis_output/<project_name>/<project_name>__<target_name>__analysis.md
```

## 唯讀分析原則

當任務目的是解析程式、方法、功能或系統能力時，預設使用唯讀分析模式：

- 不修改任何 skill 文件
- 不修改任何專案程式
- 不修改設定檔、SQL、XML、YAML、建置檔或測試檔
- 唯一允許的輸出，是正式分析報告 `.md`

也就是說，這套規格的核心用途是「讀取、分析、輸出報告」，不是直接修改被分析的專案。

## 證據規則

分析報告中的結論必須區分證據等級：

- `Confirmed`：可由程式碼、設定、SQL、註解、路由、topic、Bean 定義直接驗證
- `Inferred`：依命名、位置、慣例或局部片段推定
- `Unknown`：目前找不到足夠資訊

如果分析涉及 DB 或 Stored Procedure，會盡量補出：

- Service method
- DAO / Repository method
- SQL / XML / Mapper id
- Table / View / Stored Procedure 名稱

## 建議發問方式

### 已知程式

```text
project_name: project-a
target_name: OrderService.java
target_type: file
analysis_focus: 用途, 上下游, 交易細節, 路由鏈, 資料契約, 異常流
scope_hint: order-service module
output_requirements: 繁體中文, analysis_output/<project_name>/ 目錄, md 格式
```

### 已知功能，不知道程式

```text
project_name: project-a
target_name: log 集中化如何運作
target_type: feature
analysis_focus: 用途, 上下游, 交易細節, 依賴影響, 路由鏈, 資料契約, 異常流
scope_hint: logback, appender, fluentd, elk, opensearch, tracing, logging config
output_requirements: 繁體中文, analysis_output/<project_name>/ 目錄, md 格式
```

### 已知程式，想知道完整實作

```text
project_name: project-a
target_name: OrderService.java
target_type: file
analysis_focus: 實作細節, 變數分析, 方法分析, 物件結構, 完整流程
scope_hint: 我不想看原始碼，請完整拆解每個成員變數、每個方法、關鍵局部變數與物件結構
output_requirements: 繁體中文, analysis_output/<project_name>/ 目錄, md 格式
```

## 適合的使用情境

- 新接手陌生專案時快速建立理解
- 釐清某支程式在整體流程中的角色
- 釐清某個功能涉及哪些模組與設定
- 比較不同專案對同一功能的實作差異
- 在不直接閱讀大量原始碼的前提下，先建立全局理解

## 注意事項

- 這個 repo 預期只追蹤 Markdown 文件
- 若後續新增新的分析模式，優先以新增 `.md` skill 或擴充主協調器為主
- 若分析對象是大型專案，建議先從功能或模組邊界縮小範圍

## 下一步

可直接從 `sample_request_templates.md` 挑選模板開始使用，或讓主協調器依需求路由到對應技能。
