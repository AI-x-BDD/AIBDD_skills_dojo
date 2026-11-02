# 功能四：報到設定改成用選的，只剩看診號碼要手填

| 項目 | 內容 |
|---|---|
| 開發日期 | 2026-10-31 |
| 實作專案 | [next-is-you](https://github.com/KLTsai/next-is-you) |
| 需求來源 | [Issue #47](https://github.com/KLTsai/next-is-you/issues/47)（47b 前端切片，依賴功能三的名冊後端） |
| 產出 | [PR #54](https://github.com/KLTsai/next-is-you/pull/54) ✅ squash merge 至 main：[`73260e2`](https://github.com/KLTsai/next-is-you/commit/73260e2) |
| 分支 | `feat/47b-cascading-setup-ui`（短命分支，同日開分支、commit、開 PR、合入） |
| 測試結果 | 269 passed（含瀏覽器驗收與 rehearsal E2E 重寫）、ruff format/check、tsc、test:ui 13 passed、vite build 全通過 |
| 三支 skill 怎麼被使用 | [見道館總覽](./README.md#三支-skill-如何在本道館被使用) |

## 這次做了什麼

把設定表單從「每格都手打」換成「串接式選取」：

- 醫院清單、顯示名稱、支援的科別、capability 全部來自後端型錄（`GET /api/organizer/hospitals`），前端不再帶任何醫院專屬常數；移除原本寫死的 `hospitalNames`、`ntuh-main` 預設值、台大專屬的說明文字。
- 流程改成：醫院 → 科別＋日期 → （自動抓名冊，附載入狀態）→ 醫師/診間下拉（時段自動帶出）→ 只填看診號碼。
- 五種狀態：完整名冊、載入中、降級手動、醫院不支援、送出前驗證。
- 名冊不可用時（#22 試點前讀取被 held、約診日不是今天、讀取失敗）優雅降級為手動輸入，附來源進度頁連結與即時解析預覽（把輸入拆成診間/醫師/門診別秀出來，但不擋格式——格式是醫院專屬知識，刻意留給後端 Start 擋）。
- 送出前驗證：日期不能選過去；roster 模式下選到的醫師/診間必須真的在名冊裡。

## 原本預期做什麼？為什麼實際做出來的不一樣？

這一輪開工前，先用 Lavish 互動原型跑了一次使用者審閱，鎖定六個 UX 決策（版面不重新設計、醫院清單全列出但不支援的灰掉標註「尚未支援」、未來日期自動轉手動輸入、選完科別+日期自動抓名冊不用按鈕、手動輸入用單欄＋參考連結＋解析預覽、科別清單只列後端宣告支援的）。這六點完整記錄在 [#47 的實作規格留言](https://github.com/KLTsai/next-is-you/issues/47#issuecomment-6538923721)，實作基本照這份規格走，沒有再發生認知修正。

唯一的執行面意外：CI 在較慢的 runner 上偶發失敗（儲存成功訊息先出現，列表還沒重新整理完成就被斷言檢查），本機跑不出來。這不是功能需求的誤解，是測試本身對「非同步重整完成時機」假設過窄，用 CPU 節流重現後，改用 Playwright 的自動重試 `expect()` 取代一次性斷言修好。修這個不是使用者要求的——是依照專案既有規則（看到測試不穩定就要修，不管是不是本次任務造成的）主動做的。

## Double Diamond 四階段實際發生的事

### Discover + Develop（用 Lavish 原型跑的這一段）

沒有再去源站重新讀取資料，Discover 改成「跟使用者對齊 UX 決策」：為什�么科別只列眼科、診間選取語意是什麼、用精確比對還是關鍵字比對。這些問題透過 Lavish 的互動原型逐條問出來，六個決策鎖定後才開始寫程式——這點符合 double-diamond「解法參數在問題陳述定案前不要丟給使用者選」這條[功能一提出的建議](./feature-01-watch-window.md#1-double-diamond-principle--discover-階段需要明確禁止問解法層面的問題)的精神：先問清楚語意與邊界（Discover），再把選項收斂成決策（Define），不是一開始就丟「要不要重新設計 UI」這種大哉問。

### Define

> 送出前要能分辨「選單模式」（roster 可用，使用者只選不打字）跟「手動模式」（roster 不可用，使用者打字但有解析預覽輔助），兩種模式的驗證邊界不同，且手動模式的格式正確性不是前端的責任。

### Deliver

- `tests/test_local_acceptance.py` 瀏覽器驗收重寫：真實挑日期、挑診間字串、檢查晚診文字可見。
- `tests/test_rehearsal.py` UI 測試調整：填診間欄位、科別改用下拉選取後的斷言方式。
- 269 個測試全綠才 push，過程中額外修好一個與本次改動無關但被放大的既有 flaky 測試。

## 最終的 commit 紀錄

四則 commit：

- [`03164c2`](https://github.com/KLTsai/next-is-you/commit/03164c2) `chore: ignore the .lavish agent review directory`
- [`00b23fa`](https://github.com/KLTsai/next-is-you/commit/00b23fa) `feat(next_is_you): add supported departments to the hospital catalog`
- [`cbe60df`](https://github.com/KLTsai/next-is-you/commit/cbe60df) `feat(frontend): catalog-driven cascading reminder-setup form`
- [`a2dbff6`](https://github.com/KLTsai/next-is-you/commit/a2dbff6) `test: wait for the list reload before asserting the saved session`

**合入 main 的 squash commit**：[`73260e2`](https://github.com/KLTsai/next-is-you/commit/73260e2) `feat(frontend): catalog-driven cascading reminder-setup UI (#47) (#54)`

type 選擇的理由：`chore` 給純粹的 gitignore 規則；`feat` 拆成兩則——後端的科別型錄、前端的串接表單本體——因為兩者分屬不同檔案、不同主題，即使都是 `feat` 也拆開，符合[功能一的建議](./feature-01-watch-window.md#4-git-conventional-commit--相同-type-但主題不同的多件事也要拆要寫得更明確)；`test` 給那則與本功能需求無關、但同一次 push 前發現的 flaky 修復，單獨成則而不是塞進 feat commit 裡，讓之後回溯時容易分辨「這是功能、那是補測試穩定性」。

## Trunk-Based 執行核對

| 規則 | 做到了嗎 | 證據 |
|---|---|---|
| 短命分支 | ✅ | 同日開分支、commit、push、開 PR、合入 |
| push 前跑完整本機 build | ✅ | 269 測試 + 全部 gate |
| 保持 trunk 可發布 | ✅ | CI 綠燈才請求 merge |
| 壞掉就快速處理（這裡是 CI 不穩，不是壞掉的 build，但精神相同） | ✅ | 發現 flaky 測試後用節流重現、修好、驗證 3/3 再移除節流 |
| 整合前先同步 trunk | ⚠️ 未留下證據 | 同前兩輪，沒有明確 `git fetch` main 的紀錄 |

## 功能 2/3/4 共通觀察：道館的「一功能一輪」規則沒有被遵守

這是這三份補寫紀錄裡最重要的一條，寫在這裡統一講一次。

[道館總覽](./README.md)把 #45 原始需求單拆成四個獨立功能（2/3/4/5），目的是**一次只做一個，觀察 agent 對三支 skill 的遵循程度**。實際發生的事：

1. 功能二（PR #48）的 Deliver 階段——PR 開出、CI 還在跑、還沒合入——使用者就要求「CI 跑的同時先去規劃 #47」。也就是**功能三/四的 Discover 在功能二真正交付之前就開始了**，兩輪之間沒有留白可供觀察。
2. agent 自己提出「先做 47a 後端切片」的建議（即功能三），使用者同意後，功能三與功能四（47b）被當成同一張需求單（#47）的兩個切片接著做，而不是分別當成道館表格裡兩個獨立條目來跑。
3. 整個過程裡沒有一次停下來，像 `feature-01-watch-window.md` 那樣，在每個功能做完後寫觀察紀錄；三個功能、三個 PR，直到使用者事後發問「怎麼一次把 #45 的範圍都做完了」才回頭補寫。

**連帶後果：功能五被遺漏，不是被合併。** 功能五（送出前擋掉填錯的看診資料）跟 2/3/4 不一樣——它完全沒被當成一個題目處理過。功能四做完後，PR #54 附帶加了兩條驗證（日期不能選過去、roster 模式下選項必須存在），但那是**功能四的必要配套**，不是功能五的完整範圍。功能五真正要求的（手動輸入格式錯誤要擋、部門/醫師/診間組合不符的獨立驗證）至今沒有 Discover 過，而且手動格式驗證目前是刻意不擋（因為 #47 的規格要求前端不能硬寫醫院格式）。這記錄在 [#47 的收尾留言](https://github.com/KLTsai/next-is-you/issues/47#issuecomment-6559348742)。

### 根因

這不是三支 skill 內容本身的問題——double-diamond、conventional-commit、trunk-based 各自在單一功能範圍內都有被遵守的證據（見上面三份文件的核對表）。根因是**agent 沒有拿道館表格的功能編號逐項核對範圍邊界**，而是順著 issue 之間的 `Refs` / `Depends on` 連結（#45 → #47 → 47a → 47b）一路往前推進。Issue 之間的相依關係是真的，但道館要觀察的「一次一個」邊界跟 issue 拆分邊界是兩件不同的事，agent 把兩者混為一談。

### 建議：道館流程本身要補一條規則

這條不屬於三支 skill 任何一支，是道館執行流程本身的缺口：

> 每次只交給 agent 道館表格裡的一個功能編號。一個功能的 PR 合入 main 之後，**先停下來**，確認觀察紀錄已經寫完、道館表格已經更新，才開始下一個功能編號的 Discover。即使 agent 自己發現某張 issue 可以再拆成更小的切片，也要先回報「這張 issue 可以拆成 N 個道館功能」，等人類確認編號怎麼對應，再動工。
