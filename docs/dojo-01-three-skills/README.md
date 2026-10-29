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
| 2 | — | — | — | 未開始 | — |
| 3 | — | — | — | 未開始 | — |
| 4 | — | — | — | 未開始 | — |
| 5 | — | — | — | 未開始 | — |

## 完成條件核對

| 條件 | 狀態 |
|---|---|
| 三支 Skill 的完整資料夾 | 🚧 檔案已就位（[`.claude/skills/`](../../.claude/skills/)、[`.agents/skills/`](../../.agents/skills/)），但**尚未 commit 進 git**，推上去才算繳交 |
| 同一專案完成 5-8 個功能的 git commits 紀錄 | 🚧 1 / 5 |
| 人類觀察：哪些原則被遵循、哪些沒有、優化空間 | 🚧 隨各功能紀錄累積，見[累積的 skill 改進提案](#累積的-skill-改進提案) |
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

## 本道館最值得記下的一件事

**兩次把方向拉回來的都是人，不是流程。** 功能一的開發過程中，如果沒有人類兩次介入（一次要求去看實際看診狀況、一次提供進度頁截圖），Agent 會交出一個「用列的出現消失判斷監看」的版本 — 在 fixture 上全綠，在真實環境永遠不會結束監看。

Double Diamond 整套方法就是為了避免這件事，而這次是人擋下來的。這也說明 skill 不是保證，它只是讓 Agent 更容易被問對問題。

這也是為什麼上面的「累積的 skill 改進提案」要認真做：**每一條改進都在把這次需要人介入才避開的坑，變成下次 skill 本身就會避開的坑。**
