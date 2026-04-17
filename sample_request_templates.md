# Sample Request Templates

## 用途
這份文件提供可直接複製貼上的請求模板，讓使用者用一致格式向 `main_orchestrator.md` 發出分析任務。欄位名稱對齊目前技能系統的輸入契約：

- `project_name`
- `project_path`
- `target_name`
- `target_type`
- `analysis_focus`
- `scope_hint`

若資訊不完整，也可先用簡化模板；但建議至少提供 `project_name` 與 `target_name`。

## 推薦欄位說明
| 欄位 | 必填 | 說明 |
|------|------|------|
| `project_name` | 是 | 專案名稱或根目錄名稱 |
| `project_path` | 否 | 專案不在預設位置時提供完整路徑 |
| `target_name` | 視情況 | 類別名、檔名、方法名、功能名、流程名 |
| `target_type` | 否 | `class` / `file` / `method` / `feature` / `flow` |
| `analysis_focus` | 否 | `用途` / `上下游` / `交易細節` / `依賴影響` / `跨專案比較` |
| `scope_hint` | 否 | 模組、API 路徑、資料表、topic、workflow key 等線索 |

## Template 1: 單一程式完整分析
適合分析單一類別、檔案或方法的用途、上游、下游與交易細節。

```text
project_name: project-a
target_name: OrderService.java
target_type: file
analysis_focus: 用途, 上下游, 交易細節, 依賴影響
scope_hint: order-service module
```

## Template 2: 單一類別簡化分析
適合你已經知道類別名稱，只想快速開始。

```text
project_name: project-a
target_name: OrderService
```

## Template 3: 指定方法分析
適合追特定 method 的交易邊界與副作用。

```text
project_name: project-a
target_name: OrderService.createOrder
target_type: method
analysis_focus: 用途, 上下游, 交易細節
scope_hint: API create order
```

## Template 4: 功能流程分析
適合從功能名稱往回找入口，再一路追到 DB、MQ、外部服務。

```text
project_name: project-a
target_name: 建立訂單流程
target_type: flow
analysis_focus: 用途, 上下游, 交易細節
scope_hint: create order, order submit, payment, inventory
```

## Template 5: 專案結構導覽
適合專案剛放進來時，先看模組結構與主要入口。

```text
project_name: project-a
analysis_focus: 用途
scope_hint: 專案結構, 模組架構, 入口模組
```

## Template 6: 明確指定專案路徑
適合專案不在預設 `projects/` 目錄下。

```text
project_name: legacy-order-system
project_path: /absolute/path/to/legacy-order-system
target_name: RefundService
target_type: class
analysis_focus: 用途, 上下游, 交易細節, 依賴影響
```

## Template 7: 比較兩個專案的同一功能
適合比較流程差異、風險點與共用能力。

```text
project_name: project-a, project-b
target_name: create order
target_type: feature
analysis_focus: 跨專案比較, 用途, 上下游, 交易細節, 依賴影響
scope_hint: API submit order, payment, inventory, order persistence
```

## Template 8: 從 API 路徑反查
適合只知道 endpoint，不知道實際程式名稱。

```text
project_name: project-a
target_name: /api/orders/create
target_type: feature
analysis_focus: 用途, 上下游, 交易細節
scope_hint: controller endpoint, create order API
```

## Template 9: 從 MQ / Event 反查
適合事件驅動系統。

```text
project_name: project-a
target_name: order-created
target_type: feature
analysis_focus: 上下游, 交易細節, 依賴影響
scope_hint: kafka topic, producer, consumer, payment, notification
```

## Template 10: 從資料表或 Mapper 反查
適合你想知道某張表是誰在寫、誰在讀。

```text
project_name: project-a
target_name: order_main
target_type: feature
analysis_focus: 上下游, 交易細節, 依賴影響
scope_hint: table, mapper, repository, insert, update
```

## Template 11: 專案剛導入時的 onboarding 請求
適合你還沒決定先分析哪個程式。

```text
project_name: project-a
analysis_focus: 用途
scope_hint: 先幫我盤點模組、主要入口、核心 service、外部整合點
```

## Template 12: 模糊查詢縮小範圍
適合名稱很常見、你不確定是哪個實作。

```text
project_name: project-a
target_name: UserService
target_type: class
analysis_focus: 用途, 上下游
scope_hint: login, auth, token, member
```

## 建議使用習慣
- 若已知類別名，優先提供 `target_type: class` 或 `target_type: file`。
- 若要查完整交易流，`analysis_focus` 建議至少包含 `上下游, 交易細節`。
- 若只知道 API、topic、table、workflow key，也可以當成 `target_name` 或放進 `scope_hint`。
- 若要比較兩個專案，請在 `project_name` 一次列出多個專案。

## 最短可用格式
如果你想最快開始，可以只用這個版本：

```text
project_name: project-a
target_name: OrderService
```

## 建議輸入格式
未來最推薦直接照這個格式送出：

```text
project_name: project-a
project_path: /absolute/path/to/project-a
target_name: OrderService.createOrder
target_type: method
analysis_focus: 用途, 上下游, 交易細節, 依賴影響
scope_hint: create order API, payment, inventory, order_main table, order-created topic
```
