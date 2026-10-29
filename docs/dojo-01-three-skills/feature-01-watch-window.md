# 功能一：讓監看區間由叫號號碼決定，不再寫死時鐘時間

| 項目 | 內容 |
|---|---|
| 開發日期 | 2026-10-29 |
| 實作專案 | [next-is-you](https://github.com/KLTsai/next-is-you) |
| 需求來源 | [Issue #34](https://github.com/KLTsai/next-is-you/issues/34) |
| 產出 | [PR #46](https://github.com/KLTsai/next-is-you/pull/46) ✅ 已 squash merge 至 main：[`cd199d7`](https://github.com/KLTsai/next-is-you/commit/cd199d7)<br>原始兩則 commit（分支已刪，PR 內可查）：[`724532d`](https://github.com/KLTsai/next-is-you/commit/724532d) [`443a68b`](https://github.com/KLTsai/next-is-you/commit/443a68b) |
| 分支 | `reader-deptlighttable-contract`（短命分支，同日開分支、提交、開 PR、合入 main、分支已刪） |
| 測試結果 | 255 passed、ruff format/check、tsc、migrations、UI 測試、build 全通過 |
| 三支 skill 怎麼被使用 | [見道館總覽](./README.md#三支-skill-如何在本道館被使用)（本道館所有功能共用） |

## 這次做了什麼

next-is-you 替家人盯台大醫院的看診叫號進度，號碼接近時發 LINE 提醒。原本的 reader 用**寫死的時鐘**決定什麼時候要去讀來源：下午診 13:30-16:30 前後各加 10 分鐘，時間外一律不讀，而且早診、夜診直接拒絕。

這次完成了兩刀：

1. **讓 reader 能真的讀到來源。** 原本發的普通 GET 不會回任何診間資料。改成兩步驟：先 GET 取 token 與 cookie，再 `POST DeptLightTable` 帶 token、cookie、`DeptCode`、`AmpmCode`，拿回 div 卡片。解析器重寫，fixture 用實際抓到的 markup 重建。
2. **用叫號號碼決定監看區間，不再用時鐘。** 號碼第一次出現進入監看、連兩次空白結束監看。開放早/午/晚三個時段都能監看，時段由 `AmpmCode` 參數帶，不再需要「已驗證的時數」當前提。

## 原本預期做什麼？為什麼實際做出來的不一樣？

[Issue #34](https://github.com/KLTsai/next-is-you/issues/34) 原本寫的是：**「列第一次出現 → 進入監看；列消失 → 結束監看」**。

但開發過程中去實地讀台大醫院的公開進度頁，發現兩件事推翻了這個前提：

- **列不是活躍信號。** 下午診的列中午 12:05 就全部掛在上面了（號碼空白），上午診的列 12:03 過了名目結束時間還全部在。列代表「今天這個時段排了這一診」，不代表「正在看診」。
- **真正的進出場信號是「號碼」。** 03 診 12:03 的號碼是 045、12:20 變空白；11 診 12:20 是 018、12:39 變空白。兩間的卡片都還在。

所以實作改用**叫號號碼**取代「列」當作監看邊界。這個差異也回頭修正了 Issue #34 的驗收條件。

## Double Diamond 四階段實際發生的事

### Discover

讀了 issue、原始碼、既有測試，再**實際連到台大醫院的公開看診進度頁**做了約 20 次讀取，發現：

1. 需求的前提是錯的（見上一節）。
2. reader 根本讀不到真實來源。原本發普通 GET 然後找 `<table>`；真實頁面那個 GET 回來沒有任何診的列，整頁零個 table。資料要用 `POST DeptLightTable` 才拿得到，回傳是 div 卡片不是表格。
3. 真正的結束信號是「號碼變空白、卡片留著」。

第 3 點是鑽石重開第二次才找到的 — 過程見下方「人類介入的兩個轉折」。

### Define

問題陳述從「把時鐘換成列的出現消失」修正為：

> 監看區間應該由該診的叫號號碼決定，不是時鐘、也不是列的存在。
> 號碼第一次出現進入監看、號碼消失結束監看。
> 而且在那之前，reader 必須先能真的讀到來源。

### Develop

範圍切成兩刀，各自可獨立審查：

1. 先讓 reader 讀得到來源（兩步驟請求 + 卡片解析器 + 用真實 markup 重建 fixture）。
2. 再拿掉時鐘（監看邊界改由號碼決定）。

同時評估並**排除**了幾個做法：用明細頁的「已叫最大號 / 已報到」判斷結束（明細頁的 `ServiceIDSE` 不穩定、英文版沒有醫師名、23:51 曾觀察到空白欄位配新時間戳會騙人）；三個時段都發請求來推斷活躍時段（請求量變三倍，違反專案「每 2-3 分鐘一次」的節制承諾）。

### Deliver

- 解析器用**實際抓下來的真實頁面**驗證，19 列全部正確解析。
- fixture 用真實 markup 結構重建，內容合成。
- 255 個測試 + 全部 gate 通過才 push。
- 誠實標示證據等級：fixture/本地整合 ✅、人工實地擷取 ✅、**reader 對線上來源的行為 ❌ 完全未驗證**。

## 人類介入的兩個轉折

這是最值得記錄的部分 — **兩次把方向拉回來的都是人，不是流程。**

**轉折一**：Agent 一開始就問了三個解法層面的問題（稀疏輪詢間隔、結束判定緩衝、要不要開放早晚診），**那時它連來源都還沒看過**。其中「列消失要不要馬上判定結束」整題建立在不存在的前提上。人類的回應不是直接回答選項，而是把流程推回 Discover —— 要求去實際看看台大總院眼科部當下門診的樣貌，比較已結束的診與還在看的診有什麼差別。

**轉折二**：Agent 查完後得出「來源沒有結束信號」的結論並準備以此收工。人類提供了一張當下的進度頁截圖，顯示哪些上午診已結束、哪些還在跑。對照之後才發現結束信號一直都在，只是會消失的東西是**號碼**不是**列**。Agent 原本的結論是錯的。

## 最終的 commit 紀錄

**審查時的兩則 commit**（在 PR 分支上，依 Conventional Commits 拆開以便逐則審查）：

- [`724532d`](https://github.com/KLTsai/next-is-you/commit/724532d) `fix(next_is_you): read the NTUH clinic list through its real request`
- [`443a68b`](https://github.com/KLTsai/next-is-you/commit/443a68b) `feat(next_is_you): watch a clinic by its number, not by the clock`

共 9 個檔案、+829 / -247 行。

**合入 main 的 squash commit**：

- [`cd199d7`](https://github.com/KLTsai/next-is-you/commit/cd199d7) `feat(next_is_you): read the live clinic list and bound the watch by its number (#46)`

type 選擇的理由：

- `724532d` 為什麼是 `fix`：reader 從來讀不到真實來源，這是真實缺陷，不是重構。雖然功能在 feature flag 後、production held，沒有使用者經歷過這個 bug，但缺陷性質決定 type，不是嚴重度。
- `443a68b` 為什麼是 `feat`：它開放了原本被拒絕的早診與夜診，是新能力。
- squash commit 的 type 選了 `feat`：使用者可見的淨變化是新能力（開放三個時段 + 新的監看規則），fix 部分只是前置條件。

兩則審查 commit 都附 `Refs: #34` footer，body 寫動機與對比而非複述 diff。沒有破壞相容性的變更，所以沒有 `!` 標示。

### 為什麼合入時用 squash

這個 repo 的 main 歷史全是 squash merge（每個 commit 只有一個 parent、尾綴 `(#NN)`）。跟著 house style 走，而不是把兩則細顆粒 commit 推上 main。審查時的拆解仍然保留在 PR #46 的 commit 歷史裡，追查時仍找得到。這與 TBD「linear history + 短命分支」的建議一致。

## Trunk-Based 執行核對

| 規則 | 做到了嗎 | 證據 |
|---|---|---|
| 短命分支 | ✅ | 同一個 session 內開分支、commit、push、開 PR |
| push 前跑完整本機 build | ✅ | 255 測試 + ruff format + ruff check + tsc + migrations + UI 測試 + build + `git diff --check` |
| 每個 commit 都不弄壞 build | ✅ | 兩個 commit 各自獨立通過 |
| 保持 trunk 可發布 | ✅ | CI `deterministic-gates` 綠燈才請求 merge |
| feature flag 藏未完成功能 | ✅ | reader 在 `NTUH_READER_ENABLED` 後、production held（專案既有設計，本次維持） |
| 整合前先同步 trunk | ❌ | 整個過程沒 fetch 過 main（事後確認 main 沒動是運氣，不是管理） |
| 不相關變更拆開提交 | ⚠️ | `443a68b` 其實包含「開放三個時段」與「改用號碼判斷監看」兩件事，可以再拆 |

## 這次暴露的 skill 可改進點

每一項都指到：**哪一支 skill 的哪一條原則沒被徹底落實**，以及這支 skill 可以怎麼修。

### 1. `/double-diamond-principle` — Discover 階段需要明確禁止「問解法層面的問題」

**從哪看出**：Agent 在 Discover 完成前就問了三個解法選擇題（間隔、緩衝、時段），其中一題整題報廢。

**skill 現況**：目前寫「核心紀律是『先找對問題，再找對解法』」，但沒有把它變成明確的執行禁令。

**建議新增規則**：

> 在問題陳述（Define 腰身）定案前，不要向使用者丟出解法選擇題（例如「要用 A 做法還是 B 做法」、「參數設 N 還是 M」）。可以列出 Discover 要調查的事實清單，但不能把尚未定義的解法參數變成選擇題。

### 2. `/double-diamond-principle` — Discover 需要補一條「第一站是專案的證據目錄」

**從哪看出**：答案本來就在 `research/private/source-evidence/ntuh-oph-morning-session-2026-10-28.md`（前一天的觀察記錄，明文標註「for example Issue #34」），Agent 卻是問完三個問題後 grep 才偶然找到它。

**skill 現況**：Discover 階段的常用方法只列了「user research、田野觀察、stakeholder interviews」等外部活動。

**建議新增規則**：

> 在接觸外部來源或使用者之前，先掃過專案自身的證據與研究檔案目錄（例如 `research/`、`docs/evidence/`）。團隊可能已經記錄過與本次問題直接相關的事實；跳過這一步會重做別人已經做完的功課。

### 3. `/double-diamond-principle` — Discover 需要指引「使用外部資源的量體要先宣告」

**從哪看出**：11 分鐘內對醫院公開頁面發了約 14 次請求，超出專案「每 2-3 分鐘一次」節制承諾約 3 倍。最後被來源回了 `ERR_CONNECTION_TIMED_OUT` 才停手。

**skill 現況**：Discover 沒有關於「調查量體」的指引。

**建議新增規則**：

> 動用外部資源（網站、API、人力訪談）前，先宣告預計用量上限（例如「我會讀 N 次、間隔 ≥ M 分鐘」），並按宣告執行；過程中若發現需要超出，先停下來與使用者確認，不要自己默默擴張。

### 4. `/git-conventional-commit` — 「相同 type 但主題不同的多件事也要拆」要寫得更明確

**從哪看出**：`443a68b` 同時做了「開放三個時段」與「改用號碼判斷監看」兩件事。兩件都是 `feat`，不衝突 type，但主題不同，可以也應該拆成兩則：

```
feat: accept every session via AmpmCode
feat: bound the watch by the clinic's number
```

**skill 現況**：寫「如果一次變更同時符合多個 type，優先拆成多個 commit」。這條只提到 type。

**建議改寫**：

> 如果一次變更同時符合多個 type **或對應多個獨立主題**，優先拆成多個 commit，讓每則 commit 聚焦。判準：這則 commit 的 description 寫起來需要兩個子句才能講清楚嗎？若是，就該拆。

### 5. `/trunk-based-development` — 開分支那一刻需要明確要求 fetch main

**從哪看出**：整個開發過程完全沒 `git fetch` 過 main。事後確認 main 沒動是運氣，不是流程。

**skill 現況**：寫「頻繁 clone / pull trunk，約每小時整合一次」，但偏「過程中」的規則。開分支那一刻缺一條明確要求。

**建議在「執行規則」第 1 條補**：

> 開新分支、或在既有短命分支上恢復工作前，先 `git fetch` 並以最新的 trunk 作為基準點，而不是以本機舊版本為基準。這一步也是確認「分支還短命」最便宜的方法。

## 衍生產出

Agent 在過程中同時修正了需求本身，而不只是寫程式：

- **改寫 [Issue #34](https://github.com/KLTsai/next-is-you/issues/34)**：標題從「derive the active session」改成「bound the watch from the live source」（前者做不到），body 加入 6 條實地證據、把驗收條件分成「已交付」與「已決定不改」，並記錄 3 個已知缺口。
- **新增證據檔** `research/private/source-evidence/ntuh-oph-clinic-end-2026-10-29.md`，13 筆讀取逐筆記錄（註：該目錄被 git 排除，僅存在本機）。
- **更新 `docs/NTUH-READER.md`**：請求契約、監看規則、排錯步驟、兩件「絕對不能做」的事。
