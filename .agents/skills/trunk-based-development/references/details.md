# Trunk-Based Development - 完整細節與出處

> 依官方 trunkbaseddevelopment.com（Paul Hammant 等維護）整理。摘要版在 `../SKILL.md`。

## 目錄
- [術語](#術語)
- [一句話定義](#一句話定義)
- [兩種模式細節](#兩種模式細節)
- [CI 的角色](#ci-的角色)
- [發布策略細節](#發布策略細節)
- [Feature flag 與 Branch by abstraction](#feature-flag-與-branch-by-abstraction)
- [與其他模型對比](#與其他模型對比)
- [反模式全集](#反模式全集)
- [資料來源](#資料來源)

## 術語

- **trunk / main**：團隊共享的唯一主要分支，所有人從這裡 clone/pull 並整合回去。
- **short-lived feature branch**：從 trunk 切出、存活約 2 天內、經 PR + review 整合回 trunk 的分支（更精確叫 change branch，bug fix 也走同流程）。
- **release branch**：為發布即時切出的分支，不持續開發、不 merge 回 trunk，只接收 cherry-pick。
- **feature flag / toggle**：以旗標在啟動或執行期控制功能開關。分 build flag 與 runtime-switchable flag（執行期可切，需持久化並在叢集同步，可用 Consul / etcd）。
- **branch by abstraction（BbA）**：Stacy Curl 2007 命名。在 trunk 上做大型替換而不開長命分支的技術。
- **committing straight to the trunk**：不開分支 PR，直接 commit/push 到 trunk。
- **scaled trunk-based development**：大型組織單一 monorepo trunk（Google 數萬開發者），靠高速 CI、merge queue 支撐。
- **green trunk**：trunk 隨時保持建置通過、可發布。
- **Merge Meister**：負責把 trunk 修補 cherry-pick 到 release branch 的人，可每日輪替。
- **commit queue / merge queue**：機器人把待整合變更對最新 trunk rebase、build、驗證後才落地。起源 Bors（2013），後有 bors-ng、GitLab merge trains（2019）、GitHub merge queue（2023/07 GA）。

## 一句話定義

開發者共用一條稱為 trunk 的主要分支，避免維護其他長命開發分支，大家頻繁把工作整合進 trunk。核心是最小化「進行中程式碼」與「共享整合點 trunk」的距離。

## 兩種模式細節

### (a) 直接提交到 trunk
- 適用：小團隊（官方給「三、四人」為任何人都能修壞掉 build 的規模；無 AI agent 的現代團隊通常約 16 人以下）。
- 做法：同步最新 trunk → 本機跑完整 build → 通過才 push → commit 小而完整，大工作藏在 feature flag / BbA 後 → push 被拒就重試。
- CI 在 commit 落地後才建置 trunk，是有延遲的安全網；遇紅燈快速 revert。不取代 push 前本機 build。

### (b) 短命功能分支
- 適用：規模化團隊（傳統 16+ 人；約 15 人以下可考慮直接提交 trunk）。
- 一人一分支（pair 可兩人），分享供 review 可以但別人不應在上面寫 production code。
- 約 2 天內結束（含等 review）。可把 trunk merge 進分支保持同步；分支只在收尾刪除前 merge 回 trunk。
- 經 PR 整合，必須優先 review。
- AI agent 時代：以「併發寫入者 + 提交頻率」取代人頭判斷規模；每 agent worktree ≈ 一短命分支；review 成瓶頸，建議 AI 輔助 review + CI 非 AI 自動檢查。

## CI 的角色

- 本意：單一共享整合點 + 開發者一天多次整合 + CI server 驗證品質。
- 本機先跑和 CI 一樣的 build 腳本（compile、test-compile、unit、service、functional test）才提交。
- 較佳做法：commit 進 trunk 前就跑 review + CI；因時序仍可能失敗，push 後再跑一次並以自動 revert 當後備。
- 高吞吐量用 speculative merge（把待驗證 commit 放在接在最後 green commit 後的分支驗證再自動 merge）。
- build 要快：約 1 分鐘佳，10 分鐘以上算慢；聚焦 compile 與純 unit test，減少 thread/socket/file I/O 的 integration test。
- 安全措施：Test Impact Analysis（只跑受影響測試）、自動 bisect、快速 revert。

## 發布策略細節

### Release from trunk
- 適合發布頻率非常高、完全不開 release branch。版本用 commit reference 或日期時間。bug 一律 fix-forward。

### Branch for release
- 發布前幾天即時從 trunk 切（切分支只是輕量 commit，開發者照常全速提交 trunk）。切點可非 HEAD、可事後從 release tag 切。
- 修補：bug 先在 trunk 修 + 測試 + CI → cherry-pick 到 release branch（branch 有複製的 CI pipeline 再驗）。
- **絕不**在 release branch 修再 merge 回 trunk（唯一例外：bug 在 trunk 重現不出來，承認 regression 風險）。
- cherry-pick 由 Merge Meister 負責，記錄核准供稽核（例：Go 專案 backport issue 須 release manager / code owner 核准，預設答案 no）。

### 用 tag 取代 branch
- 從 trunk 上編號 tag（如 v1.1.1）發布，要修補時再事後從 tag 切分支做 point release。git 中刪 release branch 前要先建 tag，否則 commit 可能被 GC。

### cadence 對應
- 非常高頻 → 只能從 trunk 發布；一天一次或更低 → 可切 release branch + cherry-pick。

## Feature flag 與 Branch by abstraction

- **Feature flag**：程式可在關閉狀態 push 到 production，支援 A/B test、beta 群組。把旗標抽象放在啟動流程最早可控制點（範例 `PurchasingCompleting` 抽象，boot 時擇一實作），不是到處 if/else。CI 要測有意義的旗標組合；runtime flag 要持久化。清理：發布後約一個月爭取時間移除；Brad Appleton 警告旗標常無法如預期短命。
- **Branch by abstraction**：讓長時間大型替換（例 Go CI server 從 iBatis 換 Hibernate）在 trunk 上逐步進行。好處：可暫停/續做、取消成本低、避免團隊放棄 TBD。不適用：需長期支援舊 API / 舊版本且客戶自選升級時機（例 KDE 5.0）。

## 與其他模型對比

- **GitHub Flow**：很接近 PR 導向 TBD；差異是它從分支切發布（merge 回 trunk 前），TBD 從 trunk 發布。PR 擱置數天 = 換名字的長命分支。
- **GitFlow**（Vincent Driessen 2010）：官方明言「與 TBD 不相容」。多條長命分支阻礙連續版本併發開發；適合明確發布週期。
- **Mainline**：官方稱「與 TBD 完全相反，別這樣做」。
- **Cascade / Many Trunks**：官方明言不相容 / 多陷阱，偏好「單一 trunk + 模組 + 接收 cherry-pick 的 release branch」。
- 分支原則（Wingerd & Seiwald）：Branch only when necessary, on incompatible policy, late, and instead of freeze.

## 反模式全集

**分支**：有叫 trunk 的分支 ≠ 做 TBD；長命 / 共享分支；分支活超過一兩天漂離 trunk；多人同一 feature 分支；從非 trunk 的分支 pull 別人改動；workstation 上 clone 與別人分支互 merge。

**release branch**：在上面修 bug 再 merge 回 trunk；定期把 trunk merge（而非 cherry-pick）進 release branch；release branch 互相 merge；一條 release branch 跨多大版本長存；結束時整個 merge 回 trunk（沒改東西就直接刪）。

**發布**：code freeze；unmerging（未就緒工作該藏 feature flag 後）；接近發布時改變開發節奏。

**CI**：反覆 push 共享 CI 當自己的 build 測試；本機 build 太慢 / 太窄；CI 太慢 / 單執行緒批次處理（commit 越積修復越貴）；長命分支削弱 CI 效用。

## 資料來源

實際讀取的官方頁面：
- https://trunkbaseddevelopment.com/ （首頁 / one-line summary）
- https://trunkbaseddevelopment.com/5-min-overview/
- https://trunkbaseddevelopment.com/short-lived-feature-branches/
- https://trunkbaseddevelopment.com/committing-straight-to-the-trunk/
- https://trunkbaseddevelopment.com/branch-for-release/
- https://trunkbaseddevelopment.com/feature-flags/
- https://trunkbaseddevelopment.com/branch-by-abstraction/
- https://trunkbaseddevelopment.com/release-from-trunk/
- https://trunkbaseddevelopment.com/continuous-integration/
- https://trunkbaseddevelopment.com/game-changers/ （scaled TBD）
- https://trunkbaseddevelopment.com/alternative-branching-models/ （GitFlow / GitHub Flow / Mainline / Cascade 對比）
- https://trunkbaseddevelopment.com/youre-doing-it-wrong/ （反模式）

> 備註：GitFlow 對比內容在 /alternative-branching-models/ 頁，非獨立 /gitflow/ 頁。第三方比較文（proxify.io、flagsmith.com、mergify.com 等）結論與官方一致，核心整理以官方站為準。
