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
