# 功能三：讀出某科當天在看診的醫生與診間

| 項目 | 內容 |
|---|---|
| 開發日期 | 2026-10-29 ~ 2026-10-30 |
| 實作專案 | [next-is-you](https://github.com/KLTsai/next-is-you) |
| 需求來源 | [Issue #47](https://github.com/KLTsai/next-is-you/issues/47)（47a 後端切片） |
| 產出 | [PR #49](https://github.com/KLTsai/next-is-you/pull/49) ✅ squash merge 至 main：[`bec72a5`](https://github.com/KLTsai/next-is-you/commit/bec72a5) |
| 分支 | `feat/47a-hospital-catalog-roster`（短命分支） |
| 測試結果 | 268 passed、ruff format/check、tsc、test:ui、vite build 全通過 |
| 三支 skill 怎麼被使用 | [見道館總覽](./README.md#三支-skill-如何在本道館被使用) |
| ⚠️ 本功能的紀律問題 | 這一輪在功能二（PR #48）**合入之前就已經開始規劃**，而且本身又是從 #47 這張需求單再切出來的半邊，見[共通觀察](./feature-04-cascading-setup-ui.md#功能-234-共通觀察道館的一功能一輪規則沒有被遵守) |

## 這次做了什麼

在功能二交出可替換的讀取介面之後，這一輪讓系統**能讀出「某科當天有哪些醫生在看診、各自在哪個診間」**，而不只是讀單一醫生的叫號進度：

- 新增 `ScheduleReader` capability：`list_clinics(RosterQuery) -> ClinicRoster`，回傳 `RosterQuery` / `ClinicOption` / `ClinicRoster`。
- `NtuhProgressReader` 實作這個 capability：一次性讀取，合併早/午/晚三個時段的卡片，並把每張卡片屬於哪個時段記下來（使用者選診間時不用自己填時段）。這個讀取**只支援當天**（台大的來源本身不帶日期），查別的日期一律 fail-closed 回「不可用」，且刻意跟已驗證過的輪詢讀取路徑分開，不互相影響。
- registry 每筆改成多帶一個 `display_name`；`RegistryHospitalReader` 依醫院路由 `list_clinics`，只有 `FULL_SCHEDULE` 等級的來源才有名冊，`CURRENT_NUMBER_ONLY` 和 `UNSUPPORTED` 一律降級為手動輸入。
- 新增兩個端點：`GET /api/organizer/hospitals`（回傳醫院型錄）、`GET /api/organizer/hospitals/{hospital}/clinics`（回傳當天名冊），兩者都走跟既有讀取一樣的 gate，在 #22 試點前一律 held。

## 原本預期做什麼？為什麼實際做出來的不一樣？

這一輪的範圍不是來自重新定義問題，而是**我自己的切法建議**：#47 本身要做「串接式 UI」，但 UI 要能運作，後端得先有「名冊」這個新能力。我建議先出後端切片（47a），UI 本身（47b）留到下一輪，使用者同意了這個切法。這跟功能一、功能二不同——沒有發生「原本以為要做 X，查下去發現其實要做 Y」的認知修正，純粹是技術相依順序的安排。

## Double Diamond 四階段實際發生的事

### Discover

沒有新的田野調查。沿用功能一已經驗證過的 `parse_light_table()` 多時段合併解析能力，以及功能二剛交付的 registry/capability 介面。Discover 做的事是確認「名冊」這個概念在既有資料結構裡已經有素材可以組出來，不用再去源站多讀一次。

### Define

> 只有宣稱 `FULL_SCHEDULE` 的醫院才需要、也才能回答「今天這科有誰在看診」；查詢非今天的日期、查詢不支援的範疇、或讀取被拒絕，都要 fail-closed 回「名冊不可用」，不能回傳部分或猜測的結果。

### Develop

評估並排除的做法：讓 `list_clinics` 共用既有輪詢讀取的快取結果，而不是獨立發一次請求——排除的理由是輪詢路徑有自己的信任/時效假設（`trustworthy_current`），混用會讓名冊繼承不該有的時效限制；維持兩條路徑分開，即使因此多發一次請求。

### Deliver

- `tests/test_hospital_registry.py` 新增名冊路由測試；`tests/test_ntuh_reader.py` 新增三時段合併、非今天、拒絕、範疇不符四種情境。
- 新增一則端到端測試：型錄 + held 名冊路由。
- 268 個測試全綠才 push。
- 順手把一則早先遺留、尚未進 commit 的 `.gitignore` 單行改動（忽略本機 agent 工具目錄）一起帶進這個 PR——這是 trunk-based 鼓勵的「小而頻繁整合」，但也差點把一個跟本功能無關的改動混進 commit 裡（見下方 commit 紀錄）。

## 最終的 commit 紀錄

兩則 commit：

- [`2236107`](https://github.com/KLTsai/next-is-you/commit/2236107) `chore: ignore local agent tooling directories`
- [`88f3c1e`](https://github.com/KLTsai/next-is-you/commit/88f3c1e) `feat(next_is_you): serve a hospital catalog and clinic roster for the setup picker`

**合入 main 的 squash commit**：[`bec72a5`](https://github.com/KLTsai/next-is-you/commit/bec72a5) `feat(next_is_you): serve a hospital catalog and clinic roster for the setup picker (#47)`

type 選擇的理由：`chore` 用在純粹的忽略規則（不影響 src/test 的雜務），`feat` 用在真正的新能力。兩者主題不同、type 也不同，拆開是對的。

**一個差點犯的錯**：這則 `.gitignore` 改動原本是前一輪（功能二）結束時就該進 commit 的待辦，第一次嘗試是用 `git add -A` 把它跟功能三的改動一起掃進同一則 commit，後來發現兩者主題無關，用 `git reset --soft` + `git restore --staged` 拆開重做。這正好示範 [功能一發現的建議](./feature-01-watch-window.md#4-git-conventional-commit--相同-type-但主題不同的多件事也要拆要寫得更明確)：`git add -A` 很容易把不相關的檔案一起掃進去，拆分判準要在 commit **前**先核對 staged 內容，不能等 commit 完才發現。

## Trunk-Based 執行核對

| 規則 | 做到了嗎 | 證據 |
|---|---|---|
| 短命分支 | ✅ | 分支存活約一天內 |
| push 前跑完整本機 build | ✅ | 268 測試 + 全部 gate |
| 不相關變更拆開提交 | ⚠️ | `git add -A` 第一次把 `.gitignore` 掃進功能 commit，事後才拆開重做（見上） |
| 整合前先同步 trunk | ❌ 未留下證據 | 同[功能一的建議](./feature-01-watch-window.md#5-trunk-based-development--開分支那一刻需要明確要求-fetch-main)，這一輪也沒有明確 `git fetch` main 的紀錄 |

## 這次暴露的問題

這一輪除了重複出現「沒有明確 fetch main」之外，最值得記的是：**`git add -A` 容易把不相關、但「反正也该进」的零散改動一起掃進功能 commit**，必須在 `git commit` 前核對 staged 清單，不能依賴事後補救。至於這一輪跟上一輪之間完全沒有停下來觀察，同[功能四的共通觀察](./feature-04-cascading-setup-ui.md#功能-234-共通觀察道館的一功能一輪規則沒有被遵守)。
