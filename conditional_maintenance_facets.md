# Conditional Maintenance Facets（條件式維護面向）

此文件定義「只有符合特徵時才補的維護附錄」。目的不是把所有報告都寫成同一種樣子，而是依目標程式的實際型態補強必要資訊。

## 使用原則
- 主報告維持通用骨架：用途、上下游、資料契約、正常流、異常流、依賴、風險、證據。
- 只有當目標具備特定特徵時，才追加對應 facet。
- facet 是條件式附錄，不是每份報告都必填。
- 若偵測到特徵但證據不足，仍要建立該 facet，並把未確認部分標成 `Unknown`。

## 觸發方式
- 可由主協調器自動判定。
- 也可由使用者明示指定 `maintenance_facets`。
- 若自動判定與使用者指定衝突，以「保留使用者指定 + 補充高風險缺漏」為原則。

## 輸入欄位

| 欄位 | 必填 | 說明 |
|------|------|------|
| `project_name` | 是 | 專案名稱 |
| `target_name` | 是 | 類別名、方法名、功能名或流程名 |
| `target_type` | 否 | `class` / `file` / `method` / `feature` / `flow` |
| `maintenance_facets` | 否 | `batch_scheduler` / `db_write` / `broadcast_event` / `external_contract` / `manual_rerun` / `cache_sync` |
| `resolved_target_path` | 建議 | 由主協調器或導航器帶入 |

## Facet 清單

### 1. `batch_scheduler`
適用特徵：
- `@Batch`、`@Scheduled`、job、scheduler、批次 controller、手動觸發批次入口

必補內容：
- 啟動方式：手動 / 排程 / 其他系統觸發
- 排程條件或 job mode
- 執行前檢查
- 成功後驗證
- 失敗後排查
- 可否重跑
- 併發、重入、重複執行風險

### 2. `db_write`
適用特徵：
- `save` / `update` / `delete` / mapper update / insert SQL / stored procedure / 寫資料表

必補內容：
- 資料寫入矩陣
- update / insert key
- 寫入欄位
- 遞增欄位、版本欄位、同步欄位
- 部分成功場景
- 人工修復注意事項

### 3. `broadcast_event`
適用特徵：
- MQ publish、event publish、gRPC broadcast、同步通知、快取刷新通知

必補內容：
- 觸發條件
- 發送目標
- 發送內容或事件代號
- 下游預期行為
- 廣播失敗的影響
- 回歸驗證點

### 4. `external_contract`
適用特徵：
- HTTP / gRPC / gateway / 第三方系統 / 主機電文 / 外部 API / 外部回應狀態

必補內容：
- request 契約
- response 契約
- 成功條件
- 未知狀態處理
- 契約缺口
- 變更時的相容性風險

### 5. `manual_rerun`
適用特徵：
- 可手動重送、重跑、補跑、人工補執行

必補內容：
- 重跑入口
- 可重跑前提
- 不可重跑情境
- 重跑前要清查的表或狀態
- 重跑後驗證點

### 6. `cache_sync`
適用特徵：
- 快取刷新、快取失效、同步後通知其他服務重新載入

必補內容：
- 刷新對象
- 觸發條件
- 刷新前後依賴資料
- 失敗後影響
- 驗證方式

## 輸出規格
- facet 應放在主報告後半段，作為條件附錄或條件段落。
- facet 名稱應直接反映用途，不要只寫技術名。
- 同一份報告可同時套用多個 facet。

## 建議段落名稱
- `批次與排程維護`
- `資料寫入矩陣`
- `廣播/事件通知矩陣`
- `外部契約與成功條件`
- `重跑與補救`
- `快取/同步刷新驗證`

## 證據規則
- `Confirmed`：由註解、入口、SQL、repo、client、事件發送點、設定檔、line 直接驗證。
- `Inferred`：由命名、慣例、相鄰證據推定。
- `Unknown`：有維護風險，但目前無法確認。

## 品質門檻
- [ ] 是否只在符合特徵時套用 facet？
- [ ] 是否避免把某一種系統型態寫成所有報告的固定模板？
- [ ] 是否對高風險特徵補到足以維護的資訊？
- [ ] 是否保留 `Confirmed` / `Inferred` / `Unknown`？
