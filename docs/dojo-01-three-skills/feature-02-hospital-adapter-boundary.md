# 功能二：讓系統不再只懂台大一家，把讀取方式獨立成可替換模組

| 項目 | 內容 |
|---|---|
| 開發日期 | 2026-10-29 |
| 實作專案 | [next-is-you](https://github.com/KLTsai/next-is-you) |
| 需求來源 | [Issue #45](https://github.com/KLTsai/next-is-you/issues/45)（本次只做 scope 切出來的 (A) 半邊） |
| 產出 | [PR #48](https://github.com/KLTsai/next-is-you/pull/48) ✅ squash merge 至 main：[`5a96d4f`](https://github.com/KLTsai/next-is-you/commit/5a96d4f) |
| 分支 | `feat/45-hospital-adapter-boundary`（短命分支，同日開分支、commit、開 PR、合入 main） |
| 測試結果 | 259 passed（含新增 `tests/test_hospital_registry.py` 4 案例）、ruff format/check、tsc、test:ui 13 passed、vite build 全通過 |
| 三支 skill 怎麼被使用 | [見道館總覽](./README.md#三支-skill-如何在本道館被使用) |
| ⚠️ 本功能的紀律問題 | 這一輪的 Deliver 階段（PR #48 開出、CI 還在跑）**還沒結束，就被要求去規劃下一個功能**，見下方「這次暴露的問題」與[功能四的共通觀察](./feature-04-cascading-setup-ui.md#功能-234-共通觀察道館的一功能一輪規則沒有被遵守) |

## 這次做了什麼

Issue #45 原本把兩件事綁在一張需求單裡：(A) 後端要有一個固定的 `HospitalReader` 介面，讓台大總院的讀取細節全部收進一個模組，其餘程式只認介面；(B) 設定表單要改成串接式選單。這次只做 (A)：

- `HospitalReader` 介面搬進 `domain.py`，補上 `observe_for_start`，讓 Start 流程不再用 `isinstance(reader, NtuhProgressReader)` 判斷要用哪個讀取器。
- 新增 `hospital_registry.py`：`Hospital -> (SourceCapability, reader factory)` 的對照表，`RegistryHospitalReader` 依 `scope.hospital` 分派，查不到讀取器就 fail-closed 回「unavailable」（不是噴錯，也不是假裝有資料）。
- 新增 `SourceCapability`（`full_schedule` / `current_number_only` / `unsupported`），作為之後設定 UI 要讀的降級依據。
- `hospital_adapters.py` 改名 `source_samples.py`，因為它其實是離線驗證用的解析樣本，不是真正讀取器會用到的那層，原名容易讓人接錯地方。
- 新增 `docs/ADDING-A-HOSPITAL.md`：不看原始碼就能判斷一家醫院能不能接、要做什麼。

## 原本預期做什麼？為什麼實際做出來的不一樣？

Issue #45 原始需求單寫的是「串接式設定 UI + 可替換讀取架構」兩件事一起驗收。Define 階段發現這兩件事可以獨立交付、依賴關係是單向的（UI 需要架構，架構不需要 UI），所以把需求單拆成 (A) 架構／(B) UI，(A) 留在 #45，(B) 另開 [#47](https://github.com/KLTsai/next-is-you/issues/47)。這個拆分本身是 Double Diamond Define 階段該做的事 —— 從議題出發重新框定，而不是照單全收。

拆分決定記錄在 [#45 的 scope split 留言](https://github.com/KLTsai/next-is-you/issues/45#issuecomment-6496872746)。

## Double Diamond 四階段實際發生的事

### Discover

這一輪沒有重新做田野調查。Issue #45 的 body 本身已經包含成熟的需求描述（降級規則、驗收條件、排除項），前提架構（`parse_light_table()` 回傳完整診別+時段名冊、用號碼判斷監看邊界）是功能一（#34/#46）已經驗證過的結果。Discover 做的事是**讀既有程式碼與既有決策**，不是向外部來源重新取證。

### Define

把「兩件事綁一張需求單」收斂成「先出 (A)，(B) 另開票」，問題陳述定案為：

> 系統其餘部分只跟一個固定介面講話；讀取器查不到就 fail-closed；新醫院 = 新模組 + 一筆登記，不用動呼叫端。

### Develop

評估過的做法：要不要讓 registry 在啟動時就把所有讀取器建好，還是延遲建立、用快取重複使用？選了延遲建立 + 快取（`RegistryHospitalReader` 內部快取已建立的 reader 實例），理由是未來醫院數變多時不用每個都在啟動時付建置成本。

### Deliver

- `tests/test_hospital_registry.py` 覆蓋：capability 查詢、dispatch 快取、unsupported fail-closed、current-number-only 仍能讀但沒有名冊。
- 259 個測試 + 全部 gate 通過才 push。
- 既有台大提醒行為不變，沒有遷移、沒有破壞性變更。

## 最終的 commit 紀錄

單一 commit（PR 內未拆）：

- [`7fb478f`](https://github.com/KLTsai/next-is-you/commit/7fb478f) `feat(next_is_you): isolate hospital reading behind a replaceable adapter`

**合入 main 的 squash commit**：[`5a96d4f`](https://github.com/KLTsai/next-is-you/commit/5a96d4f) `feat(next_is_you): isolate hospital reading behind a replaceable adapter (#45)`

type 選擇的理由：`feat`，因為使用者（未來要接的醫院）得到一個新能力（可替換介面），不是修缺陷。commit body 用 `Refs: #45`，沒有破壞相容性，所以沒有 `!` 標示。這則 commit 把「移介面」「建 registry」「加 capability」「改檔名」「寫 docs」五件事放進同一則——按 [功能一發現的建議](./feature-01-watch-window.md#4-git-conventional-commit--相同-type-但主題不同的多件事也要拆要寫得更明確)，這五件事其實圍繞同一個主題（建出介面邊界），判準「description 要不要兩個子句才講得清楚」下，一句話講得清楚，沒有違反該建議；但確實是這次範圍裡最大的一顆 commit。

## Trunk-Based 執行核對

| 規則 | 做到了嗎 | 證據 |
|---|---|---|
| 短命分支 | ✅ | 同日開分支、commit、push、開 PR、合入 |
| push 前跑完整本機 build | ✅ | 259 測試 + ruff + tsc + test:ui + build |
| 保持 trunk 可發布 | ✅ | CI 綠燈才請求 merge |
| 整合前先同步 trunk | ⚠️ 未特別記錄是否 fetch 過 main | 分支存活時間極短（同日），風險低，但沒有留下 fetch 的證據 |
| 未完成功能安全進 trunk（branch by abstraction） | ✅ | 新介面與既有行為並存，沒有 feature flag 也不需要——因為對外行為完全沒變 |

## 這次暴露的問題

技術交付本身（介面、registry、fail-closed）是乾淨的，Double Diamond 在「拆分需求單」這一步也用對了。**但這一輪跟下一輪之間沒有留白。** PR #48 的 CI 還在跑的時候，就被要求「CI 跑的同時先去規劃 #47」——也就是說，功能二的 Deliver 階段還沒收尾，功能三/四的 Discover 就已經開始。詳細分析見[功能四文件的共通觀察](./feature-04-cascading-setup-ui.md#功能-234-共通觀察道館的一功能一輪規則沒有被遵守)。
