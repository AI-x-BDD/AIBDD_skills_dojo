# 功能三：讀出某科當天在看診的醫生與診間

| 項目 | 內容 |
|---|---|
| 開發日期 | 2026-10-29 ~ 2026-10-30 |
| 實作專案 | [next-is-you](https://github.com/KLTsai/next-is-you) |
| 需求來源 | [Issue #47](https://github.com/KLTsai/next-is-you/issues/47)（功能二 Define 階段從 #45 切出來的那一半，本輪做它的後端切片） |
| 產出 | [PR #49](https://github.com/KLTsai/next-is-you/pull/49) ✅ 已 squash merge 至 main：[`bec72a5`](https://github.com/KLTsai/next-is-you/commit/bec72a5)<br>原始兩則 commit（分支已刪，PR 內可查）：[`2236107`](https://github.com/KLTsai/next-is-you/commit/2236107) [`88f3c1e`](https://github.com/KLTsai/next-is-you/commit/88f3c1e) |
| 分支 | `feat/47a-hospital-catalog-roster`（短命分支，存活約一日） |
| 測試結果 | 268 passed、ruff format/check、tsc、test:ui、vite build 全通過 |
| 三支 skill 怎麼被使用 | [見道館總覽](./README.md#三支-skill-如何在本道館被使用)（本道館所有功能共用） |

## 這次做了什麼

功能二交出了可替換的讀取介面，但那個介面只會回答一件事：「某一位醫師、某一診間，現在叫到幾號」。要讓設定表單能用選的，系統得先能回答另一個問題：**「這一科今天有哪些醫師在看診，各自在哪個診間」**。

- 新增 `ScheduleReader` capability：`list_clinics(RosterQuery) -> ClinicRoster`，以及 `RosterQuery` / `ClinicOption` / `ClinicRoster` 三個值物件。
- `NtuhProgressReader` 實作它：一次性讀取，合併早／午／晚三個時段（`AmpmCode` 三個值各發一次），並把每張卡片屬於哪個時段記在 `ClinicOption` 上 - 這樣使用者之後選診間時不用自己判斷時段。
- 這個讀取**只支援當天**：台大的來源本身不帶日期，查別的日期一律 fail-closed 回「不可用」（`date_not_today`），而不是把今天的資料當成那天的答案。
- registry 每筆多帶一個 `display_name`；`RegistryHospitalReader` 依醫院路由 `list_clinics`，而且**只有 `FULL_SCHEDULE` 等級的來源才路由**，`CURRENT_NUMBER_ONLY` 與 `UNSUPPORTED` 直接回「名冊不可用」，讓 UI 降級為手動輸入。
- 新增兩個端點：`GET /api/organizer/hospitals`（醫院型錄）、`GET /api/organizer/hospitals/{hospital}/clinics`（當天名冊）。兩者都走跟既有讀取一樣的閘門，在 #22 試點前一律 held。

## 原本預期做什麼？為什麼實際做出來的不一樣？

這一輪**沒有**發生認知修正 - 沒有「原本以為要做 X，查下去發現其實要做 Y」。這點跟功能一（前提被實地證據推翻）和功能二（需求單被重新框定）都不同，值得記下來：並不是每一輪都會有大轉折，當前提已經被前面幾輪驗證過時，Double Diamond 的兩顆鑽石會收得很快。

本輪範圍的來源是**技術相依順序**：#47 要做的是串接式 UI，但 UI 要能運作，後端得先有「名冊」這個能力。agent 建議先出後端切片，UI 本體留到下一輪，人類同意。

值得標記的是：這個切法讓 #47 這張需求單變成兩輪工作，而道館表格原本把它們列為兩個獨立功能（功能三「讀出某科當天在看診的醫生與診間」、功能四「報到設定改成用選的」） - 切法與道館的功能編號其實對得上，但當時 agent 是照 issue 相依關係切的，不是照道館編號切的，所以沒有在切完後回頭核對編號。

## Double Diamond 四階段實際發生的事

### Discover

沿用既有資產，沒有新的對外取證：

- 功能一已經驗證過 `parse_light_table()` 能解析一個時段的完整卡片列表，且三個時段由 `AmpmCode` 參數決定。
- 功能二剛交付的 registry / capability 介面提供了「這家醫院支援到什麼程度」的描述位置。

Discover 的動作是確認「名冊」這個概念能從既有資料結構組出來 - 答案是可以，三個時段各讀一次再合併即可，不需要新的來源或新的解析邏輯。

### Define

> 只有宣稱 `FULL_SCHEDULE` 的醫院需要、也才能回答「今天這一科有誰在看診」。查詢非今天的日期、查詢不支援的科別、讀取被閘門拒絕、或根本沒有讀取器，全部 fail-closed 回「名冊不可用」，不能回傳部分或推測的結果。時段由系統從來源帶出，不要讓使用者填。

### Develop

評估並**排除**的做法：

- **讓 `list_clinics` 共用既有輪詢讀取的快取結果**，而不是獨立發請求。排除理由：輪詢路徑帶著自己的信任與時效假設（`trustworthy_current`），名冊如果繼承那些假設，會把「這個號碼可不可信」的語意混進「今天有誰看診」裡。選擇維持兩條路徑分開，代價是多發一次請求。
- **讓名冊支援任意日期**（例如回傳「這一科平常的門診表」）。排除理由：來源沒有日期概念，拿今天的資料回答別天等於製造一個會過期的謊。改成明確回「只支援當天」，讓 UI 自己決定怎麼降級。

### Deliver

- `tests/test_hospital_registry.py` 新增名冊路由測試（`FULL_SCHEDULE` 才路由、`CURRENT_NUMBER_ONLY` 與 `UNSUPPORTED` 回不可用）。
- `tests/test_ntuh_reader.py` 新增四種情境：三時段合併、非今天、讀取被拒、科別不符。
- 新增一則端到端測試：型錄端點 + 名冊端點在 held 狀態下的回應形狀。
- 268 個測試 + 全套 gate 通過才 push。

## 人類介入

**一、切法由 agent 提議，由人核准。** agent 提出「先做後端切片、UI 留到下一輪」，人類同意。這一步本身沒問題，但見上面「原本預期」那節：agent 當時是照 issue 相依切的，沒有回頭對照道館的功能編號。

**二、新舊決策衝突時，人裁決了以哪一份為準。** 過程中碰到 [#22](https://github.com/KLTsai/next-is-you/issues/22)（上線試點）裡的舊討論跟近期幾輪的決策不一致 - 名冊端點要不要跟讀取一樣被閘門 held，舊討論與近期決定方向不同。agent 把衝突攤開問，人類的裁決是「#22 基本上過去討論的東西，基本上以最近討論的內容為主」。

這個介入很關鍵：如果 agent 自己挑了舊討論當依據，會做出一個「名冊不受閘門限制」的版本 - 在測試上看不出問題，但等於在試點前就讓 production 對外發出真實讀取請求，違反專案一直守的 held 姿態。**判斷「哪一份決策還有效」需要人的記憶，不是 agent 能從 repo 裡讀出來的。**

**三、順手帶進的 `.gitignore` 改動經過人核准。** 前一輪遺留一則還沒進 commit 的 `.gitignore` 單行改動（忽略本機 agent 工具目錄）。agent 問了要不要一起帶進這個 PR，人類回「.gitignore 可以進」。

## 最終的 commit 紀錄

**審查時的兩則 commit**：

- [`2236107`](https://github.com/KLTsai/next-is-you/commit/2236107) `chore: ignore local agent tooling directories`
- [`88f3c1e`](https://github.com/KLTsai/next-is-you/commit/88f3c1e) `feat(next_is_you): serve a hospital catalog and clinic roster for the setup picker`

**合入 main 的 squash commit**：

- [`bec72a5`](https://github.com/KLTsai/next-is-you/commit/bec72a5) `feat(next_is_you): serve a hospital catalog and clinic roster for the setup picker (#47)`

type 選擇的理由：

- `chore` 給 `.gitignore`：不動 src／test 的雜務，正好對應 skill 表格裡 `chore` 的定義。
- `feat` 給名冊能力：新增一個系統原本沒有的能力（回答「今天這一科有誰在看診」）。
- 兩者 type 不同、主題也不同，拆成兩則是必然的，沒有判斷難度。
- footer 用 `Refs: #47`，不用 `Closes` - 本輪只做需求單的後端切片。

**一個差點犯的錯，以及它怎麼被抓到**：那則 `.gitignore` 改動，第一次是用 `git add -A` 跟本輪的功能改動一起掃進同一個 staging 區，準備放進同一則 commit。核對 `git status` 時發現兩者主題無關，用 `git reset --soft` + `git restore --staged` 拆開重做成兩則。

這正好示範[功能一提出的那條建議](./feature-01-watch-window.md#4-git-conventional-commit--相同-type-但主題不同的多件事也要拆要寫得更明確)的實務面：拆分判準要在 `git commit` **之前**核對 staged 清單，`git add -A` 的問題不是它會失敗，是它會成功 - 不相關的檔案悄悄進了 commit，之後要回溯就得連帶處理。

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
| 不相關變更拆開提交 | ⚠️ | `git add -A` 第一次把 `.gitignore` 掃進功能 staging，commit 前核對時才拆開 |
| 整合前先同步 trunk | ⚠️ 沒有留下 fetch 的證據 | 同功能一、功能二，重複出現的同一個缺口 |
| 一輪工作完成後才開始下一輪 | ❌ | 本輪在功能二的 PR 合入前就已經開始規劃（見功能二的「人類介入」第二點） |

## 這次暴露的 skill 可改進點

### 1. `/git-conventional-commit` - 需要一條「commit 前核對 staged 清單」的動作規則

**從哪看出**：`git add -A` 把一則與本輪主題無關的 `.gitignore` 改動掃進功能 commit 的 staging 區，是 commit 前核對才發現的。如果沒核對，就會生出一則 description 講功能、內容卻夾帶雜務的 commit。

**skill 現況**：skill 的「撰寫步驟」第 7 條是自我檢查，但檢查項目全部是**訊息格式**（冒號後有空白？description 非空？breaking 有標？type 拼對？），沒有一條檢查**內容範圍**。

**建議在「撰寫步驟」補一條（放在寫訊息之前）**：

> 在寫 commit 訊息之前，先核對將要提交的檔案清單（`git status`、`git diff --cached --stat`），確認每個檔案都屬於這則 commit 的主題。用 `git add -A` 或 `git add .` 時尤其要核對 - 它們會把工作目錄裡所有改動一起帶走，包含你這次沒有要處理的待辦。發現不屬於本則主題的檔案，先移出 staging 區並獨立成另一則 commit。

### 2. `/trunk-based-development` - 「順手整合待辦的小改動」要說清楚怎麼做才不違反拆分原則

**從哪看出**：那則 `.gitignore` 改動該不該帶進這個 PR，是個真實的兩難 - TBD 鼓勵頻繁小整合（別讓它一直擱著），但 Conventional Commits 要求主題聚焦（別混進功能 commit）。agent 問了人才決定。

**skill 現況**：skill 強調「一天多次整合」「每次都是不會弄壞 build 的小 commit」，但沒有處理「手上有一則與本輪無關的零碎改動」這個常見情境。

**建議新增規則**：

> 手上有與本輪主題無關的零碎改動（忽略規則、拼字、格式）時，傾向把它整合進去而不是擱著，但必須**獨立成一則自己的 commit**，不併入功能 commit。整合的頻率由 TBD 決定，commit 的邊界由主題決定 - 這兩件事不衝突。

## 衍生產出

- **更新 `docs/ADDING-A-HOSPITAL.md`**：補上 `ScheduleReader` 這個可選能力的說明（哪些醫院需要實作、不實作會怎麼降級），以及兩個新端點的契約。
- **把「只支援當天」寫成明確契約**：`ClinicRoster` 帶 `reason` 欄位（`date_not_today` / `scope_mismatch` / `no_observation` / `unsupported`），讓前端能分辨「為什麼沒有名冊」並給出不同的說明文字，而不是籠統地顯示失敗。這個欄位設計直接決定了功能四五種 UI 狀態裡的兩種。
