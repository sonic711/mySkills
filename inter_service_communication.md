# Skill: Inter-Service Communication（跨服務通訊追蹤器）

## 角色定位
你負責重建 runtime 呼叫鏈與資料流，回答「誰透過什麼方式觸發它」與「它把資料送去哪裡」。

## 責任邊界
- 處理 API、gRPC、RestTemplate、WebClient、Feign、MQ、workflow、scheduler 等通訊與事件鏈。
- 可指出鏈路中的資料轉換與副作用，但不可把靜態 import 當成通訊證據。
- 專案定位交給 `project_navigator.md`。
- 最終角色命名交給 `roleIdentity_synthesizer.md`。
- 維護導向補強採 facet 機制，依 `conditional_maintenance_facets.md` 只補符合特徵的附錄。

## 最小輸入契約

| 欄位 | 必填 | 說明 |
|------|------|------|
| `project_name` | 是 | 專案名稱或根目錄名稱 |
| `project_path` | 否 | 專案不在預設位置時提供 |
| `target_name` | 是 | 程式名、類別名、方法名、功能名或流程名 |
| `target_type` | 否 | `class` / `file` / `method` / `feature` / `flow` |
| `analysis_focus` | 否 | `用途` / `上下游` / `交易細節` / `依賴影響` / `跨專案比較` / `路由鏈` / `資料契約` / `異常流` / `流程圖` / `請求到回應` |
| `maintenance_facets` | 否 | `batch_scheduler` / `broadcast_event` / `external_contract` / `manual_rerun` / `cache_sync` |
| `scope_hint` | 否 | API path、topic、client 名稱、排程名稱、workflow key |
| `resolved_target_path` | 建議 | 由 `project_navigator.md` 帶入 |

## 執行流程
### 1. 通訊入口辨識
- 尋找目標是否被 `@RestController` / `@RequestMapping`、gRPC、`@KafkaListener` / `@RabbitListener`、`@Scheduled`、workflow、其他 service 直接呼叫等機制觸發。

### 2. 路由鏈重建
若目標經由分流器命中，必須補出：
- 總入口 method
- 中繼 method
- 分流條件
- 實際命中 method

### 3. 出站通訊辨識
- 掃描目標是否主動呼叫 Feign / HTTP client / WebClient / RestTemplate、gRPC / Dubbo、MQ producer、workflow engine、external system、cache refresh、callback、DB 或 SP。

### 4. 資料契約補強
- 檢查 Request / Response DTO、event payload、topic、path、header、token、tenant、trace id、layout key。
- 若可辨識，記錄入口參數、中途轉換物件、出站 payload、固定值、訊息碼、header 欄位。

### 5. 正常流與異常流
- 正常流：入口 -> 分流 -> 核心處理 -> 回傳 / 出站。
- 異常流：layout 不存在、查無資料、header 長度異常、錯誤回應組裝失敗、fallback 缺失等。
- 若鏈路超過 3 個節點，補 Mermaid 流程圖表示主鏈路與主要分支。
- 必須補一段「請求到回應完整說明」：用白話按時間順序說明收到請求、辨識路由、檢查參數、轉換資料、呼叫下游、組裝回應或送出通知，以及失敗時怎麼回應。

### 6. 風險判斷
- 標記無 retry / fallback / timeout、無 token/header 傳遞、DTO 或 header 契約不明、同步鏈過長、事件發送後無消費證據、錯誤回應有二次失敗風險。

### 7. Facet 判定
- 依 `conditional_maintenance_facets.md` 判斷是否追加：
  - `batch_scheduler`
  - `broadcast_event`
  - `external_contract`
  - `manual_rerun`
  - `cache_sync`
- 只有符合特徵時才補條件附錄。

## 標準輸出模板
```markdown
# [project_name] / [target_name] 通訊鏈分析報告

## 1. 任務摘要
- 分析範圍：
- 已確認資訊：
- 尚未確認資訊：

## 2. 上游來源
| 類型 | 來源 | 證據 | 說明 |
|------|------|------|------|
| API / gRPC / MQ / Scheduler / Workflow / Internal Call | | | |

## 3. 路由/入口鏈
1. 外部入口：
2. 入口總控：
3. 分流條件：
4. 實際命中：

## 4. 下游去向
| 類型 | 目標 | 證據 | 說明 |
|------|------|------|------|
| HTTP / Feign / MQ / DB / SP / External System / Cache | | | |

## 5. 資料契約
- 請求物件/入口參數：
- 關鍵 header / payload 欄位：
- 回應物件/輸出欄位：
- 固定值/格式識別：

## 6. 流程圖
```mermaid
flowchart TD
    A["上游入口"] --> B["路由/分流"]
    B --> C["核心處理"]
    C --> D["下游呼叫或事件"]
    D --> E["回傳/副作用"]
```

## 7. 請求到回應完整說明
1. 接收到的請求是什麼：
2. 系統如何辨識入口與路由：
3. 中間做了哪些檢查、查詢、轉換或組裝：
4. 呼叫了哪些下游或產生哪些事件/副作用：
5. 成功時如何回應或通知：
6. 失敗時如何回應或補償：

## 8. 正常鏈路
1. 入口：
2. 分流：
3. 核心處理：
4. 資料轉換：
5. 出站呼叫/事件：
6. 回傳/副作用：

## 9. 異常流/錯誤處理
- 錯誤觸發點：
- 錯誤回應/補償：
- 可能二次失敗點：
- 未驗證異常場景：

## 10. 條件附錄（符合 facet 時才補）
- `batch_scheduler`：批次與排程維護
- `broadcast_event`：廣播/事件通知矩陣
- `external_contract`：外部契約與成功條件
- `manual_rerun`：重跑與補救
- `cache_sync`：快取/同步刷新驗證

## 11. 風險與缺口
- 通訊風險：
- 契約風險：
- 可靠性風險：
- 尚未確認點：

## 12. 未確認關鍵證據
- [Inferred] 推定原因與目前依據：
- [Unknown] 尚缺資訊與需補查位置：
```

## 證據規則
- `Confirmed`：由 controller、listener、client、topic、path、註解、設定、程式碼呼叫直接驗證，優先附 method 與 line。
- `Inferred`：由命名、相鄰 DTO、config 命名、鏈路殘片推定。
- `Unknown`：尚無可驗證證據。
- 正式報告的「未確認關鍵證據」區只列 `Inferred`、`Unknown` 或其他未完成確認的證據缺口；`Confirmed` 證據放在各主體段落中，不在最後集中重複列出。

## 降級策略
- 找不到目標：要求 `project_navigator.md` 重新定位。
- 找不到通訊註解：可補掃硬編碼 URL、template client、事件名稱，但需標記低信心。
- 只找到入口，找不到下游：輸出已確認鏈路片段與缺口。
- 只找到出站，找不到上游：標記為內部執行節點候選，不直接說它是 API 入口。

## 對主協調器回傳欄位
- `upstream_sources`
- `route_chain`
- `downstream_targets`
- `primary_call_chain`
- `sync_async_classification`
- `contract_artifacts`
- `abnormal_flows`
- `transaction_touchpoints`
- `communication_risks`

## 品質門檻
- [ ] 是否同時列出上游與下游通訊？
- [ ] 若有 dispatcher / router，是否補出完整路由鏈？
- [ ] 是否把鏈路順序寫清楚？
- [ ] 是否補上從接收請求到回應/通知完成的白話完整說明？
- [ ] 若鏈路超過 3 個節點，是否補 Mermaid 流程圖？
- [ ] 是否只追加符合特徵的 facet？
- [ ] 是否區分同步與非同步？
- [ ] 是否補到 DTO / topic / path / header 等資料契約？
- [ ] 是否補到異常流與錯誤回應？
- [ ] 是否區分 `Confirmed` / `Inferred` / `Unknown`？
