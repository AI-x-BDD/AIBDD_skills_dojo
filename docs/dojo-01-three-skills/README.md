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
| 2 | 讓系統不再只懂台大一家，把每家醫院的讀取方式獨立成可替換模組 | [#45](https://github.com/KLTsai/next-is-you/issues/45) | [PR #48](https://github.com/KLTsai/next-is-you/pull/48)<br>main：[`5a96d4f`](https://github.com/KLTsai/next-is-you/commit/5a96d4f) | ✅ 已合入 main（⚠️ 紀錄為事後補寫，見下方流程缺口） | [詳細紀錄](./feature-02-hospital-adapter-boundary.md) |
| 3 | 讀出某科當天在看診的醫生與診間 | [#47](https://github.com/KLTsai/next-is-you/issues/47)（從 #45 拆出） | [PR #49](https://github.com/KLTsai/next-is-you/pull/49)<br>main：[`bec72a5`](https://github.com/KLTsai/next-is-you/commit/bec72a5) | ✅ 已合入 main（⚠️ 紀錄為事後補寫，見下方流程缺口） | [詳細紀錄](./feature-03-clinic-roster.md) |
| 4 | 報到設定改成用選的，只剩看診號碼要手填 | [#47](https://github.com/KLTsai/next-is-you/issues/47)（從 #45 拆出） | [PR #54](https://github.com/KLTsai/next-is-you/pull/54)<br>main：[`73260e2`](https://github.com/KLTsai/next-is-you/commit/73260e2) | ✅ 已合入 main（⚠️ 紀錄為事後補寫，見下方流程缺口） | [詳細紀錄](./feature-04-cascading-setup-ui.md) |
| 5 | 送出前擋掉填錯的看診資料 | [#45](https://github.com/KLTsai/next-is-you/issues/45) | — | ❌ 未開始，且被功能 2/3/4 的範圍擴張連帶遺漏 | 見[功能四的共通觀察](./feature-04-cascading-setup-ui.md#功能-234-共通觀察道館的一功能一輪規則沒有被遵守) |

## 完成條件核對

| 條件 | 狀態 |
|---|---|
| 三支 Skill 的完整資料夾 | ✅ 已 commit 並推上 remote（[`.claude/skills/`](../../.claude/skills/)、[`.agents/skills/`](../../.agents/skills/)） |
| 同一專案完成 5-8 個功能的 git commits 紀錄 | 🚧 4 / 5（功能 5 尚未開工） |
| 人類觀察：哪些原則被遵循、哪些沒有、優化空間 | 🚧 隨各功能紀錄累積，見[累積的 skill 改進提案](#累積的-skill-改進提案)與[道館流程本身的缺口](#道館流程本身的缺口功能-234-被連著做完才發現) |
| 一分鐘介紹影片 | ❌ 未製作 |

## 累積的 skill 改進提案

從實際使用中發現「skill 自身可以修」的地方，依 skill 聚合。每一條都註明是哪一次功能發現的，點進去看完整脈絡。

### `/double-diamond-principle`

| 建議改動 | 提出時機 | 細節 |
|---|---|---|
| Discover 階段明確禁止「問解法層面的問題」 | 功能一 | [紀錄 §7-1](./feature-01-watch-window.md#1-double-diamond-principle--discover-階段需要明確禁止問解法層面的問題) |
| Discover 補一條「第一站是專案的證據目錄」 | 功能一 | [紀錄 §7-2](./feature-01-watch-window.md#2-double-diamond-principle--discover-需要補一條第一站是專案的證據目錄) |
| Discover 補「使用外部資源的量體要先宣告」 | 功能一 | [紀錄 §7-3](./feature-01-watch-window.md#3-double-diamond-principle--discover-需要指引使用外部資源的量體要先宣告) |

### `/git-conventional-commit`

| 建議改動 | 提出時機 | 細節 |
|---|---|---|
| 「相同 type 但主題不同的多件事也要拆」要寫得更明確 | 功能一 | [紀錄 §7-4](./feature-01-watch-window.md#4-git-conventional-commit--相同-type-但主題不同的多件事也要拆要寫得更明確) |

### `/trunk-based-development`

| 建議改動 | 提出時機 | 細節 |
|---|---|---|
| 開分支那一刻明確要求 fetch main | 功能一 | [紀錄 §7-5](./feature-01-watch-window.md#5-trunk-based-development--開分支那一刻需要明確要求-fetch-main) |
| `git add -A` 前先核對 staged 清單，避免把不相關的零散改動掃進功能 commit | 功能三 | [紀錄](./feature-03-clinic-roster.md#最終的-commit-紀錄) |

## 道館流程本身的缺口：功能 2/3/4 被連著做完才發現

這一條不屬於三支 skill 任何一支的內容，是**道館執行流程本身**的缺口，留在這裡而不是歸進上面任何一支 skill。

本道館的規則是「每次只做一個功能，觀察 agent 對三支 skill 的遵循程度」。實際執行時，功能二（PR #48）的 CI 還沒跑完、PR 還沒合入，就已經被要求去規劃功能三/四（當時還是 #47 這張需求單）；功能三完成後緊接著做功能四，中間沒有停下來寫觀察紀錄，也沒有回頭核對道館表格的功能編號。三個功能、三個 PR 都合入 main 之後，才由人類發問「怎麼一次把範圍都做完了」，才回頭補寫這三份紀錄（[功能二](./feature-02-hospital-adapter-boundary.md)、[功能三](./feature-03-clinic-roster.md)、[功能四](./feature-04-cascading-setup-ui.md)）。

連帶後果：功能五（送出前擋掉填錯的看診資料）被整個遺漏，不是被合併——它從頭到尾沒被當成一個題目 Discover 過，詳情見[功能四文件的共通觀察](./feature-04-cascading-setup-ui.md#功能-234-共通觀察道館的一功能一輪規則沒有被遵守)。

根因不是三支 skill 內容本身的問題——各支 skill 在單一功能範圍內都有被遵守的證據。根因是 agent 順著 issue 之間的 `Refs` / `Depends on` 連結一路往前推進，卻沒有拿道館表格的功能編號逐項核對範圍邊界。這兩種邊界（issue 拆分邊界、道館觀察邊界）是不同的事，agent 把它們混為一談。

**給下一輪的規則**：每次只交給 agent 道館表格裡的一個功能編號；一個功能合入 main 後先停下來，確認觀察紀錄寫完、表格更新完，才開下一個編號。agent 自己發現某張 issue 能再拆成更小的切片時，先回報「這張 issue 可以拆成 N 個道館功能、對應哪幾個編號」，等人類確認，再動工。

## 本道館最值得記下的一件事

**兩次把方向拉回來的都是人，不是流程。** 功能一的開發過程中，如果沒有人類兩次介入（一次要求去看實際看診狀況、一次提供進度頁截圖），Agent 會交出一個「用列的出現消失判斷監看」的版本 — 在 fixture 上全綠，在真實環境永遠不會結束監看。

Double Diamond 整套方法就是為了避免這件事，而這次是人擋下來的。這也說明 skill 不是保證，它只是讓 Agent 更容易被問對問題。

這也是為什麼上面的「累積的 skill 改進提案」要認真做：**每一條改進都在把這次需要人介入才避開的坑，變成下次 skill 本身就會避開的坑。**
