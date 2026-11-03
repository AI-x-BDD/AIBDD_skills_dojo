# 功能五：送出前擋掉填錯的看診資料

| 項目 | 內容 |
|---|---|
| 開發日期 | 2026-11-03 |
| 實作專案 | [next-is-you](https://github.com/KLTsai/next-is-you) |
| 需求來源 | [Issue #45](https://github.com/KLTsai/next-is-you/issues/45) AC #4，開發前另切出 [Issue #58](https://github.com/KLTsai/next-is-you/issues/58) 作為聚焦票 |
| 產出 | [PR #59](https://github.com/KLTsai/next-is-you/pull/59) ✅ 已合入 main：[`514b03f`](https://github.com/KLTsai/next-is-you/commit/514b03f)（merge commit，兩個 parent，六則原始 commit 原封保留）<br>原始六則 commit：[`04b0168`](https://github.com/KLTsai/next-is-you/commit/04b0168) [`c43ffee`](https://github.com/KLTsai/next-is-you/commit/c43ffee) [`3190f15`](https://github.com/KLTsai/next-is-you/commit/3190f15) [`f1a58be`](https://github.com/KLTsai/next-is-you/commit/f1a58be) [`b343861`](https://github.com/KLTsai/next-is-you/commit/b343861) [`d0b9279`](https://github.com/KLTsai/next-is-you/commit/d0b9279) |
| 分支 | `feat/pre-flight-validation`（短命分支，同日開分支、六則 commit、開 PR、合入 main、分支已刪） |
| 測試結果 | 最終狀態：293 Python tests pass、18 UI tests pass、ruff format/check、tsc 全綠 |
| 三支 skill 怎麼被使用 | [見道館總覽](./README.md#三支-skill-如何在本道館被使用)（本道館所有功能共用） |

## 這次做了什麼

送出前擋掉三類設定錯誤，讓錯誤的草稿根本進不了監看：

1. **看診日期已過去** — 純本地檢查 vs 台北今天。
2. **選的醫生那天根本沒看診** — 比對 NTUH 的實地資料：今天的日期查 `DeptLightTable`（功能四已經在的路徑），未來 14 天內的日期查 `RegDeptSchedule`（這次新加）。
3. **科別/醫生/診間組合和來源當下對不上** — 同上，實地比對。

超過 14 天的草稿不是直接拒絕，而是先存草稿、每日 sweep 自動驗證、進入視窗後再補驗。

**不做**：號碼本身合不合理（來源不提供，驗不了）。這一條在 Issue #58 的 Non-goals 寫明。

## 原本預期做什麼？為什麼實際做出來的不一樣？

一開始我做了一個**錯誤的技術推斷**：告訴使用者「NTUH 只公告當天的看診進度，未來日期沒有源可查，所以錯誤②③只能驗今天」。這會把功能五的範圍壓小到只處理今天的草稿。

使用者直接推翻這個推斷：**「這樣有點不太合理吧？就一般認知，可以知道哪些醫生在哪幾個月哪幾日什麼時段有開診，然後病人可以先查詢完後掛號自己想看的日期。」**

實地查證後找到了 `reg.ntuh.gov.tw/WebReg/WebReg/RegDeptSchedule`：公開、不用登入、不用 CAPTCHA、未來 2 週滾動視窗、整個全院都用同一組 URL + 科別碼。這把功能五的技術天花板從「只能驗今天」直接拉到「14 天內可以完整 pre-flight」。

這個轉折直接改變了 Define 階段的範圍決策：原本只打算讓後端 Start fail-closed，變成前端 picker 也擴成 14 天內都用下拉選，draft-first 的草稿等 sweep 補驗。

## Double Diamond 四階段實際發生的事

### Discover

- 先讀 Issue #45 AC、`docs/NTUH-READER.md`、現有 reader 原始碼、功能四已經在的 `validateForm`。釐清出功能四已經蓋掉今天路徑的大半，剩下的真正缺口是：手動輸入模式（roster 讀不到時）、非今天日期、以及後端 Start 的 fail-closed 權威檢查。
- 第一輪 Discover 的「NTUH 只公告今天」是**依賴記憶而非實地證據得到的錯誤結論**。被使用者推翻後重新查證，找到 `RegDeptSchedule`。
- 實地抓一份 OPH 子科別的真實頁面（1.9 MB raw HTML）保存為證據，順便挖出一個 trap：schedule 側的「診別」（角膜/網膜/...）和 today 側的 `clinic_type`（醫師約診/特別門診/...）是**不同語彙系統**，兩套字串對不上。

### Define

問題陳述收斂到：

> 看診日期已過（純本地）、所選醫師當日沒看診（14 天內可用排班源驗證）、科/醫/診間組合不符（同上）：三類在送出前擋下，進入監看前以後端 Start fail-closed 作為權威檢查。超過 14 天的草稿接受，由每日 sweep 進入視窗時補驗。

把三個決策點拉出來請使用者拍板：

1. **K 系列**（匹配 key 的設計）：K1 = pre-flight 只用 `(date, session, room, doctor)`、clinic_type 留給 visit-day；K2 = 兩邊共用 `(date, session, room, doctor)`、改 reader 匹配；K3 = 用 `預約碼` 當全院唯一 key。建議 K1，使用者選 K1。
2. **R 系列**（超過 14 天的草稿怎麼辦）：R1 = 接受草稿、每日 sweep 補驗；R2 = 直接拒絕未來過 14 天的日期；R3 = 接受、不自動補驗。建議 R1，使用者選 R1。
3. 把 R1 的「每日 sweep」正式納入功能五範圍，使用者同意。

另外三個預設項使用者沒反對就照建議走：源讀不到時 Start fail-closed（與專案既有姿態一致）、14 天內的未來日期走下拉選（讓錯誤 ②③ 在 UI 層就消滅）、整輪目標一個 PR（branch-by-abstraction）。

### Develop

對話紀錄裡明確記下的四項技術決定：

1. `ntuh_reader.list_clinics` 內部 dispatch：today → 既有 `DeptLightTable`，`[today+1, today+14]` → 新的 `RegDeptSchedule`，過去 → `visit_date_past`，超過 14 天 → `visit_date_out_of_schedule_window`。外層介面不變。
2. 後端 `send_gate.start_visit` 加 fail-closed 的 pre-flight 檢查，回四種錯誤碼。同一診需要 resolve step（schedule 側的「診別」改寫成 today 側的 `clinic_type`，讓 visit-day reader 原封不動地繼續用）。
3. 前端 picker 的 roster-fetch gate 從 `date === today` 擴成 `date in [today, today+14]`。新增一個 `schedule-window.ts` 把五個錯誤碼的使用者文案集中管理。
4. 排程器加一條每日 sweep：對 `[today+1, today+14]` 內的草稿群組 by (hospital, dept) 做一次 schedule fetch，比對 `(session, room, doctor)`，對不上就轉為 `scope_mismatch` + 發 organizer-attention 訊號；源壞掉時 no-op。

### Deliver

- pytest（293 tests）與 UI 測試（18 tests）在多數 commit 前都跑過；第一次 push 沒跑 `ruff format --check`，CI 第一輪因此紅燈，補跑格式化、修完 lint 後重 push 才轉綠。全部指標最終通過才合併。
- fixture 用一份實地抓下來的真實頁面結構重建（98 行、2 天、5 卡片 + 2 停診/颱風停診案例），6 個卡片都正確解析。
- 誠實標示證據等級：fixture + 本地整合 ✅、人工實地擷取 ✅、**reader 對線上來源的行為 ❌ 完全未驗證**（和既有 NTUH-READER.md 同一條姿態）。
- 新增的 reason code 全部可被 UI 分別出，不是通用一個，使用者按下 Start 失敗時知道是「換個醫師」還是「稍後再試」。

## 人類介入

Agent 根據記憶聲稱「NTUH 只公告當天的看診進度，未來日期沒有源可查」。使用者沒接受這個結論，用常識反推「正常人掛號本來就看得到未來幾個月的醫師班表」。Agent 於是重新查證，找到 `RegDeptSchedule`。沒有這次介入，功能五的技術範圍會被錯誤的推斷壓到只能處理當天，正好把 Issue #45 AC #2「a dropdown of the doctors ... actually seeing patients in that department on that day」中「on that day」的字面意思做小了。

這跟功能一的轉折是同一個性質：Agent 對外部來源提出的技術宣稱沒有先查證，是人把它拉回來的。

## 最終的 commit 紀錄

PR #59 上有六則 commit，依 Conventional Commits 拆開：

- [`04b0168`](https://github.com/KLTsai/next-is-you/commit/04b0168) `feat(ntuh): schedule reader for 14-day pre-flight lookahead`
- [`c43ffee`](https://github.com/KLTsai/next-is-you/commit/c43ffee) `feat(frontend): extend doctor picker to visit dates within schedule window`
- [`3190f15`](https://github.com/KLTsai/next-is-you/commit/3190f15) `feat(scheduler): daily pre-flight sweep over drafts in schedule window`
- [`f1a58be`](https://github.com/KLTsai/next-is-you/commit/f1a58be) `feat(start): fail-closed pre-flight validation before monitoring`
- [`b343861`](https://github.com/KLTsai/next-is-you/commit/b343861) `docs: pre-flight schedule reader, status entry, recovery notes`
- [`d0b9279`](https://github.com/KLTsai/next-is-you/commit/d0b9279) `style: apply ruff format and lint fixes across pre-flight additions`

PR #59 用一般 merge commit 合入（[`514b03f`](https://github.com/KLTsai/next-is-you/commit/514b03f)，兩個 parent），六則原始 commit 原封保留在 main 的歷史上；repo 本身 squash / merge / rebase 三種合併策略都開放。

## Trunk-Based 執行核對

| 規則 | 做到了嗎 | 證據 |
|---|---|---|
| 短命分支 | ✅ | 同一天開分支、六則 commit、開 PR、合入、分支已刪 |
| 整合前先同步 trunk | ✅ | 開分支前 `git fetch origin main` + `pull --ff-only` + 從最新 main 切出 |
| push 前跑完整本機 build | ⚠️ | pytest 與 UI 測試大多有跑且綠；但第一次 push 沒跑 `ruff format --check`，CI 第一輪紅燈 |
| 每個 commit 都不弄壞 build | ⚠️ | 同上，`style: apply ruff format and lint fixes` 這則 commit 本身就是在補前面幾則 commit 漏掉的格式檢查 |
| 保持 trunk 可發布 | ✅ | CI `deterministic-gates` 最終綠燈才合併 |
| 未完成功能用 flag 或 branch by abstraction | ✅ | `list_clinics` 內部依日期 dispatch，外層介面不變，沿用既有 `NTUH_READER_ENABLED` 旗標 |
| 不相關變更拆開提交 | ✅ | 六則 commit 依主題分離（reader / frontend / scheduler / start / docs / style），沒有混雜 |

## 這次暴露的 skill 可改進點

### 1. `/double-diamond-principle` - Discover 對外部來源的事實宣稱要先核對，不能只靠記憶

**問題**：Discover 階段對 NTUH 來源能力的描述（「只公告當天」）是憑記憶寫出來的宣稱，沒有先去源頭核對就當作範圍限制的前提往下走。使用者質疑後才查證，發現宣稱是錯的，範圍因此整整差了一個量級（只能驗今天 vs. 14 天內可完整 pre-flight）。

**影響**：這不是第一次。功能一的 Discover 也假設「列的出現消失」是監看信號，同樣沒有先去實地核對就當前提，後來被人類要求去看實際看診狀況才推翻。兩次錯誤型態相同：對外部來源的事實宣稱，沒有先查證就使用。

**建議改動**：Discover 要有一條明確規則：「任何跟外部來源能力或行為有關的事實宣稱，在用它限縮範圍或排除做法之前，必須先查證（讀文件、讀程式碼、或實地檢視來源），不能僅憑記憶或常識假設」。這條目前在 `/double-diamond-principle` 裡沒有，功能一的提案 2（Discover 補一條「第一站是專案的證據目錄」）只覆蓋了專案內部證據，沒有覆蓋「對外部系統能力的宣稱」這一類。

### 2. `/trunk-based-development` - push 前的本機 gate 清單要跟專案 CI 的 deterministic-gates 對齊

**問題**：push 前跑了 pytest 並確認全綠，但沒有把 `ruff format --check` 和 `ruff check` 一併跑過，導致 CI 第一輪在格式與 lint 上失敗，要再補一個 `style` commit 才轉綠。

**影響**：跟功能一的提案 5（開分支那一刻需要明確要求 fetch main）同類型 — 都是 `/trunk-based-development` 裡「push 前」步驟寫得不夠具體，只寫「跑完整 build」這種抽象描述，容易漏掉某一項本機沒跑過的 CI gate。

**建議改動**：skill 裡的「push 前先本機 build」要明確寫成「比對目標 repo 的 CI 設定檔，列出 CI 實際執行的每一個 gate（測試、格式、lint、型別檢查等），本機逐項跑過才 push」，而不是只寫一句「跑完整 build」帶過。

## 衍生產出

- **新增證據檔** `research/private/source-evidence/ntuh-schedule-2026-11-03.md`，記錄 `RegDeptSchedule` 的請求契約、14 天滾動視窗、`診別` 與 `clinic_type` 的語彙落差；另外兩份 raw HTML fixture 存在 `research/private/source-evidence/raw/`（該目錄被 git 排除，僅存在本機）。
- **新開 [Issue #58](https://github.com/KLTsai/next-is-you/issues/58)**：從 #45 AC #4 切出的聚焦票，記錄範圍、K1/R1 等決策與 non-goals。
- **更新 `docs/NTUH-READER.md`**：新增「Future-date schedule reader」章節，記錄 `RegDeptSchedule` 契約、match 策略、狀態標記、五個 reason code、每日 sweep 行為。
- **更新 `README.md` / `README.zh-TW.md`**：Status 新增一條已完成項目，連到 #58。
