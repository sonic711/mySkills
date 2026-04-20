# Sample Request Templates

## 用途
這份文件提供可直接複製貼上的請求模板，讓使用者以一致格式向 `main_orchestrator.md` 發出分析任務。欄位名稱對齊目前技能系統的輸入契約：

- `project_name`
- `project_path`
- `target_name`
- `target_type`
- `analysis_focus`
- `scope_hint`

若資訊不完整，也可先用簡化模板；但建議至少提供 `project_name` 與 `target_name`。

## 固定輸出規格
- 正式分析報告必須使用繁體中文。
- 正式分析報告必須輸出到 `analysis_output/<project_name>/` 目錄。
- 正式分析報告必須為 `.md` 檔案。
- 預設檔名格式建議為 `analysis_output/<project_name>/<project_name>__<target_name>__analysis.md`。

## 推薦欄位說明

| 欄位 | 必填 | 說明 |
|------|------|------|
| `project_name` | 是 | 專案名稱或根目錄名稱 |
| `project_path` | 否 | 專案不在預設位置時提供完整路徑 |
| `target_name` | 視情況 | 類別名、檔名、方法名、功能名、流程名 |
| `target_type` | 否 | `class` / `file` / `method` / `feature` / `flow` |
| `analysis_focus` | 否 | `用途` / `上下游` / `交易細節` / `依賴影響` / `跨專案比較` / `路由鏈` / `資料契約` / `異常流` |
| `scope_hint` | 否 | 模組、API 路徑、資料表、topic、workflow key、request header、response header 等線索 |

## Template 1: 單一程式完整分析

```text
project_name: project-a
target_name: OrderService.java
target_type: file
analysis_focus: 用途, 上下游, 交易細節, 依賴影響, 路由鏈, 資料契約, 異常流
scope_hint: order-service module
```

## Template 2: 單一類別簡化分析

```text
project_name: project-a
target_name: OrderService
```

## Template 3: 指定方法分析

```text
project_name: project-a
target_name: OrderService.createOrder
target_type: method
analysis_focus: 用途, 上下游, 交易細節, 路由鏈, 資料契約, 異常流
scope_hint: API create order
```

## Template 4: 功能流程分析

```text
project_name: project-a
target_name: 建立訂單流程
target_type: flow
analysis_focus: 用途, 上下游, 交易細節, 路由鏈, 資料契約, 異常流
scope_hint: create order, order submit, payment, inventory
```

## Template 4-1: 從系統功能反查相關程式

```text
project_name: project-a
target_name: log 集中化如何運作
target_type: feature
analysis_focus: 用途, 上下游, 交易細節, 依賴影響, 路由鏈, 資料契約, 異常流
scope_hint: logback, appender, fluentd, elk, opensearch, tracing, logging config
```

## Template 4-2: 從系統能力反查並找出多支核心程式

```text
project_name: project-a
target_name: 權限驗證如何運作
target_type: feature
analysis_focus: 用途, 上下游, 依賴影響, 路由鏈, 資料契約, 異常流
scope_hint: auth filter, interceptor, token, jwt, permission, role, security config
```

## Template 5: 專案結構導覽

```text
project_name: project-a
analysis_focus: 用途
scope_hint: 專案結構, 模組架構, 入口模組
```

## Template 6: 明確指定專案路徑

```text
project_name: legacy-order-system
project_path: /absolute/path/to/legacy-order-system
target_name: RefundService
target_type: class
analysis_focus: 用途, 上下游, 交易細節, 依賴影響, 路由鏈, 資料契約, 異常流
```

## Template 7: 比較兩個專案的同一功能

```text
project_name: project-a, project-b
target_name: create order
target_type: feature
analysis_focus: 跨專案比較, 用途, 上下游, 交易細節, 依賴影響, 路由鏈, 資料契約, 異常流
scope_hint: API submit order, payment, inventory, order persistence
```

## Template 8: 從 API 路徑反查

```text
project_name: project-a
target_name: /api/orders/create
target_type: feature
analysis_focus: 用途, 上下游, 交易細節, 路由鏈, 資料契約, 異常流
scope_hint: controller endpoint, create order API
```

## Template 9: 從 MQ / Event 反查

```text
project_name: project-a
target_name: order-created
target_type: feature
analysis_focus: 上下游, 交易細節, 依賴影響, 路由鏈, 資料契約, 異常流
scope_hint: kafka topic, producer, consumer, payment, notification
```

## Template 10: 從資料表或 Mapper 反查

```text
project_name: project-a
target_name: order_main
target_type: feature
analysis_focus: 上下游, 交易細節, 依賴影響
scope_hint: table, mapper, repository, insert, update, stored procedure
```

## Template 11: 專案剛導入時的 onboarding 請求

```text
project_name: project-a
analysis_focus: 用途
scope_hint: 先幫我盤點模組、主要入口、核心 service、外部整合點
```

## Template 12: 模糊查詢縮小範圍

```text
project_name: project-a
target_name: UserService
target_type: class
analysis_focus: 用途, 上下游
scope_hint: login, auth, token, member
```

## Template 13: 路由型/邊界型服務深度版
適合像 `G0126RIM01Service` 這種經過 dispatcher、吃 header、回固定 response 契約的服務。

```text
project_name: project-a
target_name: SomeGrpcOrRoutingService
target_type: class
analysis_focus: 用途, 上下游, 交易細節, 依賴影響, 路由鏈, 資料契約, 異常流
scope_hint: dispatcher, txCode, dscpt, request header, response header, error handling, SQL, stored procedure
```

## 建議使用習慣
- 若已知類別名，優先提供 `target_type: class` 或 `target_type: file`。
- 若要查完整交易流，`analysis_focus` 建議至少包含 `上下游, 交易細節`。
- 若目標是 gRPC、batch、MQ、workflow 或內部分流服務，建議再加 `路由鏈, 資料契約, 異常流`。
- 若你知道的是「系統功能」而不是程式名，請直接把功能句子放在 `target_name`，例如 `log 集中化如何運作`。
- 若只知道 API、topic、table、workflow key，也可以當成 `target_name` 或放進 `scope_hint`。
- 若要比較兩個專案，請在 `project_name` 一次列出多個專案。
- 若你希望直接產生正式報告檔，可在需求中明寫「請輸出到 `analysis_output/<project_name>/*.md`」，但目前規格已預設如此。

## 最短可用格式

```text
project_name: project-a
target_name: OrderService
```

## 建議輸入格式

```text
project_name: project-a
project_path: /absolute/path/to/project-a
target_name: OrderService.createOrder
target_type: method
analysis_focus: 用途, 上下游, 交易細節, 依賴影響, 路由鏈, 資料契約, 異常流
scope_hint: create order API, payment, inventory, order_main table, order-created topic
output_requirements: 繁體中文, analysis_output/<project_name>/ 目錄, md 格式
```
