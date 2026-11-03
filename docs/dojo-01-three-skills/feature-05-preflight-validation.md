# 功能五：送出前擋掉填錯的看診資料

| 項目 | 內容 |
|---|---|
| 開發日期 | 2026-11-03 |
| 實作專案 | [next-is-you](https://github.com/KLTsai/next-is-you) |
| 需求來源 | [Issue #45](https://github.com/KLTsai/next-is-you/issues/45) AC #4，開發前另切出 [Issue #58](https://github.com/KLTsai/next-is-you/issues/58) 作為聚焦票 |
| 產出 | [PR #59](https://github.com/KLTsai/next-is-you/pull/59) ✅ 已 merge 至 main：[`514b03f`](https://github.com/KLTsai/next-is-you/commit/514b03f)（merge commit，六則原始 commit 保留在 main 上）<br>原始六則 commit：[`04b0168`](https://github.com/KLTsai/next-is-you/commit/04b0168) [`c43ffee`](https://github.com/KLTsai/next-is-you/commit/c43ffee) [`3190f15`](https://github.com/KLTsai/next-is-you/commit/3190f15) [`f1a58be`](https://github.com/KLTsai/next-is-you/commit/f1a58be) [`b343861`](https://github.com/KLTsai/next-is-you/commit/b343861) [`d0b9279`](https://github.com/KLTsai/next-is-you/commit/d0b9279) |
| 分支 | `feat/pre-flight-validation`（短命分支，同日開分支、六則 commit、開 PR、CI 綠、合入 main、分支已刪） |
| 測試結果 | 293 Python tests pass、18 UI tests pass、ruff format/check、tsc 全綠 |
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

我於是 fork 一支 agent 實地去挖，找到了 `reg.ntuh.gov.tw/WebReg/WebReg/RegDeptSchedule`：公開、不用登入、不用 CAPTCHA、未來 2 週滾動視窗、整個全院都用同一組 URL + 科別碼。這把功能五的技術天花板從「只能驗今天」直接拉到「14 天內可以完整 pre-flight」。

這個轉折直接改變了 Define 階段的範圍決策：原本只打算讓後端 Start fail-closed，變成前端 picker 也擴成 14 天內都用下拉選，draft-first 的草稿等 sweep 補驗。

## Double Diamond 四階段實際發生的事

### Discover

- 先讀 Issue #45 AC、`docs/NTUH-READER.md`、現有 reader 原始碼、功能四已經在的 `validateForm`。釐清出功能四已經蓋掉今天路徑的大半，剩下的真正缺口是：手動輸入模式（roster 讀不到時）、非今天日期、以及後端 Start 的 fail-closed 權威檢查。
- 第一輪 Discover 的「NTUH 只公告今天」是**我依賴記憶而非實地證據得到的錯誤結論**。被使用者推翻後重新開 fork 挖，找到 `RegDeptSchedule`。
- fork 第二次下去抓一份 OPH 子科別的 raw HTML（1.9 MB）保存為 fixture 證據，順便挖出一個 trap：schedule 側的「診別」（角膜/網膜/...）和 today 側的 `clinic_type`（醫師約診/特別門診/...）是**不同語彙系統**，兩套字串對不上。

### Define

問題陳述收斂到：

> 看診日期已過（純本地）、所選醫師當日沒看診（14 天內可用排班源驗證）、科/醫/診間組合不符（同上）：三類在送出前擋下，進入監看前以後端 Start fail-closed 作為權威檢查。超過 14 天的草稿接受，由每日 sweep 進入視窗時補驗。

把三個決策點拉出來請使用者拍板：

1. **K 系列**（匹配 key 的設計）：K1 = pre-flight 只用 `(date, session, room, doctor)`、clinic_type 留給 visit-day；K2 = 兩邊共用 `(date, session, room, doctor)`、改 reader 匹配；K3 = 用 `預約碼` 當全院唯一 key。我建議 K1，使用者選 K1。
2. **R 系列**（超過 14 天的草稿怎麼辦）：R1 = 接受草稿、每日 sweep 補驗；R2 = 直接拒絕未來過 14 天的日期；R3 = 接受、不自動補驗。我建議 R1，使用者選 R1。
3. 把 R1 的「每日 sweep」正式納入功能五範圍，使用者同意。

另外三個預設項使用者沒反對就照建議走：源讀不到時 Start fail-closed（與專案既有姿態一致）、14 天內的未來日期走下拉選（讓錯誤 ②③ 在 UI 層就消滅）、整輪目標一個 PR（branch-by-abstraction）。

### Develop

範圍切成四刀，用 branch-by-abstraction 讓 trunk 隨時可建置：

1. `ntuh_reader.list_clinics` 內部 dispatch：today → 既有 `DeptLightTable`，`[today+1, today+14]` → 新的 `RegDeptSchedule`，過去 → `visit_date_past`，超過 14 天 → `visit_date_out_of_schedule_window`。外層介面不變。
2. 後端 `send_gate.start_visit` 加 fail-closed 的 pre-flight 檢查，回四種錯誤碼。同一診需要 resolve step（schedule 側的「診別」改寫成 today 側的 `clinic_type`，讓 visit-day reader 原封不動地繼續用）。
3. 前端 picker 的 roster-fetch gate 從 `date === today` 擴成 `date in [today, today+14]`。新增一個 `schedule-window.ts` 把五個錯誤碼的使用者文案集中管理。
4. 排程器加一條每日 sweep：對 `[today+1, today+14]` 內的草稿群組 by (hospital, dept) 做一次 schedule fetch，比對 `(session, room, doctor)`，對不上就轉為 `scope_mismatch` + 發 organizer-attention 訊號；源壞掉時 no-op。

**動用 subagent 的方式**：使用者在中途提醒「如果可以可嘗試將後續任務可以平行處理就交給subagent，避免污染context」。於是 2-4 以三支 fork 平行跑，使用共享契約（錯誤碼名稱、匹配 key、resolve step 的歸屬、檔案互斥所有權）。中途兩支 fork 撞到 5 小時 rate limit，透過 `SendMessage` 從原本的 agent 續跑，不是重開新的，所以 context 不被二次污染。

### Deliver

- 293 Python 測試、18 UI 測試、ruff format/check、tsc 全通過才 push。
- fixture 用一份實地抓下來的真實頁面結構重建（98 行、2 天、5 卡片 + 2 停診/颱風停診案例），6 個卡片都正確解析。
- 誠實標示證據等級：fixture + 本地整合 ✅、人工實地擷取 ✅、**reader 對線上來源的行為 ❌ 完全未驗證**（和既有 NTUH-READER.md 同一條姿態）。
- 新增的 reason code 全部可被 UI 分別出，不是通用一個，使用者按下 Start 失敗時知道是「換個醫師」還是「稍後再試」。

## 人類介入的兩個轉折

**轉折一**：Agent 根據記憶聲稱「NTUH 只公告當天的看診進度，未來日期沒有源可查」。使用者沒接受這個結論，用常識反推「正常人掛號本來就看得到未來幾個月的醫師班表」。Agent 於是 fork 一支去實地挖，找到 `RegDeptSchedule`。沒有這次介入，功能五的技術範圍會被我自己錯誤的推斷壓到只能處理當天，正好把 Issue #45 AC #2「a dropdown of the doctors ... actually seeing patients in that department on that day」中「on that day」的字面意思做小了。

**轉折二**：使用者在開發中段主動提議「交給subagent，避免污染context」。這一條把我從「一支 agent 把 2-4 寫完」改成「三支 fork 平行跑」，不只省了 context，也強制讓三個子任務的檔案所有權邊界被明確畫出來（因為 fork 之間不能互踩）。結果三支 fork 的改動合在一起幾乎沒有 merge 衝突；唯一的交會點（`persistence.py`）事先在 fork prompt 裡標為「additive only」，實際上一支 fork 加了 `update_clinic_code`、另一支加了 `drafts_due_for_preflight_sweep`，兩人各自 commit，沒踩到。

## 最終的 commit 紀錄

PR #59 上有六則 commit，依 Conventional Commits 拆開：

- [`04b0168`](https://github.com/KLTsai/next-is-you/commit/04b0168) `feat(ntuh): schedule reader for 14-day pre-flight lookahead`
- [`c43ffee`](https://github.com/KLTsai/next-is-you/commit/c43ffee) `feat(frontend): extend doctor picker to visit dates within schedule window`
- [`3190f15`](https://github.com/KLTsai/next-is-you/commit/3190f15) `feat(scheduler): daily pre-flight sweep over drafts in schedule window`
- [`f1a58be`](https://github.com/KLTsai/next-is-you/commit/f1a58be) `feat(start): fail-closed pre-flight validation before monitoring`
- [`b343861`](https://github.com/KLTsai/next-is-you/commit/b343861) `docs: pre-flight schedule reader, status entry, recovery notes`
- [`d0b9279`](https://github.com/KLTsai/next-is-you/commit/d0b9279) `style: apply ruff format and lint fixes across pre-flight additions`

PR #59 用 merge commit 合入（commit [`514b03f`](https://github.com/KLTsai/next-is-you/commit/514b03f)），六則原始 commit 原封保留在 main 的歷史上。

## 三支 skill 的觀察

### `/double-diamond-principle`

- ✅ Discover 真的做了（而且被使用者推翻後**重開了一次** — 這次推翻是「我記錯 NTUH 的來源」而不是「列不是信號」那種真實資料推翻）。
- ✅ Define 的範圍決策明確用 K1/K2/K3、R1/R2/R3 的並列比較表交給使用者拍板，不是給一個方案要人說「對」。
- ⚠️ Discover 第一輪再度**依賴記憶而非實地證據**。這次是對 NTUH 來源的錯誤推斷。功能一的時候也是人類提醒才去實地讀；這次是人類質疑才去實地讀。模式一樣。
- ⚠️ 整輪 PR +1689 / -60，比功能四的 +484 / -87 又大了，雖然用了 branch-by-abstraction 保 trunk 綠，但單一 PR 的大小還是沒收斂。

**建議的 skill 改進**：Discover 要有一條「跟外部來源有關的事實宣稱，一律先去實地核對、不準用記憶」。這次是人類擋下，上次（功能一）也是人類擋下，模式太穩定了。

### `/git-conventional-commit`

- ✅ 六則 commit 都依主題拆開，type 選擇正確（`feat` 三則、`docs`、`style`、沒有 `fix`/`refactor`/`!`）。
- ✅ `style` commit 的範圍只有 ruff format/lint 的機械變更，沒有混入功能改動。
- ✅ 每則 commit 的 subject 都在 72 字元內（這次沒踩到功能三的坑）。

### `/trunk-based-development`

- ✅ 分支短命（同日完成），用 branch-by-abstraction（`list_clinics` 內部 dispatch）讓 trunk 隨時可建置。
- ✅ PR 開出前就本機跑過完整 build（pytest + typecheck + ruff + UI test）才 push。
- ⚠️ CI 第一輪失敗 — ruff format 的 3 個檔案 + 4 個 lint 自動修。這是「本機跑過 pytest 綠就 push，忘了跑 ruff format --check」的經典遺漏。
- ⚠️ fetch main 這條有做（PR 開前已經 rebase 過），但其中一支 fork 沒 fetch 就動工（這是 Opus 4.7 的 fork 行為，不是 skill 本身的 gap）。

**建議的 skill 改進**：「push 前要跑的本機 gate」清單要和專案 CI 的 deterministic-gates 對齊。這次 pytest 是通過的，但 `ruff format --check` 沒跑；本機 gate 和 CI gate 不對稱才會第一次 CI 就紅。可以在 skill 加一條「push 前先把 CI 的 lint/format gate 本機跑一次」。

## 這次最值得記下的一件事

**平行 subagent 的 context 省法在實務上有效，但要有共享契約撐住**。三支 fork 平行跑完後幾乎沒有 merge 衝突，不是因為它們互不相關，而是因為啟動時每一支都收到了明確的錯誤碼名稱、匹配 key、resolve step 歸屬、檔案所有權邊界。少了這些的話，三支 fork 的錯誤碼拼寫、`persistence.py` 的新 column、`list_clinics` 的呼叫方式會各自飛，整合時非常痛。

這跟使用者這次中途加的「交給 subagent，避免污染 context」是一體兩面：fork 本身省 context，但省掉的那部分必須預先寫進 fork 的 prompt。
