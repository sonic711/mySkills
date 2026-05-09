# Skill: Git Analysis Preflight（分析前 Git 分支檢查）

## 角色定位
你負責在任何程式分析前，先確認被分析專案的 Git 分支、遠端同步狀態與 pull 後異動範圍，避免用錯分支或舊程式碼產出報告。

## 責任邊界
- 只操作被分析專案 `project_path` 的 Git 狀態，不操作 `mySkills` repo。
- 當使用者提供 `branch` 時，必須以該分支當下 pull 後的程式狀態作為分析基準。
- 必須保守處理工作區：若被分析專案有未提交修改，不可直接切分支或 pull，除非使用者明確同意。
- 不自動 stash、reset、checkout 檔案或解衝突。
- `git pull` 後必須檢查本次拉下來的異動，判斷是否影響本次目標、已分析過的程式，或需要加入待辦清單。
- 正式報告只使用 pull 後 commit 作為分析基準，不輸出分支名稱；分支名稱只寫入索引與 Git 歷程。

## 最小輸入契約

| 欄位 | 必填 | 說明 |
|------|------|------|
| `project_name` | 是 | 專案名稱或根目錄名稱 |
| `project_path` | 是 | 被分析專案實際 Git repo 路徑 |
| `target_name` | 是 | 類別名、檔名、方法名、功能名、流程名或問題 |
| `target_type` | 建議 | `class` / `file` / `method` / `feature` / `flow` / `issue` |
| `branch` | 否 | 要分析的 Git 分支，例如 `uat`；提供時必須切到該分支並 pull |
| `scope_hint` | 否 | 模組、API、topic、table、workflow key、DTO、錯誤碼等線索 |

## 執行流程

### 1. 確認 Git repo
- 在 `project_path` 執行 `git rev-parse --show-toplevel`。
- 若不是 Git repo，回傳 `git_available=false`，後續分析可繼續，但報告需標記「未做 Git 分支校驗」。

### 2. 檢查工作區是否乾淨
- 執行 `git status --short`。
- 若有未提交修改且使用者提供 `branch`：
  - 不可切分支。
  - 不可 pull。
  - 回傳 `preflight_status=BlockedDirtyWorktree`。
  - 請使用者先 commit、stash 或允許 agent 處理。
- 若沒有指定 `branch`，可只記錄 dirty 狀態，不阻止唯讀分析；但報告需標記分析基準包含本地未提交修改。

### 3. 確認或切換分支
當有 `branch` 時：
1. 執行 `git fetch --all --prune`。
2. 若目前分支不是 `branch`：
   - 本地已有分支：執行 `git switch <branch>`。
   - 本地沒有、遠端有 `origin/<branch>`：執行 `git switch --track origin/<branch>`。
   - 找不到分支：回傳 `preflight_status=BranchNotFound`，停止分析。
3. 記錄 `before_pull_commit = git rev-parse HEAD`。

### 4. Pull 指定分支
當有 `branch` 時：
- 執行 `git pull --ff-only`。
- 若 fast-forward 失敗或需要 merge/rebase，回傳 `preflight_status=PullBlocked`，停止分析並回報原因。
- 記錄 `after_pull_commit = git rev-parse HEAD`。
- 若 `before_pull_commit == after_pull_commit`，表示本次 pull 沒有新增異動。
- `after_pull_commit` 是正式報告的 `分析基準 commit`。

### 5. 取得本次 pull 異動清單
若 pull 前後 commit 不同：

```bash
git diff --name-only <before_pull_commit>..<after_pull_commit>
```

並依檔案類型分類：
- `source_code`：`.java`、`.kt`、`.groovy`、`.py`、`.js`、`.ts`
- `sql_mapper`：`sql/`、mapper XML、`.sql`
- `config`：`.yml`、`.yaml`、`.properties`、`.xml`
- `dto_contract`：DTO、VO、proto、OpenAPI、schema
- `build`：`pom.xml`、`build.gradle`、`settings.gradle`
- `docs_tests`：文件與測試

### 6. 判斷 diff 是否影響本次目標
用 `target_name`、`target_type`、`scope_hint` 與索引比對：
- 直接命中目標檔案：`TargetImpacted`
- 命中目標上游/下游/DAO/SQL/config/DTO：`RelatedImpacted`
- 未命中本次目標，但命中 `program_index.md` 中已分析過的程式：`ExistingReportsImpacted`
- 未命中本次目標，也未命中已分析程式：`NoImpact`

### 7. 更新文件或建立待辦
根據 diff 結果決策：

| 結果 | 行動 |
|------|------|
| `NoImpact` | 忽略本次 pull 異動，照既有索引策略分析目標 |
| `TargetImpacted` | 若已有舊報告，更新舊文件；若無舊報告，重新分析 |
| `TargetImpacted` | 若已有舊報告，更新舊文件，並在正式報告補「本次邏輯變更」章節 |
| `RelatedImpacted` | 更新本次報告中相關鏈路、依賴、契約或未確認項；必要時補「本次邏輯變更」 |
| `ExistingReportsImpacted` | 不立即重做所有報告；將受影響報告寫入 `analysis_registry/<project_name>/impact_todo.md` |
| `PullBlocked` / `BranchNotFound` / `BlockedDirtyWorktree` | 停止分析，回報阻塞原因 |

### 8. 記錄 Git 歷程
每次有 Git preflight 時，更新：

```text
analysis_registry/<project_name>/git_history.md
analysis_registry/<project_name>/impact_todo.md
```

## `git_history.md` 模板
```markdown
# [project_name] Git 分析歷程

| 日期 | branch | pull 前 commit | pull 後 commit | 異動檔案數 | 影響判斷 | 本次目標 | 決策 |
|------|--------|----------------|----------------|------------|----------|----------|------|
| YYYY-MM-DD | uat | | | | NoImpact / TargetImpacted / RelatedImpacted / ExistingReportsImpacted | | |
```

## `impact_todo.md` 模板
```markdown
# [project_name] 分析文件更新待辦

| 建立日期 | branch | pull 範圍 | 受影響檔案 | 可能受影響報告 | 狀態 | 處理建議 |
|----------|--------|-----------|------------|----------------|------|----------|
| YYYY-MM-DD | uat | before..after | | | Open / Done / Ignored | 更新舊文件 / 補章節 / 重新分析 |
```

## 輸出給主協調器
- `git_available`
- `requested_branch`
- `current_branch`
- `tracking_branch`
- `before_pull_commit`
- `after_pull_commit`
- `analysis_commit`：正式報告要顯示的 commit，通常等於 `after_pull_commit`
- `pulled_changed_files`
- `impacted_current_target`
- `impacted_existing_reports`
- `impact_decision`
- `preflight_status`
- `blocker_reason`

## 品質門檻
- [ ] 若提供 `branch`，是否先確認工作區乾淨？
- [ ] 若提供 `branch`，是否已 fetch、切到該分支並執行 `git pull --ff-only`？
- [ ] 是否記錄 pull 前後 commit？
- [ ] 是否用 pull 前後 commit diff 判斷拉下來的異動？
- [ ] 是否將 `after_pull_commit` 作為正式報告唯一顯示的分析基準 commit？
- [ ] 是否判斷 diff 是否命中本次目標？
- [ ] 若 diff 命中已分析過的其他程式，是否加入 `impact_todo.md`？
- [ ] 是否避免自動 stash、reset、merge 或解衝突？
