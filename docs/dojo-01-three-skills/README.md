# 道館一：三支 Skills 組合實戰

把 `/double-diamond-principle`、`/git-conventional-commit`、`/trunk-based-development` 三支 skill 組合起來，在同一個專案上連續開發 5 到 8 個功能，每次只做一個，並以人類視角觀察 Agent 是否真的遵循這三套原則。

- **實作對象**：[next-is-you](https://github.com/KLTsai/next-is-you) — 替家人盯台灣醫院看診叫號進度、號碼接近時發 LINE 提醒的私人小工具
- **三支 skill 的本體**：[`.claude/skills/`](../../.claude/skills/)、[`.agents/skills/`](../../.agents/skills/)

## 三支 skill 如何在本道館被使用

每一次開發功能都走同一套流程，不會每個功能重複寫這段 — 這裡講一次就好。

**啟用順序（固定）**：

```
/double-diamond-principle  →  /git-conventional-commit  →  /trunk-based-development
```

| Skill | 本道館的作用 |
|---|---|
| `/double-diamond-principle` | 收到需求的第一時間啟用。要求 Agent 在動工前先 Discover（挖事實）→ Define（鎖問題陳述）→ Develop（想做法）→ Deliver（驗證），而不是直接開寫。 |
| `/git-conventional-commit` | 在要寫 commit 訊息時啟用。決定 type（`fix` / `feat` / `docs` / …）、是否標 `!` 破壞性變更、`BREAKING CHANGE:` footer、以及 `Refs: #<issue>` 等 token 寫法。 |
| `/trunk-based-development` | 在決定開不開分支、分支活多久、什麼時候整合、未完成功能怎麼安全進 trunk 時啟用。 |

每次都用這組 skill、依前面順序啟用，再把當次要做的功能需求交給 Agent。提示本身故意保持簡短 — 開發細節由三支 skill 的流程推動，不塞進 prompt。

## 功能開發紀錄

| # | 功能 | 需求單 | 產出 | 狀態 | 紀錄 |
|---|---|---|---|---|---|
| 1 | 讓監看區間由叫號號碼決定，不再寫死時鐘時間 | [#34](https://github.com/KLTsai/next-is-you/issues/34) | [PR #46](https://github.com/KLTsai/next-is-you/pull/46)<br>main：[`cd199d7`](https://github.com/KLTsai/next-is-you/commit/cd199d7) | ✅ 已合入 main | [詳細紀錄](./feature-01-watch-window.md) |
| 2 | 讓系統不再只懂台大一家，把每家醫院的讀取方式獨立成可替換模組 | [#45](https://github.com/KLTsai/next-is-you/issues/45) | [PR #48](https://github.com/KLTsai/next-is-you/pull/48)<br>main：[`5a96d4f`](https://github.com/KLTsai/next-is-you/commit/5a96d4f) | ✅ 已合入 main | [詳細紀錄](./feature-02-hospital-adapter-boundary.md) |
| 3 | 讀出某科當天在看診的醫生與診間 | [#47](https://github.com/KLTsai/next-is-you/issues/47)（功能二在 Define 階段從 #45 切出） | [PR #49](https://github.com/KLTsai/next-is-you/pull/49)<br>main：[`bec72a5`](https://github.com/KLTsai/next-is-you/commit/bec72a5) | ✅ 已合入 main | [詳細紀錄](./feature-03-clinic-roster.md) |
| 4 | 報到設定改成用選的，只剩看診號碼要手填 | [#47](https://github.com/KLTsai/next-is-you/issues/47)（同上） | [PR #54](https://github.com/KLTsai/next-is-you/pull/54)<br>main：[`73260e2`](https://github.com/KLTsai/next-is-you/commit/73260e2) | ✅ 已合入 main | [詳細紀錄](./feature-04-cascading-setup-ui.md) |
| 5 | 送出前擋掉填錯的看診資料 | [#45](https://github.com/KLTsai/next-is-you/issues/45) 的 AC #4，另切 [#58](https://github.com/KLTsai/next-is-you/issues/58) 作為聚焦票 | [PR #59](https://github.com/KLTsai/next-is-you/pull/59)<br>main：[`514b03f`](https://github.com/KLTsai/next-is-you/commit/514b03f) | ✅ 已合入 main | [詳細紀錄](./feature-05-preflight-validation.md) |

## 完成條件核對

| 條件 | 狀態 |
|---|---|
| 三支 Skill 的完整資料夾 | ✅ 已 commit 並推上 remote（[`.claude/skills/`](../../.claude/skills/)、[`.agents/skills/`](../../.agents/skills/)） |
| 同一專案完成 5-8 個功能的 git commits 紀錄 | ✅ 5 / 5-8（功能 1-5 已完成，滿足下限） |
| 人類觀察：哪些原則被遵循、哪些沒有、優化空間 | ✅ 功能 1-5 均有完整觀察紀錄，見[累積的 skill 改進提案](#累積的-skill-改進提案)與[道館流程本身的觀察](#道館流程本身的觀察一功能一-session) |
| 一分鐘介紹影片 | ❌ 未製作 |

## 對照作業的觀察項目

作業原文的「你要觀察什麼」列了四件事：開發前的流程（Double Diamond）、整合方式（trunk）、commit 的格式與拆分、完成時的回報。下表把五個已完成的功能逐項對照，✅ 表示有看到，⚠️ 表示部分做到或有缺口。每格的證據在該功能的紀錄裡。

| 觀察項目 | [功能一](./feature-01-watch-window.md) | [功能二](./feature-02-hospital-adapter-boundary.md) | [功能三](./feature-03-clinic-roster.md) | [功能四](./feature-04-cascading-setup-ui.md) | [功能五](./feature-05-preflight-validation.md) |
|---|---|---|---|---|---|
| 開發前理解問題、收斂範圍、提出做法、選定後小範圍測試、取得新資訊時回頭修正 | ✅ Discover 重開兩次，前提被推翻後修正 | ⚠️ 需求單收斂並提出範圍選項；設計層級沒看到並列比較的做法 | ⚠️ 同左，且沒有需要回頭修正的新資訊 | ✅ 動工前做原型審閱，六項決策；Q5 與自己引入的硬寫格式缺陷都回頭修正 | ✅ Discover 因使用者質疑重開一次，錯誤的來源宣稱被推翻，範圍跟著擴大 |
| 一次一個足夠小的功能、短命分支、整合前跑測試、trunk 可發布、未完成變更用 flag 或 branch by abstraction | ✅ 短命分支、255 測試；⚠️ 沒 fetch main | ✅ branch by abstraction；⚠️ +393 / -23、11 檔案 | ⚠️ 一度疊在未合入的分支；+493 / -12、10 檔案 | ✅ CI 偶發失敗修根因；⚠️ +484 / -87、11 檔案 | ✅ branch-by-abstraction、293+18 測試全綠才合併；⚠️ +1689 / -60、16 檔案，比功能四更大；⚠️ push 前漏跑 ruff format，CI 紅一輪 |
| 不相關變更拆開、Conventional Commits 格式與 type、破壞性變更標示 | ⚠️ 一則 commit 含兩個主題 | ⚠️ `git add -A` 掃進無關改動 | ⚠️ 一則標題 82 字元，被 GitHub 截斷 | ✅ 四則依主題拆開 | ✅ 六則依主題拆開，標題在字元限制內 |
| 完成時簡短指出 Double Diamond 怎麼用、測試結果、branch 與整合狀態、實際 commit message | ✅ 說明實作與原要求的差異與原因（實地讀取推翻了「列」是活躍信號）、PR 與 CI 狀態、兩則 commit 的 type 與用途；測試只說 CI 綠 | ⚠️ 有測試與分支；Double Diamond 只一句帶過；沒有 commit 原文 | ⚠️ 有測試與分支；沒提 Double Diamond；沒有 commit 原文 | ⚠️ 同左 | ⚠️ 有 PR、merge 與測試狀態；完成總結訊息沒有重新講 Double Diamond 怎麼用，也沒有在那段訊息裡列 commit 原文 |

作業另外要求「每一次只開發一個功能」。功能二到四是同一個 session 連著做完的，見[道館流程本身的觀察](#道館流程本身的觀察一功能一-session)；功能五依下方給下一輪的執行規則，獨立開一個新 session，合入 main 並寫完觀察紀錄後就停下，沒有接著規劃功能六。

## 累積的 skill 改進提案

從實際使用中發現「skill 自身可以修」的地方，依 skill 聚合。「細節」欄的「提案 N」，對應該功能紀錄裡「這次暴露的 skill 可改進點」底下的第 N 條，點進去看完整脈絡。

### `/double-diamond-principle`

| 建議改動 | 提出時機 | 細節 |
|---|---|---|
| Discover 階段明確禁止「問解法層面的問題」 | 功能一 | [提案 1](./feature-01-watch-window.md#1-double-diamond-principle--discover-階段需要明確禁止問解法層面的問題) |
| Discover 補一條「第一站是專案的證據目錄」 | 功能一 | [提案 2](./feature-01-watch-window.md#2-double-diamond-principle--discover-需要補一條第一站是專案的證據目錄) |
| Discover 補「使用外部資源的量體要先宣告」 | 功能一 | [提案 3](./feature-01-watch-window.md#3-double-diamond-principle--discover-需要指引使用外部資源的量體要先宣告) |
| Deliver 要有「交付完就停」的規則，agent 自己會提議下一個功能 | 功能二 | [提案 2](./feature-02-hospital-adapter-boundary.md#2-double-diamond-principle---deliver-沒有交付完就停的規則agent-自己會提議下一個功能) |
| 不要把系統的嚴格要求直接轉嫁給使用者（要同時給取得正確值的路徑） | 功能四 | [提案 1](./feature-04-cascading-setup-ui.md#1-double-diamond-principle---不要把系統的嚴格要求直接轉嫁給使用者) |
| 完成時要回報四件事：流程怎麼用、測試、整合狀態、commit 原文（三支 skill 皆適用） | 功能四 | [提案 3](./feature-04-cascading-setup-ui.md#3-double-diamond-principle---沒有要求完成時回報怎麼用了這個流程三支-skill-皆適用) |
| Discover 對外部來源的事實宣稱要先核對，不能只靠記憶 | 功能五 | [提案 1](./feature-05-preflight-validation.md#1-double-diamond-principle---discover-對外部來源的事實宣稱要先核對不能只靠記憶) |

### `/git-conventional-commit`

| 建議改動 | 提出時機 | 細節 |
|---|---|---|
| 「相同 type 但主題不同的多件事也要拆」要寫得更明確 | 功能一 | [提案 4](./feature-01-watch-window.md#4-git-conventional-commit--相同-type-但主題不同的多件事也要拆要寫得更明確) |
| 自我檢查要加一條「commit 前核對 staged 檔案清單」（目前只檢查訊息格式） | 功能二 | [提案 1](./feature-02-hospital-adapter-boundary.md#1-git-conventional-commit---自我檢查要加一條commit-前核對-staged-檔案清單) |
| 標題長度要有數字上限（72 字元），目前只寫「精簡」 | 功能三 | [提案 1](./feature-03-clinic-roster.md#1-git-conventional-commit---沒有-subject-長度的規則commit-標題被-github-截成兩半) |

### `/trunk-based-development`

| 建議改動 | 提出時機 | 細節 |
|---|---|---|
| 開分支那一刻明確要求 fetch main | 功能一 | [提案 5](./feature-01-watch-window.md#5-trunk-based-development--開分支那一刻需要明確要求-fetch-main) |
| 「小」要有大小標準，目前只有時間上限 | 功能三 | [提案 2](./feature-03-clinic-roster.md#2-trunk-based-development---小只有時間上限沒有大小標準) |
| 「不穩的測試」不能跟「慢的測試」一樣往 pipeline 後段搬，要當壞掉的 build 修 | 功能四 | [提案 2](./feature-04-cascading-setup-ui.md#2-trunk-based-development---不穩的測試不能跟慢的測試一樣處理) |
| push 前的本機 gate 清單要跟專案 CI 的 deterministic-gates 對齊，不能只寫「跑完整 build」 | 功能五 | [提案 2](./feature-05-preflight-validation.md#2-trunk-based-development---push-前的本機-gate-清單要跟專案-ci-的-deterministic-gates-對齊) |

## 道館流程本身的觀察：一功能一 session

這一條不屬於三支 skill 任何一支的內容，是**道館實驗協定**沒有被帶進 session 的紀錄，所以留在這裡，不歸進上面任何一支 skill。

本道館的設計是「每次只做一個功能、每個功能開新的 session，分開觀察 agent 對三支 skill 的遵循程度」。這個想法在功能一的 session 結尾由人類說過（「之後會開新的 session 來製作剩下的功能」），但沒有寫進功能二那個 session 的開場 prompt（只有「第二個功能（#45 issue）」），agent 也沒有讀過道館的功能表。

結果功能二、三、四是在**同一個 session** 裡做完的：功能二的 PR（#48）開出、CI 還在跑時，agent 主動問「要等 CI 綠就先停，還是先去規劃 #47」，人類選了後者；之後功能三、功能四接著做，中途沒有分段。agent 是順著 issue 之間的 `Refs` / `Depends on` 連結往前推進，而不是照道館表格的功能編號。

連帶後果：功能五（送出前擋掉填錯的看診資料）沒有被涵蓋。它不是被合併進前三輪，是整個沒被當成題目 Discover 過；PR #54 附帶的兩條驗證是功能四選單模式的必要配套，不是功能五的範圍。完整說明見[功能 2/3/4 的共同觀察](./feature-04-cascading-setup-ui.md#功能-234-的共同觀察一個-session-連著跑完三輪)。

「一次只做一個功能」是作業本身的要求；三支 skill 能不能幫 agent 守住它，是另一件事 - 三支 skill 都沒有任何「交付後停下來」的文字，這個觀察記在功能二的[提案 2](./feature-02-hospital-adapter-boundary.md#2-double-diamond-principle---deliver-沒有交付完就停的規則agent-自己會提議下一個功能)。

**給下一輪的執行規則**：每個功能開一個新 session，開場 prompt 寫明道館的功能編號、對應的需求單，以及「做完這一個就停，不規劃下一個」。該功能合入 main 並寫完觀察紀錄、更新道館表格之後，才在新 session 開始下一個編號。agent 若發現某張 issue 能再拆成更小的切片，只回報切法，等人指派編號再動工。

**功能五驗證了這條規則有效**：功能五的 session 開場就先確認對應道館的哪個功能編號、讀了道館表格，動工前也先開了聚焦票 #58。PR #59 合入 main、觀察紀錄寫完、道館表格更新之後，session 就停下，沒有自己提議功能六。這跟功能二到四那次「agent 主動問要不要先規劃下一個」的情況不同 - 差別在於這次開場 prompt 明確要求「做完這一個就停」，agent 也先讀過道館表格，而不是只拿到 issue 編號就順著 issue 連結往前推進。

## 本道館最值得記下的一件事

**兩次把方向拉回來的都是人，不是流程。** 功能一的開發過程中，如果沒有人類兩次介入（一次要求去看實際看診狀況、一次提供進度頁截圖），Agent 會交出一個「用列的出現消失判斷監看」的版本 — 在 fixture 上全綠，在真實環境永遠不會結束監看。

Double Diamond 整套方法就是為了避免這件事，而這次是人擋下來的。這也說明 skill 不是保證，它只是讓 Agent 更容易被問對問題。

**這個模式在功能五重複出現。** 功能五的 Discover 同樣對外部來源（NTUH 的公告範圍）做了一個沒有查證的事實宣稱，又是使用者用常識質疑才被推翻、重新查證後找到真正的來源能力。五個功能裡，兩次需要人類擋下的錯誤都是同一種：**對外部系統的事實宣稱，沒有先查證就當作前提**。這個缺口已經記在功能五的提案 1，但連續兩次被同一種錯誤卡住，說明這條規則的優先序應該提高，不只是列在提案清單裡等待採納。

這也是為什麼上面的「累積的 skill 改進提案」要認真做：**每一條改進都在把這次需要人介入才避開的坑，變成下次 skill 本身就會避開的坑。**
