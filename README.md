# AIBDD skills dojo

## 關於這個 repo

我在這個 repo 整理了**軟體工程方法論**相關的 skills，把 Conventional Commits、Trunk-Based Development、Double Diamond 這三套有官方文獻的方法論，變成 AI agent 可以實際執行的 skill。

目的很單純：讓 AI 在開發時不要只是「照字面做」，而是先確認問題對不對、每次只改一小塊、並把改動的理由留在 commit 訊息裡。

## 結構說明

| 路徑 | 內容 |
|---|---|
| `.claude/skills/` | Claude Code 格式的 skills |
| `.agents/skills/` | Codex／.agents 格式的 skills |
| `VERSION` | template 版本 |

> 以上路徑是《AI x BDD》課程平台抓取與展示 skills 的固定位置，請勿更名、搬移或刪除。

## 道館作業

| 道館名稱 | 繳交版本 | 挑戰內容 |
|---|---|---|
| [道館一：三支 Skills 組合實戰](./docs/dojo-01-three-skills/) | v1（2026-10-29） | 研究 Conventional Commits、Trunk-Based Development、Double Diamond 三個主題並各自做成 skill，再把三支一起用在同一個專案上連續開發 5-8 個功能 —— [開發紀錄與觀察](./docs/dojo-01-three-skills/) |

每一館的細節都收在 `docs/` 底下，點進去可以看到該館的功能開發紀錄、完成條件核對，以及我自己的觀察。

## Skills 與設計想法

三支都走同一個流程產出：先讀官方文獻、沿站內相關頁面延伸、整理成關鍵字／方法論／執行步驟／規則／適用條件與例外，再用 `/skill-creator` 寫成 skill。每支都保留 `references/` 讓人能回到原始出處核對。

### `/double-diamond-principle`（雙鑽石原則）

- **用途**：用 Discover → Define → Develop → Deliver 四階段引導問題探索，強制「先找對問題，再找對解法」。
- **使用時機**：需求看起來很明確、很想直接開工的時候 —— 那正是最該用它的時候。
- **設計想法**：AI 最常見的失敗不是寫錯程式，是**把錯的需求完美實作出來**。這支 skill 的價值在於逼出「這個前提是真的嗎」這個問題。
- **來源**：[Design Council — The Double Diamond](https://www.designcouncil.org.uk/resources/the-double-diamond/)、[Framework for Innovation](https://www.designcouncil.org.uk/resources/framework-for-innovation/)
- **驗證狀態**：已實際使用，見[道館一功能一](./docs/dojo-01-three-skills/feature-01-watch-window.md)。該次它確實推翻了需求的核心前提。

### `/trunk-based-development`（主幹式開發）

- **用途**：決定要不要開分支、分支活多久、什麼時候整合、未完成的功能怎麼安全進 trunk。
- **使用時機**：要進版控的時候；以及「這個改動太大，要不要開長命分支」的念頭浮現的時候。
- **設計想法**：核心是「進行中的程式碼與 trunk 的距離」。距離越大，合併越痛。skill 裡把 feature flag 與 branch by abstraction 寫成明確選項，避免 AI 用長命分支解決問題。
- **來源**：[trunkbaseddevelopment.com](https://trunkbaseddevelopment.com/)
- **驗證狀態**：已實際使用，見[道館一功能一](./docs/dojo-01-three-skills/feature-01-watch-window.md)。

### `/git-conventional-commit`（慣例式提交）

- **用途**：依 Conventional Commits v1.0.0 寫、檢查、修正 commit 訊息，包含 type 選擇與破壞性變更標示。
- **使用時機**：每次 commit 前。
- **設計想法**：規格只明定 `feat` 與 `fix`，其餘是業界慣例，所以 skill 裡附完整 type 對照表與 SemVer 對應，並強調「一次變更同時符合多個 type 就拆開」。
- **來源**：[Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)、[SemVer](https://semver.org/)
- **驗證狀態**：已實際使用，見[道館一功能一](./docs/dojo-01-three-skills/feature-01-watch-window.md)。

## 交流

這些 skills 歡迎直接取用，或依你的專案調整。如果有使用上的問題或改進建議，歡迎開 issue 與我討論。

---

## 官方宣告（以下區塊請保留）

本 repo 為[水球軟體學院](https://world.waterballsa.tw)《AI x BDD：規格驅動全自動化開發術》課程的學員產出，用於公告課程中道館作業的展示，並依[官方 Template](https://github.com/AI-x-BDD/aixbdd-skill-homework-template) 建立。

《AI x BDD：規格驅動全自動化開發術》課程四大特色：

1. **【軟工方法論＋結果導向】**
   只談 AI coding 不談軟工方法論，是不學無術！課程中強調結果導向的開發方法：只要訂好驗收標準，就可以 One-Shot 開發到位！
2. **【最完整的 SDD 概念與實作】**
   市面上最完整的 SDD 課程，一次教你 Skills、SDD、TDD、BDD 的知識與實踐，輕鬆實踐高效可靠全自動開發。
3. **【企業級實戰導入】**
   用企業級實踐導入高規格，不只學會工具，更知道如何舉一反三完全客製化企業專案需求。
4. **【課程教學方式與物超所值】**
   這是一堂理論、實務跟實戰三者兼具的 SDD 線上課程，用工作坊級別的學習體驗，不是只是教概念，而是直接帶著做！

[了解更多課程內容說明](https://waterballs.tw/Ghl6O)

## 開源專案：AI x BDD

水球老師把這套 AI x BDD 的方法論做成開源 pipeline skill：把產品迭代寫成 Gherkin，再一路帶到 RED → GREEN → REFACTOR。

覺得有幫助，歡迎給它一顆 Star：

[![Star Waterball-Software-Academy/aixbdd](https://img.shields.io/github/stars/Waterball-Software-Academy/aixbdd?style=social)](https://github.com/Waterball-Software-Academy/aixbdd)

## 相關問題洽詢管道

1. [水球軟體學院 LINE 官方帳號](https://lin.ee/STIsOLZ)
2. 客服信箱：[support@waterballsa.tw](mailto:support@waterballsa.tw)
3. [水球軟體學院（社群）Discord](https://discord.gg/Ymjz7NmZXn)

---

© 2026 水球球特務有限公司版權所有，侵害必究。
