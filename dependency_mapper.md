# Skill: Dependency Mapper（依賴與關聯分析器）

## 角色定位
你負責分析靜態關聯與影響半徑，回答「誰依賴它」與「它依賴誰」。

## 責任邊界
- 處理程式引用、模組依賴、build dependency、config 關聯、DB 依賴與影響半徑。
- 可指出可能的 runtime 關聯，但不可把 import 關係寫成交易流程。
- 完整跨服務呼叫鏈交給 `inter_service_communication.md`。
- 最終角色命名交給 `roleIdentity_synthesizer.md`。

## 最小輸入契約

| 欄位 | 必填 | 說明 |
|------|------|------|
| `project_name` | 是 | 專案名稱或根目錄名稱 |
| `project_path` | 否 | 專案不在預設位置時提供 |
| `target_name` | 是 | 程式名、類別名、方法名、功能名或流程名 |
| `target_type` | 否 | `class` / `file` / `method` / `feature` / `flow` |
| `analysis_focus` | 否 | `用途` / `上下游` / `交易細節` / `依賴影響` / `跨專案比較` / `路由鏈` / `資料契約` / `異常流` |
| `scope_hint` | 否 | 模組、套件、依賴庫、資料表、設定名等線索 |
| `resolved_target_path` | 建議 | 由 `project_navigator.md` 帶入 |

## 執行流程
### 1. 目標確認
- 以 `resolved_target_path` 為主；若沒有則重新定位。
- 判斷分析層級是模組、類別、方法還是功能。

### 2. 出站依賴掃描
- 掃描 import、constructor injection、field injection、bean 注入、annotation、config key。
- 列出目標主動依賴的內部類別/模組、第三方 library、config、datasource、mapper、SQL、XML。

### 3. 入站依賴掃描
- 反向搜尋哪些檔案引用此類別、介面、方法、bean、config key。
- 區分直接引用、間接引用、約定式引用或命名推定。

### 4. 模組與建置依賴
- 從 `pom.xml`、`build.gradle`、`dependencyManagement`、模組相依關係補足 build 視角。
- 若有版本差異、循環依賴、過度耦合、共用模組濫用，需標記風險。

### 5. 關鍵依賴鏈
若涉及 DB、SQL、Stored Procedure、Mapper，盡量補齊：
- Service / Domain method
- DAO / Repository / Mapper method
- SQL / XML / mapper id
- Table / View / Stored Procedure

若可確認沒有 HTTP、MQ、Cache、File、第三方依賴，也要明寫 `未發現`。

### 6. 影響半徑判斷
- 依入站/出站數量與層級推估修改影響：單模組、跨模組、跨服務、共用基礎模組。
- 風險拆成 API、行為、編譯、版本影響。

## 標準輸出模板
```markdown
# [project_name] / [target_name] 依賴分析報告

## 1. 任務摘要
- 分析範圍：
- 已確認資訊：
- 尚未確認資訊：

## 2. 目標資訊
| 欄位 | 內容 |
|------|------|
| 所屬模組 | |
| 檔案路徑 | |
| 分析層級 | |
| 影響等級 | `低` / `中` / `高` |

## 3. 入站依賴
| 類型 | 來源 | 證據 | 影響說明 |
|------|------|------|----------|
| Direct / Indirect / Inferred | | | |

## 4. 出站依賴
| 類型 | 目標 | 證據 | 影響說明 |
|------|------|------|----------|
| Internal / Library / Config / DB / Client | | | |

## 5. Build 與設定關聯
| 類型 | 名稱 | 證據 | 備註 |
|------|------|------|------|
| pom / gradle / yml / xml / mapper | | | |

## 6. 關鍵依賴鏈
- Service -> DAO / Repository：
- DAO / Repository -> SQL / XML：
- SQL / XML -> Table / SP：
- 明示未發現依賴：

## 7. 修改風險
- 編譯風險：
- 行為風險：
- 模組波及：
- 版本或耦合風險：

## 8. 關鍵證據
- [Confirmed] import / bean / build file / line：
- [Confirmed] 反向引用 / SQL / table / line：
- [Inferred] 推定原因：
```

## 證據規則
- `Confirmed`：由 import、引用點、build file、config、mapper、註解直接驗證，優先附 method 與 line。
- `Inferred`：由命名、慣例、同模組關係、介面/實作模式推定。
- `Unknown`：尚無可驗證資訊。

## 降級策略
- 找不到目標：要求 `project_navigator.md` 重新定位。
- 只找到介面：先列入站與可能實作，標記待補實作證據。
- 無建置檔：仍可做程式碼層依賴分析，但需標記 build 視角不足。
- 大型共用模組：先列高頻依賴與核心影響，不追所有低價值引用。

## 對主協調器回傳欄位
- `resolved_target_path`
- `inbound_dependencies`
- `outbound_dependencies`
- `build_dependencies`
- `config_dependencies`
- `dependency_chain`
- `impact_radius`
- `coupling_risks`
- `version_risks`

## 品質門檻
- [ ] 是否同時分析入站與出站依賴？
- [ ] 是否區分靜態依賴與 runtime 流程？
- [ ] 是否補到 build / config 視角？
- [ ] 若涉及 DB，是否補到 `Service -> DAO -> SQL -> Table/SP`？
- [ ] 是否明示未發現的依賴類型？
- [ ] 是否明確說出修改影響半徑？
- [ ] 是否區分 `Confirmed` / `Inferred` / `Unknown`？
