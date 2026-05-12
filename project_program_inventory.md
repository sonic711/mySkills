# Skill: Project Program Inventory（專案程式清單與分析排程器）

## 角色定位
你負責替單一專案建立與維護 `.java` 程式清單，記錄目前專案有多少 Java 程式檔、哪些已分析、哪些尚未分析、哪些新增/刪除/移動，並產生可供 agent 自動依序分析的佇列。

這個 skill 不產生正式分析報告；它產生與更新 `analysis_registry/<project_name>/program_inventory.md`，作為後續分析與排程依據。

## 責任邊界
- 只盤點 `.java` 檔案。
- 不修改被分析專案原始碼。
- 不把 import 關係寫成交易事實；import / injection / method call 只作為「分析順序」與「依賴候選」。
- 不取代 `analysis_report_registry.md`；本 skill 判斷程式清單與分析佇列，registry 判斷既有報告可否重用。
- 不取代 `dependency_mapper.md`；本 skill 只做足以排程的輕量依賴圖，深度依賴交給 dependency mapper。

## 索引檔位置
每個專案固定建立：

```text
analysis_registry/<project_name>/program_inventory.md
```

若檔案不存在，第一次掃描時建立。
若檔案已存在，先讀舊清單，再用目前專案 `.java` 檔比對新增、刪除、移動與可能需更新項。

## 最小輸入契約

| 欄位 | 必填 | 說明 |
|------|------|------|
| `project_name` | 是 | 專案名稱或根目錄名稱 |
| `project_path` | 是 | 被掃描專案根目錄 |
| `inventory_mode` | 建議 | `scan` / `diff` / `plan_analysis` / `next_unanalyzed` |
| `target_name` | 否 | 指定要分析的 class/file/method/feature；`plan_analysis` 時建議提供 |
| `target_type` | 否 | `class` / `file` / `method` / `feature` / `flow` |
| `branch` | 否 | Git 分支；只寫入 inventory metadata，不輸出到正式分析報告 |
| `analysis_commit` | 建議 | 本次掃描基準 commit；通常由 `git_analysis_preflight.md` 或 `git rev-parse HEAD` 提供 |
| `scope_hint` | 否 | 模組、套件、API、table、topic、功能線索 |

## 掃描範圍
只納入專案內 `.java` 檔。

預設排除：
- `.git/`
- `target/`
- `build/`
- `.gradle/`
- `out/`
- `.idea/`
- `node_modules/`
- 產生物或暫存目錄

若是多模組專案，保留模組路徑，例如：

```text
module-a/src/main/java/com/example/AService.java
```

## 執行流程

### 1. 載入既有 inventory
讀取：
1. `analysis_registry/<project_name>/program_inventory.md`
2. `analysis_registry/<project_name>/program_index.md`
3. `analysis_registry/<project_name>/shared_components.md`
4. 既有報告目錄 `analysis_output/<project_name>/`

用途：
- `program_inventory.md`：判斷新增、刪除、移動、舊依賴與舊分析狀態。
- `program_index.md`：對照是否已有正式分析報告。
- `shared_components.md`：判斷是否共用元件，避免重複排進深度分析。
- `analysis_output`：補回缺漏的報告路徑。

### 2. 掃描目前 `.java` 檔
對每個 `.java` 檔記錄：
- 檔名
- 相對路徑
- 模組
- package
- top-level type：class / interface / enum / annotation / record / unknown
- 角色推定：Controller / Service / Repository / DAO / Mapper / DTO / Entity / Config / Job / Listener / Client / Utility / Test / Unknown
- 是否 test 檔
- 最近掃描 commit

角色推定只能作為盤點欄位，不可直接當正式分析結論。

### 3. 比對新增、刪除、移動、保留
依舊清單與目前掃描結果判斷：

| 狀態 | 判斷方式 | 行動 |
|------|----------|------|
| `Active` | 舊清單存在，目前仍存在 | 保留分析狀態並更新掃描 commit |
| `Added` | 目前存在，舊清單不存在 | 標為 `未分析`，加入新增未分析統計 |
| `Deleted` | 舊清單存在，目前不存在 | 標為 `已刪除`，保留舊報告路徑與最後已知資訊 |
| `Moved` | 檔名/class/package 高度相似但路徑改變 | 更新路徑，保留報告關聯並標 `需確認` |
| `Changed` | Git diff 或 commit 顯示檔案異動，且已有報告 | 標為 `需更新` |

若無法安全判斷 moved，只標 `Deleted` + `Added`，不可硬合併。

### 4. 對照分析狀態
用 `program_index.md`、`shared_components.md` 與報告檔判斷：

| 分析狀態 | 說明 |
|----------|------|
| `未分析` | 沒有正式報告，也不是已分析共用元件 |
| `已分析` | 已有符合現行模板的正式報告 |
| `需更新` | 程式檔 changed，或報告缺少現行必備章節 |
| `需補章節` | 有舊報告但缺快速結論、業務流程、資料流、資料格式、SQL、未確認證據等章節 |
| `共用元件已分析` | shared_components 標示已分析，後續目標可引用摘要 |
| `已刪除` | 程式已不存在，保留歷史紀錄 |

### 5. 建立輕量依賴圖
只為了分析順序，不作為交易事實。

可用線索：
- `import`
- 同 package class name reference
- constructor injection
- field injection
- Spring annotation injection
- method parameter / return type
- repository / mapper / client / service 欄位
- extends / implements

依賴分類：

| 類型 | 說明 |
|------|------|
| `DirectCodeDependency` | 明確 import、注入、extends/implements 或型別引用 |
| `PossibleRuntimeDependency` | 方法呼叫、bean name、字串常數、route key 等可能 runtime 關聯 |
| `DataDependency` | Entity、DTO、table、mapper、event payload 關聯 |
| `SharedComponentDependency` | 共用工具、client、config、base class |

### 6. 產生分析佇列
`inventory_mode=next_unanalyzed`：
- 依角色與風險排序尚未分析項。
- 優先順序建議：
  1. Controller / Listener / Job / Scheduler / Flow entry
  2. Service / Orchestrator
  3. Client / External Adapter
  4. Repository / DAO / Mapper
  5. Entity / DTO
  6. Utility / Config
  7. Test

`inventory_mode=plan_analysis` 且有 `target_name`：
- 找出 target 對應程式。
- 往前追 target 的直接依賴。
- 若依賴尚未分析，排在 target 之前。
- 若依賴是共用元件且已分析，列為 `ReferenceOnly`，不排入深度分析。
- 若依賴太多，只納入與資料流、SQL、外部呼叫、回應組裝、業務分支相關的高價值依賴。

排程原則：
- 先分析被依賴且尚未分析的核心元件。
- 再分析使用者指定目標。
- 最後回到主報告整合依賴結論。

## `program_inventory.md` 模板

```markdown
# [project_name] Java 程式清單

## 1. 統計摘要
- 掃描時間：
- 掃描基準 commit：
- branch（僅供索引使用，正式報告不輸出）：
- Java 檔案總數：
- Active：
- Added：
- Changed：
- Moved：
- Deleted：
- 已分析：
- 未分析：
- 需更新：
- 需補章節：
- 共用元件已分析：

## 2. Java 程式清單
| 狀態 | 程式 | 路徑 | 模組 | package | 類型 | 角色推定 | 分析狀態 | 報告路徑 | 共用元件 | 最後掃描 commit | 備註 |
|------|------|------|------|---------|------|----------|----------|----------|----------|----------------|------|
| Active / Added / Changed / Moved / Deleted | XxxService.java | src/main/java/... | module-a | com.example | class / interface / enum / record / annotation / unknown | Service / Controller / DAO / DTO / Entity / Config / Job / Listener / Client / Utility / Test / Unknown | 未分析 / 已分析 / 需更新 / 需補章節 / 共用元件已分析 / 已刪除 | analysis_output/... | 是 / 否 | commit sha | |

## 3. 輕量依賴圖
| 程式 | 依賴程式 | 依賴類型 | 證據線索 | 分析狀態 | 排程建議 |
|------|----------|----------|----------|----------|----------|
| A.java | B.java | DirectCodeDependency / PossibleRuntimeDependency / DataDependency / SharedComponentDependency | import / injection / extends / field / DTO / mapper | 未分析 / 已分析 / 共用元件已分析 / Unknown | AnalyzeBeforeTarget / ReferenceOnly / Optional / Ignore |

## 4. 尚未分析清單
| 優先序 | 程式 | 路徑 | 角色推定 | 原因 | 建議下一步 |
|--------|------|------|----------|------|------------|
| 1 | | | | 未分析 / Added / Changed / 入口元件 / 高依賴 | AnalyzeFresh / Patch / ReferenceOnly |

## 5. 指定目標分析佇列
| 順序 | 程式 | 路徑 | 佇列原因 | 前置依賴 | 建議動作 |
|------|------|------|----------|----------|----------|
| 1 | B.java | | A 依賴 B 且 B 未分析 | | AnalyzeFresh |
| 2 | C.java | | A 依賴 C 且 C 已是共用元件 | | ReferenceOnly |
| 3 | A.java | | 使用者指定目標 | B.java, C.java | AnalyzeFresh / Patch |

## 6. 刪除或移動紀錄
| 狀態 | 舊路徑 | 新路徑 | 程式 | 原報告路徑 | 建議 |
|------|--------|--------|------|------------|------|
| Deleted / Moved | | | | | 更新索引 / 標記歷史 / 補新路徑 |
```

## 輸出給主協調器
回傳以下欄位：
- `inventory_found`：是否找到既有 inventory。
- `java_file_total`：目前 `.java` 檔案總數。
- `inventory_summary`：Active / Added / Changed / Moved / Deleted / 已分析 / 未分析 / 需更新統計。
- `added_programs`：新增 `.java` 清單。
- `deleted_programs`：刪除 `.java` 清單。
- `changed_programs`：變更且可能需更新報告的清單。
- `unanalyzed_programs`：尚未分析清單。
- `dependency_graph`：輕量依賴圖。
- `analysis_queue`：建議分析佇列。
- `reference_only_programs`：已分析共用元件，可引用不重複分析。
- `inventory_update_path`：`analysis_registry/<project_name>/program_inventory.md`。

## 與其他 skill 的協作
- `main_orchestrator.md`：需要盤點、檢查新增刪除、或自動依序分析時，先呼叫本 skill。
- `analysis_report_registry.md`：使用本 skill 的分析狀態與報告路徑，判斷 Reuse / Patch / Analyze Fresh。
- `dependency_mapper.md`：當本 skill 的依賴圖不足以判斷風險或順序時，交給 dependency mapper 深入分析。
- `git_analysis_preflight.md`：若有 branch/pull，本 skill 使用 preflight 的 commit 與 diff 來標記 Changed / Added / Deleted。

## 品質門檻
- [ ] 是否只掃描 `.java`？
- [ ] 是否排除 build、target、.git 等非原始碼目錄？
- [ ] 是否更新 Java 檔案總數、已分析、未分析、需更新、刪除統計？
- [ ] 是否和 `program_index.md`、`shared_components.md`、既有報告路徑對照？
- [ ] 是否保留 Deleted/Moved 歷史而不是直接刪掉紀錄？
- [ ] 是否建立輕量依賴圖，但沒有把它誤寫成交易事實？
- [ ] 若指定 target，是否產生依賴前置分析佇列？
- [ ] 是否讓 `program_inventory.md` 可作為 agent 自動分析尚未分析程式的依據？
