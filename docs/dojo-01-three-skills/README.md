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
| 5 | 送出前擋掉填錯的看診資料 | [#45](https://github.com/KLTsai/next-is-you/issues/45) | - | ❌ 未開始（功能 2-4 那一輪沒有涵蓋到它） | 見[功能 2/3/4 的共同觀察](./feature-04-cascading-setup-ui.md#功能-234-的共同觀察一個-session-連著跑完三輪) |

## 完成條件核對

| 條件 | 狀態 |
|---|---|
| 三支 Skill 的完整資料夾 | ✅ 已 commit 並推上 remote（[`.claude/skills/`](../../.claude/skills/)、[`.agents/skills/`](../../.agents/skills/)） |
| 同一專案完成 5-8 個功能的 git commits 紀錄 | 🚧 4 / 5（功能 5 尚未開工） |
| 人類觀察：哪些原則被遵循、哪些沒有、優化空間 | 🚧 隨各功能紀錄累積，見[累積的 skill 改進提案](#累積的-skill-改進提案)與[道館流程本身的觀察](#道館流程本身的觀察一輪在哪裡結束) |
| 一分鐘介紹影片 | ❌ 未製作 |

## 累積的 skill 改進提案

從實際使用中發現「skill 自身可以修」的地方，依 skill 聚合。每一條都註明是哪一次功能發現的，點進去看完整脈絡。

### `/double-diamond-principle`

| 建議改動 | 提出時機 | 細節 |
|---|---|---|
| Discover 階段明確禁止「問解法層面的問題」 | 功能一 | [紀錄 §7-1](./feature-01-watch-window.md#1-double-diamond-principle--discover-階段需要明確禁止問解法層面的問題) |
| Discover 補一條「第一站是專案的證據目錄」 | 功能一 | [紀錄 §7-2](./feature-01-watch-window.md#2-double-diamond-principle--discover-需要補一條第一站是專案的證據目錄) |
| Discover 補「使用外部資源的量體要先宣告」 | 功能一 | [紀錄 §7-3](./feature-01-watch-window.md#3-double-diamond-principle--discover-需要指引使用外部資源的量體要先宣告) |
| Define 切出去的半邊，要同時定下「重新進場的條件」 | 功能二 | [紀錄](./feature-02-hospital-adapter-boundary.md#這次暴露的-skill-可改進點) |
| Develop 的原型必須涵蓋降級與失敗路徑，不只順利路徑 | 功能四 | [紀錄](./feature-04-cascading-setup-ui.md#這次暴露的-skill-可改進點) |
| 不要把系統的嚴格要求直接轉嫁給使用者（要同時給取得正確值的路徑） | 功能四 | [紀錄](./feature-04-cascading-setup-ui.md#這次暴露的-skill-可改進點) |

### `/git-conventional-commit`

| 建議改動 | 提出時機 | 細節 |
|---|---|---|
| 「相同 type 但主題不同的多件事也要拆」要寫得更明確 | 功能一 | [紀錄 §7-4](./feature-01-watch-window.md#4-git-conventional-commit--相同-type-但主題不同的多件事也要拆要寫得更明確) |
| 自我檢查要加一條「commit 前核對 staged 檔案清單」（目前只檢查訊息格式） | 功能三 | [紀錄](./feature-03-clinic-roster.md#這次暴露的-skill-可改進點) |

### `/trunk-based-development`

| 建議改動 | 提出時機 | 細節 |
|---|---|---|
| 開分支那一刻明確要求 fetch main | 功能一 | [紀錄 §7-5](./feature-01-watch-window.md#5-trunk-based-development--開分支那一刻需要明確要求-fetch-main) |
| 定義「一輪工作的結束點」：合入 trunk 且紀錄完成前，不開始下一輪 | 功能二 | [紀錄](./feature-02-hospital-adapter-boundary.md#1-trunk-based-development---需要定義一輪工作的結束點) |
| 順手整合待辦的零碎改動要獨立成 commit，不併入功能 commit | 功能三 | [紀錄](./feature-03-clinic-roster.md#這次暴露的-skill-可改進點) |
| 「不穩的測試」不能跟「慢的測試」一樣往 pipeline 後段搬，要當壞掉的 build 修 | 功能四 | [紀錄](./feature-04-cascading-setup-ui.md#這次暴露的-skill-可改進點) |

## 道館流程本身的觀察：一輪在哪裡結束

這一條不屬於三支 skill 任何一支的內容，是**道館執行流程本身**該補的規則，留在這裡而不是歸進上面任何一支 skill。

本道館的設計是「每次只做一個功能，分開觀察 agent 對三支 skill 的遵循程度」。功能 2/3/4 實際上是在同一條連續的工作線上跑完的：功能二的 PR（#48）開出、CI 還在跑時就開始規劃 #47（功能三與功能四的來源需求單），功能三完成後緊接著做功能四，中途沒有分段。agent 是順著 issue 之間的 `Refs` / `Depends on` 連結往前推進，沒有拿道館表格的功能編號核對範圍邊界 - issue 的相依邊界與道館的觀察邊界是兩件不同的事。

連帶後果：功能五（送出前擋掉填錯的看診資料）沒有被涵蓋。它不是被合併進前三輪，是整個沒被當成題目 Discover 過；PR #54 附帶的兩條驗證是功能四選單模式的必要配套，不是功能五的範圍。完整說明見[功能 2/3/4 的共同觀察](./feature-04-cascading-setup-ui.md#功能-234-的共同觀察一個-session-連著跑完三輪)。

這個現象不是三支 skill 內容本身的問題 - 三支 skill 在每一輪的單一功能範圍內都有被遵循的證據（見三份紀錄的核對表）。缺的是「一輪在哪裡結束」這條規則，而這正是[功能二](./feature-02-hospital-adapter-boundary.md#1-trunk-based-development---需要定義一輪工作的結束點)與[功能四](./feature-04-cascading-setup-ui.md#這次暴露的-skill-可改進點)各自提出的 skill 建議要補上的東西。

**給下一輪的執行規則**：每次只交給 agent 道館表格裡的一個功能編號；該功能合入 main 後先停下來，確認觀察紀錄與表格都更新完，才開下一個編號。agent 若發現某張 issue 能再拆成更小的切片，只回報切法（「這張可以拆成 N 個道館功能」），等人指派編號再動工。

## 本道館最值得記下的一件事

**兩次把方向拉回來的都是人，不是流程。** 功能一的開發過程中，如果沒有人類兩次介入（一次要求去看實際看診狀況、一次提供進度頁截圖），Agent 會交出一個「用列的出現消失判斷監看」的版本 — 在 fixture 上全綠，在真實環境永遠不會結束監看。

Double Diamond 整套方法就是為了避免這件事，而這次是人擋下來的。這也說明 skill 不是保證，它只是讓 Agent 更容易被問對問題。

這也是為什麼上面的「累積的 skill 改進提案」要認真做：**每一條改進都在把這次需要人介入才避開的坑，變成下次 skill 本身就會避開的坑。**
