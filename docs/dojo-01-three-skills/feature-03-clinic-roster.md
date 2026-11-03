# 功能三：讀出某科當天在看診的醫生與診間

| 項目 | 內容 |
|---|---|
| 開發日期 | 2026-10-29 ~ 2026-10-30 |
| 實作專案 | [next-is-you](https://github.com/KLTsai/next-is-you) |
| 需求來源 | [Issue #47](https://github.com/KLTsai/next-is-you/issues/47)（功能二在 Define 階段從 #45 切出的那一半，本輪做它的後端切片） |
| 產出 | [PR #49](https://github.com/KLTsai/next-is-you/pull/49) ✅ 已 squash merge 至 main：[`bec72a5`](https://github.com/KLTsai/next-is-you/commit/bec72a5)<br>原始兩則 commit（分支已刪，PR 內可查）：[`2236107`](https://github.com/KLTsai/next-is-you/commit/2236107) [`88f3c1e`](https://github.com/KLTsai/next-is-you/commit/88f3c1e) |
| 分支 | `feat/47a-hospital-catalog-roster`（短命分支，存活約一日） |
| 測試結果 | 268 passed、ruff format/check、tsc、test:ui、vite build 全通過 |
| 三支 skill 怎麼被使用 | [見道館總覽](./README.md#三支-skill-如何在本道館被使用)（本道館所有功能共用） |

## 這次做了什麼

功能二交出了可替換的讀取介面，但那個介面只會回答一件事：「某一位醫師、某一診間，現在叫到幾號」。要讓設定表單能用選的，系統得先能回答另一個問題：**「這一科今天有哪些醫師在看診，各自在哪個診間」**。

- 新增 `ScheduleReader` capability：`list_clinics(RosterQuery) -> ClinicRoster`，以及 `RosterQuery` / `ClinicOption` / `ClinicRoster` 三個值物件。
- `NtuhProgressReader` 實作它：一次性讀取，合併早／午／晚三個診次，並把每筆診間屬於哪個診次記在 `ClinicOption` 上，使用者選診間時不用自己填診次。
- 這個讀取**只支援當天**：台大的來源沒有日期參數，查別的日期，或任何讀不乾淨的情況，一律 fail-closed 回「名冊不可用」，由 UI 降級為手動輸入。它刻意和已驗證的輪詢讀取路徑分開。
- registry 每筆多帶一個 `display_name`；`RegistryHospitalReader` 依醫院路由 `list_clinics`，**只有 `FULL_SCHEDULE` 等級的來源才給名冊**，`CURRENT_NUMBER_ONLY` 與 `UNSUPPORTED` 直接回「不可用」。
- 新增兩個端點：`GET /api/organizer/hospitals`（醫院型錄）、`GET /api/organizer/hospitals/{hospital}/clinics`（當天名冊）。名冊經過和既有讀取相同的閘門，在 #22 試點前一律 held。

## 原本預期做什麼？為什麼實際做出來的不一樣？

這一輪**沒有**發生認知修正 - 沒有「原本以為要做 X，查下去發現其實要做 Y」。這點跟功能一（前提被實地證據推翻）和功能二（需求單被重新框定）都不同：當前提已經被前面幾輪驗證過，兩顆鑽石收得很快。

本輪範圍來自**技術相依順序**：#47 要做串接式 UI，但 UI 要能運作，後端得先有「名冊」這個能力。agent 在 PR #48 的 CI 綠燈後提出三個起點選項（先做 47a 後端切片／整個 #47 一次做／先不動手），並建議第一個；人類選了 47a。

這個切法讓 #47 變成兩輪工作，而道館表格原本就把它們列為兩個獨立功能（功能三「讀出某科當天在看診的醫生與診間」、功能四「報到設定改成用選的」）。切法與道館編號其實對得上，但當時 agent 是照 issue 相依關係切的，不是照道館編號切的，因為它並不知道道館表格的存在（見[功能四的共同觀察](./feature-04-cascading-setup-ui.md#功能-234-的共同觀察一個-session-連著跑完三輪)）。

## Double Diamond 四階段實際發生的事

### Discover

沒有新的對外取證，做的是盤點現況接縫：前端表單是一整段 template string、醫院與科別都寫死；後端要新長出兩個能力（醫院型錄、名冊）。名冊可以建在功能一已經驗證過的 `parse_light_table()` 上，它本來就回傳整份名冊。

一個在這階段被記下來的設計點：台大請求必須帶診次參數（`AmpmCode`），所以要列出「某科整天的醫師」得讀早／午／晚三次再合併。

### Define

> 只有宣稱 `FULL_SCHEDULE` 的醫院才有名冊。查詢非今天的日期、不支援的科別、讀取被閘門拒絕、或根本沒有讀取器，全部 fail-closed 回「名冊不可用」，不回傳部分或推測的結果。診次由系統從來源帶出，不讓使用者填。

### Develop

對話紀錄裡明確記下的決定：

- 名冊用「一次性讀取、合併三個診次」實作，每筆帶自己的診次。
- 只支援當天，因為來源沒有日期參數；其餘情況降級。
- 名冊讀取和已驗證的輪詢路徑分開，不影響既有的叫號監看。
- 名冊端點沿用既有讀取的 held 姿態：沒有讀取器（正式環境，#22 試點前）時名冊就是「不可用」，前端降級為手動輸入，不改既有行為。

### Deliver

- `tests/test_hospital_registry.py` 新增名冊路由測試（`FULL_SCHEDULE` 才路由、其餘回不可用）。
- `tests/test_ntuh_reader.py` 新增 `list_clinics` 的三診次合併、讀取被拒、非今天、科別不符；閘門層新增名冊 held 與 enabled。
- 新增端到端測試：型錄 + held 狀態下的名冊路由。
- 268 個測試 + 全套 gate 通過才 push。

## 人類介入

**一、起點在 agent 給的三個選項裡由人選。** 見上面「原本預期」。人類選了 agent 建議的 47a；另外兩個選項（整個 #47 一次做、先不動手）沒被選。

**二、中途補了一句確認，方向沒有改變。** 實作進行到一半，人類補了一句：「#22 基本上過去討論的東西，基本上以最近討論的內容為主」。agent 的理解是：#22（正式環境在試點前不讀醫院）的 held 姿態已經落在最近的 #34 / #46，而 47a 的設計本來就順著它走，所以回覆「方向不變，我繼續做」。

這是一次**確認**，不是糾正：它沒有改變任何設計。記在這裡是因為它示範了一種介入的形態 - 人類不確定舊討論是否還有效時，用一句話確認，agent 對照自己的設計後可以直接繼續。

**三、順手帶進的 `.gitignore` 改動經人核准。** session 開始前就存在一則未提交的 `.gitignore` 改動（功能二時為了不混進功能 commit，刻意留在工作目錄）。人類回覆「.gitignore 可以進」，agent 把它獨立成一則 `chore` commit，沒有併入功能 commit。

## 最終的 commit 紀錄

**審查時的兩則 commit**：

- [`2236107`](https://github.com/KLTsai/next-is-you/commit/2236107) `chore: ignore local agent tooling directories`
- [`88f3c1e`](https://github.com/KLTsai/next-is-you/commit/88f3c1e) `feat(next_is_you): serve a hospital catalog and clinic roster for the setup picker`

**合入 main 的 squash commit**：

- [`bec72a5`](https://github.com/KLTsai/next-is-you/commit/bec72a5) `feat(next_is_you): serve a hospital catalog and clinic roster for the setup picker (#47)`

type 選擇的理由：

- `chore` 給 `.gitignore`：不動 src／test 的雜務，正好對應 skill 表格裡 `chore` 的定義。
- `feat` 給名冊能力：新增一個系統原本沒有的能力。
- 兩者 type 與主題都不同，拆成兩則沒有判斷難度。這次 `.gitignore` 是獨立提交的，沒有重演功能二的混入問題。
- footer 用 `Refs: #47`，不用 `Closes` - 本輪只做需求單的後端切片。

### 為什麼合入時用 squash

同功能二：跟著 repo 既有的 house style（main 全是 squash、尾綴 `(#NN)`）。審查時的兩則拆解保留在 PR #49 的 commit 歷史裡，之後若要追查 `.gitignore` 是哪一刻進來的仍查得到。

## Trunk-Based 執行核對

| 規則 | 做到了嗎 | 證據 |
|---|---|---|
| 短命分支 | ✅ | 分支存活約一日 |
| push 前跑完整本機 build | ✅ | 268 測試 + ruff + tsc + test:ui + vite build |
| 每個 commit 都不弄壞 build | ✅ | 兩則各自獨立通過 |
| 保持 trunk 可發布 | ✅ | CI 綠燈才請求 merge |
| 未完成功能安全進 trunk | ✅ | 新端點在既有閘門後維持 held，production 不會因此發出真實讀取 |
| 不相關變更拆開提交 | ✅ | `.gitignore` 獨立成 `chore` commit |
| 整合前先同步 trunk | ✅ | #48 合入後執行 `git fetch`，並以 `origin/main` 為基準重建分支 |
| 一人一分支、分支基於 trunk | ⚠️ | 47a 一度疊在尚未合入的 #48 分支上（約兩分鐘），#48 合入後才改基於 main |
| 道館協定：一功能一輪（不是 skill 的規則） | ❌ | 本輪在功能二的 PR 合入之前就已經開始規劃，見[功能二](./feature-02-hospital-adapter-boundary.md)與 [README 的流程觀察](./README.md#道館流程本身的觀察一功能一-session) |

## 這次暴露的 skill 可改進點

**本輪沒有新增 skill 建議。** 三支 skill 在這一輪的單一功能範圍內都有被遵循，沒有看到哪一條規則需要補強。

唯一值得記下的是兩條規則之間的張力：使用者的規則是 agent 不自行 merge，而 TBD 希望盡快整合。agent 為了同時滿足兩者，選擇把 47a 疊在尚未合入的 #48 分支上繼續做（這樣不需要在未獲同意時 merge），代價是一度違反「分支基於 trunk」。#48 合入後 agent 立刻把分支改基於 main。這不是 skill 缺陷，而是兩個來源的規則交互出來的現象。

本輪暴露的主要問題（沒有在上一個功能結束後停下來）屬於道館流程，不屬於任何一支 skill，見 [README 的流程觀察](./README.md#道館流程本身的觀察一功能一-session)。

## 衍生產出

- **更新 `docs/ADDING-A-HOSPITAL.md`**：補上 `ScheduleReader` 這個可選能力的說明，以及兩個新端點。
- **名冊契約帶「不可用的原因」**：`ClinicRoster` 的 `reason` 欄位區分 `date_not_today`、`scope_mismatch`、`no_observation`、`unsupported` 等原因，讓前端能依原因顯示不同的說明，而不是籠統的失敗。
