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
- 資料物件已盤點：
- 資料物件需分析：
- 共用元件已分析：

## 2. Java 程式清單
| 狀態 | 程式 | 路徑 | 模組 | package | 類型 | 角色推定 | 分析狀態 | 報告路徑 | 共用元件 | 資料物件判定 | 最後掃描 commit | 備註 |
|------|------|------|------|---------|------|----------|----------|----------|----------|----------------|----------------|------|
| Active / Added / Changed / Moved / Deleted | XxxService.java | src/main/java/... | module-a | com.example | class / interface / enum / record / annotation / unknown | Service / Controller / DAO / VO / DTO / Request / Response / Entity / Config / Job / Listener / Client / Utility / Test / Unknown | 未分析 / 已分析 / 資料物件已盤點 / 資料物件需分析 / 需更新 / 需補章節 / 共用元件已分析 / 已刪除 | analysis_output/... | 是 / 否 | DataObjectOnly / DataObjectWithLogic / NotDataObject / Unknown | commit sha | |

## 2-1. VO/DTO 屬性與使用者
| 程式 | 類型 | 屬性 | 型別 | annotation/限制 | 被哪些程式使用 | 是否含內部邏輯 | 內部邏輯摘要 |
|------|------|------|------|-----------------|----------------|----------------|--------------|
| XxxReqVO.java | VO / DTO / Request / Response / Entity | fieldName | String / BigDecimal / LocalDate / List<Xxx> | @NotNull / @JsonProperty / @Column | AService.java, BController.java | 否 / 是 / 未確認 | 純欄位 / convertXxx() / validateXxx() |

## 3. 輕量依賴圖
| 程式 | 依賴程式 | 依賴類型 | 證據線索 | 分析狀態 | 排程建議 |
|------|----------|----------|----------|----------|----------|
| A.java | B.java | DirectCodeDependency / PossibleRuntimeDependency / DataDependency / SharedComponentDependency / DataObjectUsage | import / injection / extends / field / DTO / mapper | 未分析 / 已分析 / 資料物件已盤點 / 共用元件已分析 / Unknown | AnalyzeBeforeTarget / ReferenceOnly / DataObjectSummary / Optional / Ignore |

## 4. 尚未分析清單
| 優先序 | 程式 | 路徑 | 角色推定 | 原因 | 建議下一步 |
|--------|------|------|----------|------|------------|
| 1 | | | | 未分析 / Added / Changed / 入口元件 / 高依賴 | AnalyzeFresh / Patch / ReferenceOnly |

## 5. 指定目標分析佇列
| 順序 | 程式 | 路徑 | 佇列原因 | 前置依賴 | 建議動作 |
|------|------|------|----------|----------|----------|
| 1 | B.java | | A 依賴 B 且 B 未分析 | | AnalyzeFresh |
| 2 | CReqVO.java | | A 使用 CReqVO，且 CReqVO 是純資料物件 | | DataObjectSummary |
| 3 | C.java | | A 依賴 C 且 C 已是共用元件 | | ReferenceOnly |
| 4 | A.java | | 使用者指定目標 | B.java, CReqVO.java, C.java | AnalyzeFresh / Patch |

## 6. 刪除或移動紀錄
| 狀態 | 舊路徑 | 新路徑 | 程式 | 原報告路徑 | 建議 |
|------|--------|--------|------|------------|------|
| Deleted / Moved | | | | | 更新索引 / 標記歷史 / 補新路徑 |
