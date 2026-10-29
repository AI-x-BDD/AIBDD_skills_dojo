---
name: trunk-based-development
description: >-
  以 Trunk-Based Development（主幹式開發）的分支模型指導團隊的 git 分支、整合、發布策略。
  當使用者問要不要開分支、分支該活多久、怎麼做 code review、如何保持 trunk 永遠可發布、
  要不要用 feature flag、如何從 trunk 發布或切 release branch、比較 TBD 與 GitFlow /
  GitHub Flow，或出現 merge hell、長命分支、CI 太慢等問題時，就使用本 skill。即使使用者
  沒講出「trunk-based development」這個詞，只要在討論分支策略與持續整合，也要主動套用。
---

# Trunk-Based Development（主幹式開發）

一種版本控制分支模型：團隊**共用一條主要分支 trunk（git 常稱 `main`）**，避免維護其他長命
開發分支，每個人都頻繁地把小而完整的改動整合回 trunk。核心是最小化「進行中的程式碼」與
「共享整合點 trunk」之間的**距離（distance）**——距離越大，合併越痛、build 越容易壞。

TBD 是 **Continuous Integration 的關鍵促成者**，進而支撐 Continuous Delivery。
本 skill 依官方 trunkbaseddevelopment.com 整理；完整細節與原始出處見 `references/details.md`。

## 要解決的核心問題

- **Merge hell**：長命分支累積數天差異，晚期合併又難又久。
- **壞掉的 build 拖垮團隊**：讓人把時間耗在修 build 而非開發。
- TBD 用「頻繁小整合 + 永遠保持 trunk 可建置」來根除這兩者。

## 兩種模式：先選對

| 模式 | 適用團隊 | 做法要點 |
|------|---------|---------|
| **直接提交 trunk**（committing straight to the trunk） | 小團隊（約 15 人以下、低併發） | 不開分支，本機跑完整 build 通過才 push |
| **短命功能分支**（short-lived feature branches） | 規模化團隊（約 16 人以上、高併發 / 高提交率） | 一人一分支，透過 PR + 優先 review + CI gate 整合 |

> 判斷規模看「併發寫入者數量 + 提交頻率」，不只看人頭。AI agent 時代：每個 agent 的
> worktree ≈ 一條短命分支，同樣規則適用。

## 執行規則

1. **分支要短命**：短命分支約 **2 天內**結束（含等 review 的時間），超過就淪為長命分支。
   一人一分支（pair 可兩人），別人不應在你的分支上寫 production code。
2. **一天多次整合**：頻繁 clone / pull trunk，約每小時整合一次，每次都是不會弄壞 build 的小 commit。
3. **push 前先本機 build**：跑和 CI 同一套 build（compile + unit/service/functional test），
   通過才 push；push 被拒就再同步重試。這是保持 green trunk 的關鍵。
4. **Code review 要被優先處理**：三選一——pair programming、pre-merge review、短命 PR 分支。
   review 拖好幾天就等於浪費了快速 CI（擱置數天的 PR =「換名字的長命分支」）。
5. **保持 green trunk**：trunk 隨時可建置、可發布，絕不 break the build。
   完整 build 要快（理想約 1 分鐘，10 分鐘以上算慢）；慢 / 不穩的測試移到 pipeline 後段。
6. **壞掉就快速 revert**：對事後發現的壞 commit 採 revert 政策（build cop / bot，必要時短暫鎖 trunk），
   而不是 code freeze。

## 讓未完成的功能也能安全進 trunk

大改動不要靠長命分支，改用這兩招，讓每次 push 都能上線：

- **Feature flag / toggle**：以旗標在啟動或執行期控制功能開關，未完成功能先「關閉」狀態進
  production。重點是把旗標控制放在啟動流程「最早可控制的抽象點」，而不是到處散佈 if/else。
  旗標是技術債——發布後約一個月就該爭取時間移除，並在 readme 記錄「review for delete」日期。
- **Branch by abstraction**：大型替換 / 重構時，先在舊程式外包一層抽象，寫新實作（先關閉），
  切換開關啟用，再移除舊實作與抽象。可暫停 / 續做、取消成本低，全程隊友照常工作。

## 發布策略

| 發布頻率 | 策略 |
|---------|------|
| 非常高頻 | **Release from trunk**：不開 release branch，用 commit reference / 日期時間當版本，bug 一律在 trunk 上 fix-forward（向前修） |
| 一天一次或更低 | **Branch for release**：發布前幾天即時從 trunk 切 release branch |

**Branch for release 的鐵則**：

- release branch 切出後**不做持續開發**，也**不 merge 回 trunk**。
- 修補流程：bug 先在 **trunk 修 + 加測試 + CI 驗證**，再 **cherry-pick** 到 release branch。
  **絕不**在 release branch 修完再 merge 回 trunk（會造成日後 regression）。
- cherry-pick 由輪值的 Merge Meister 負責，並記錄核准供稽核。

## 與其他模型的取捨（官方觀點）

- **GitHub Flow**：很接近 PR 導向的 TBD；關鍵差異是它從「分支」切發布，TBD 從 trunk 發布。
- **GitFlow**：官方明言「與 TBD 不相容」。多條長命分支（develop / feature / release / hotfix）
  阻礙連續版本的併發開發；適合有明確發布週期的專案，不適合快速迭代 / 持續交付。
- 分支原則（Wingerd & Seiwald）：「Branch: only when necessary, on incompatible policy,
  late, and instead of freeze.」

## 適用條件與例外

- **適合**：想真正落實 CI、需要快速迭代 / 持續交付的團隊；小團隊到超大型組織皆可（Google 單一
  monorepo trunk，數萬名開發者）。
- **不適合 / 要先補強**：自動化測試覆蓋不足（安全網會失效）、build / CI 太慢（必須先壓到幾分鐘內）、
  需長期並行維護多個對外版本且客戶自選升級時機的產品（此時才需要 release branch）。

## 反模式（You're doing it wrong）

- 有一條叫 trunk 的分支 ≠ 在做 TBD。
- 分支活超過一兩天、漂離 trunk；多人共用同一條 feature 分支。
- 在 release branch 修 bug 再 merge 回 trunk；發布前 code freeze；把未就緒的 commit 退出 trunk
  （應藏在 feature flag 後面）。
- 本機 build 太慢 / 範圍太窄、CI 太慢批次處理——都會拉長「壞掉到發現」的時間，修復越貴。

完整術語、兩種模式細節、CI 做法、feature flag 與 branch by abstraction、發布流程與原始出處：
見 `references/details.md`。
