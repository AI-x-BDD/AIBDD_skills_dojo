# 功能二：讓系統不再只懂台大一家，把讀取方式獨立成可替換模組

| 項目 | 內容 |
|---|---|
| 開發日期 | 2026-10-29 |
| 實作專案 | [next-is-you](https://github.com/KLTsai/next-is-you) |
| 需求來源 | [Issue #45](https://github.com/KLTsai/next-is-you/issues/45)（本輪只做 Define 階段切出來的 (A) 半邊） |
| 產出 | [PR #48](https://github.com/KLTsai/next-is-you/pull/48) ✅ 已 squash merge 至 main：[`5a96d4f`](https://github.com/KLTsai/next-is-you/commit/5a96d4f)<br>原始 commit（分支已刪，PR 內可查）：[`7fb478f`](https://github.com/KLTsai/next-is-you/commit/7fb478f) |
| 分支 | `feat/45-hospital-adapter-boundary`（短命分支，同日開分支、commit、開 PR、合入 main） |
| 測試結果 | 259 passed（含新增 `tests/test_hospital_registry.py` 4 案例）、ruff format/check、tsc、test:ui 13 passed、vite build 全通過 |
| 三支 skill 怎麼被使用 | [見道館總覽](./README.md#三支-skill-如何在本道館被使用)（本道館所有功能共用） |

## 這次做了什麼

next-is-you 原本只懂台大總院一家。「怎麼跟台大的網頁要資料、怎麼解析它的卡片、用哪種語言」這些專屬知識散在讀取器各處，呼叫端（排程器、送出閘門）甚至要用 `isinstance(reader, NtuhProgressReader)` 判斷手上拿的是哪一家的讀取器。

這次把這些知識收進一個可替換的模組，系統其餘部分只跟一個固定介面講話：

- `HospitalReader` 介面搬進 `domain.py`，補上 `observe_for_start`，讓 Start 流程不再用 `isinstance` 分支。
- 新增 `hospital_registry.py`：`Hospital -> (SourceCapability, reader factory)` 的對照表，`RegistryHospitalReader` 依 `scope.hospital` 分派，查不到讀取器就 fail-closed 回「不可用」（不是噴錯，也不是假裝有資料）。
- 新增 `SourceCapability`（`full_schedule` / `current_number_only` / `unsupported`），作為設定 UI 之後要讀的降級依據。
- `hospital_adapters.py` 改名 `source_samples.py`。它其實是離線驗證來源用的解析樣本，不是真正讀取器會用到的那層，原名會讓人把新醫院接錯地方。
- 新增 `docs/ADDING-A-HOSPITAL.md`：不看原始碼就能判斷一家醫院能不能接、要做什麼。

## 原本預期做什麼？為什麼實際做出來的不一樣？

[Issue #45](https://github.com/KLTsai/next-is-you/issues/45) 原始需求單把兩件事綁在同一組驗收條件裡：(A) 可替換的讀取架構、(B) 串接式設定 UI。

Define 階段發現這兩件事可以獨立交付，而且相依是單向的 - UI 需要架構，架構不需要 UI。(A) 可以「branch by abstraction」的方式進 trunk，對外行為完全不變，而且能立刻解除 #44（新光評估）的阻塞；(B) 是一塊大得多的前端工作。

所以需求單被重新框定：(A) 留在 #45 本輪交付，(B) 另開 [#47](https://github.com/KLTsai/next-is-you/issues/47)。這是 Double Diamond Define 階段該做的事 - 從議題本身出發重新框定挑戰，而不是照單全收一張綁了兩件事的需求單。拆分理由記錄在 [#45 的 scope split 留言](https://github.com/KLTsai/next-is-you/issues/45#issuecomment-6496872746)。

## Double Diamond 四階段實際發生的事

### Discover

這一輪**沒有**重新做田野調查，這跟功能一很不一樣，值得記下理由：

1. Issue #45 的 body 本身已經包含成熟的需求描述 - 降級規則（完整資料／只有叫號／兩者皆無三種情境）、驗收條件、排除項都寫好了。
2. 它要消費的架構是功能一（#34 / PR #46）剛驗證過的結果：`parse_light_table()` 已經會回傳完整的科別＋時段名冊、監看邊界已經改由叫號號碼決定。

所以 Discover 的動作是**讀既有程式碼與既有決策**：盤點「台大專屬知識」實際散落在哪幾個檔案、哪些呼叫端在對具體類別做判斷。這是功能一提出的「第一站是專案自身的證據目錄」那條建議的延伸 - 這次先翻自己家的東西，沒有再去打擾來源站。

### Define

> 系統其餘部分只跟一個固定介面講話；介面要完整到沒有任何呼叫端需要知道對面是哪一家醫院。讀取器查不到就 fail-closed 回「不可用」，不是噴錯、不是回部分資料。新醫院 = 新模組 + 一筆登記，不動呼叫端。

同時定下本輪**不做**什麼：不做 UI（移交 #47）、不加真實第二家醫院（等 #44 的來源實證）、不動送出閘門／同意流程／Stop 與截止邏輯。

### Develop

對話紀錄裡有明確記下的設計決定只有一項：**查不到讀取器時，沿用既有的 `observation_unavailable` 契約**（`ReadFailed("unavailable", "no_observation")` 恰好對應到它），而不是新增一種錯誤型別，所以 fail-closed 的既有行為原封不動。

`RegistryHospitalReader` 另外會依醫院快取已建立的讀取器，由測試覆蓋；對話紀錄裡沒有留下「還評估過哪些替代做法」的討論，這裡不另外推測。

### Deliver

- `tests/test_hospital_registry.py` 覆蓋四件事：capability 查詢回報登記的等級、dispatch 只建一次讀取器並重複使用、unsupported 醫院 fail-closed、`current_number_only` 等級仍能讀但沒有名冊。
- 最後一項是驗收條件明確要求的「用一家刻意缺少排程能力的合成醫院證明降級不會壞掉」，用 `monkeypatch` 換掉整個 registry 來做，不碰真實登記。
- 259 個測試 + ruff + tsc + test:ui + vite build 全通過才 push。
- 既有台大提醒行為不變，沒有資料遷移、沒有破壞相容性的變更。

## 人類介入

**一、範圍怎麼切，是人決定的。** agent 查完之後沒有自己挑一刀就開工，而是把「這張需求單綁了兩件事，可以這樣拆」連同選項交給人類。人類選了 (A) 後端邊界＋文件，並額外同意把 `hospital_adapters.py` 一起改名（agent 原本列為可做可不做的附帶項）。這一步符合 Double Diamond 的分工：agent 負責把議題攤開，範圍決定權在人。

**二、這一輪沒有在 PR 開出時結束 - 是 agent 先把下一個功能擺上桌的。** PR #48 開出後，agent 在回報裡寫「自然的下一步是 #47 的串接 UI」，並直接問：「你想等 CI 綠就先停，還是 CI 跑的同時我就先去規劃 #47？」人類選了後者（「CI 跑的同時先去規劃 #47」）。CI 綠燈後，agent 又提出三個起點選項（先做 47a 後端切片／整個 #47 一次做／先不動手），人類選了 47a；agent 接著把 47a 疊在尚未合入的 #48 分支上繼續做，直到人類讓 #48 合入。

所以這裡不是「人類要求、agent 沒擋」這麼單純：**延續到下一個功能的選項是 agent 自己提出的**，人類是在 agent 給的選項裡選。而這個 session 的開場 prompt 只說「第二個功能（#45 issue）」，沒有寫「做完就停、下一個功能開新 session」，agent 也沒有讀過道館的功能表，不知道 #45 在道館裡被拆成功能 2～5。這個缺口屬於道館流程，不屬於三支 skill，見 [README 的流程觀察](./README.md#道館流程本身的觀察一功能一-session)。

## 最終的 commit 紀錄

**審查時的 commit**（PR 分支上一則）：

- [`7fb478f`](https://github.com/KLTsai/next-is-you/commit/7fb478f) `feat(next_is_you): isolate hospital reading behind a replaceable adapter`

**合入 main 的 squash commit**：

- [`5a96d4f`](https://github.com/KLTsai/next-is-you/commit/5a96d4f) `feat(next_is_you): isolate hospital reading behind a replaceable adapter (#45)`

type 選擇的理由：

- 選 `feat`：使用者（未來要接的醫院）得到一個新能力 - 可替換的讀取介面。雖然對既有台大使用者來說行為完全沒變，但「新增能力」比「修缺陷」更貼近這次變更的性質。
- 沒有標 `!`、沒有 `BREAKING CHANGE:` footer：介面搬家與改名都在內部，對外行為與資料格式都沒變，既有提醒不需要遷移。
- footer 用 `Refs: #45` 而不是 `Closes`：本輪只交付需求單的 (A) 半邊，(B) 還在 #47，需求單不能在這則 commit 關掉。

**一個差點犯的錯，以及它怎麼被抓到**：工作目錄裡有一則 session 開始前就存在、與本功能無關的 `.gitignore` 未提交改動（新增忽略 `.claude/` 與 `.agents/`）。第一次提交時用了 `git add -A`，它被一起掃進了功能 commit；`git add -A` 與 commit 在同一個指令裡執行，所以是 commit 完成之後才在輸出裡看到。agent 隨即用 `git reset --soft HEAD~1` 加 `git restore --staged .gitignore` 把它拆出來，重新提交成乾淨的 `7fb478f`，`.gitignore` 維持原本未提交的狀態。

`git add -A` 的問題不是它會失敗，是它會成功 - 不相關的檔案悄悄進了 commit。這個例子直接對應下面「skill 可改進點」的那一條。

**關於這則 commit 的顆粒度**：它把「移介面」「建 registry」「加 capability 描述」「改檔名」「寫文件」五件事放在同一則。按[功能一提出的判準](./feature-01-watch-window.md#4-git-conventional-commit--相同-type-但主題不同的多件事也要拆要寫得更明確)（description 需要兩個子句才講得清楚就該拆），這五件事圍繞同一個主題 - 建出介面邊界 - 一句話講得清楚，所以不拆是合理的。但它確實是這三輪裡最大的一顆 commit（+393 / -23，11 個檔案），如果之後要回溯「改名」這件事，會跟介面重構綁在一起，不好單獨 revert。

### 為什麼合入時用 squash

這個 repo 的 main 歷史全是 squash merge（每個 commit 只有一個 parent、尾綴 `(#NN)`）。跟著 house style 走，與 TBD「linear history + 短命分支」的建議一致。本輪 PR 只有一則 commit，squash 與否差別只在標題尾綴的 PR 編號。

## Trunk-Based 執行核對

| 規則 | 做到了嗎 | 證據 |
|---|---|---|
| 短命分支 | ✅ | 同日開分支、commit、push、開 PR、合入（分支存活數小時） |
| push 前跑完整本機 build | ✅ | 259 測試 + ruff format + ruff check + tsc + test:ui + vite build |
| 每個 commit 都不弄壞 build | ✅ | 單一 commit，獨立通過全套 gate |
| 保持 trunk 可發布 | ✅ | CI 綠燈才請求 merge |
| 未完成功能安全進 trunk | ✅ | 用 branch by abstraction：新介面與既有行為並存，對外行為零變化，所以不需要 feature flag |
| 整合前先同步 trunk | ⚠️ 沒有留下 fetch 的證據 | 分支存活極短（同日），事後確認 main 沒動；但同[功能一的建議](./feature-01-watch-window.md#5-trunk-based-development--開分支那一刻需要明確要求-fetch-main)，這是運氣不是流程 |
| 不相關變更拆開提交 | ⚠️ | `git add -A` 把 session 前就存在的 `.gitignore` 改動掃進功能 commit，commit 完才發現並拆開（見上） |
| 道館協定：一功能一輪（不是 skill 的規則） | ❌ | PR 還沒合入就開始規劃 #47，見「人類介入」第二點與 [README 的流程觀察](./README.md#道館流程本身的觀察一功能一-session) |

## 這次暴露的 skill 可改進點

### 1. `/git-conventional-commit` - 自我檢查要加一條「commit 前核對 staged 檔案清單」

**從哪看出**：`git add -A` 把一則與本功能無關的 `.gitignore` 改動掃進功能 commit，是 commit 完成後才看到的（見上面「一個差點犯的錯」）。如果沒有注意到，就會生出一則 description 講功能、內容卻夾帶雜務的 commit。

**skill 現況**：「撰寫步驟」第 7 條是自我檢查，但檢查項目全部是**訊息格式**（冒號後有空白？description 非空？breaking 有標？type 拼對？），沒有一條檢查**內容範圍**。

**建議新增規則（放在寫訊息之前）**：

> 在寫 commit 訊息之前，先核對將要提交的檔案清單（`git status`、`git diff --cached --stat`），確認每個檔案都屬於這則 commit 的主題。用 `git add -A` 或 `git add .` 時尤其要核對，因為它們會把工作目錄裡所有改動一起帶走，包含你這次沒有要處理的待辦。發現不屬於本則主題的檔案，先移出 staging 區，再提交。注意：不要把 `git add -A` 和 `git commit` 串在同一個指令裡，否則核對沒有機會發生。

### 2. `/double-diamond-principle` - Deliver 沒有「交付完就停」的規則，agent 自己會提議下一個功能

**從哪看出**：PR #48 開出後，agent 在回報裡寫「自然的下一步是 #47 的串接 UI」，並主動問「要等 CI 綠就先停，還是 CI 跑的同時先去規劃 #47」。功能三的回報結尾也寫「你要我接著規劃/開做 47b」。作業要求的是「每一次只開發一個功能」，但整個 session 裡沒有任何機制讓 agent 在交付後停下來。

**skill 現況**：三支 skill 的 `SKILL.md` 都沒有任何關於「交付後停止」或「下一輪由誰決定」的文字。Deliver 階段的產出是「經驗證的最終解法」，沒有說這一輪到此為止；Define 階段把需求單切成兩半，也沒有為被切出去的那一半定下何時、由誰重新進場。

**建議新增規則（加在 Deliver 結尾）**：

> 交付並回報後停下來。下一個範圍由人指派；不要主動提議或開始下一輪的 Discover，包括「等 CI 的空檔先規劃下一個」。Define 階段切出去的部分，只在紀錄裡寫明它是獨立的下一輪，不在本輪處理。

注意這條規則和「道館作業要求一次一個功能」不是同一件事：後者是作業本身的要求，前者是 skill 能不能幫 agent 守住這個要求。本次的觀察是 skill 沒有幫上忙。

## 衍生產出

agent 在過程中同時產出了程式碼以外的東西：

- **新增 `docs/ADDING-A-HOSPITAL.md`**：capability 三個等級的定義、判斷一家醫院可行性要問的問題、接一家新醫院的步驟、每個讀取器都必須守的不變量（fail-closed、來源不可信、最小保存）。目標是讓人不讀原始碼就能判斷一家醫院能不能接。
- **重新框定 [Issue #45](https://github.com/KLTsai/next-is-you/issues/45)**：補上 scope split 留言，說明 (A)(B) 怎麼拆、為什麼 (A) 可以獨立進 trunk、(A) 具體交付什麼，並記錄「Blocked by #34」這個阻塞已經隨 #46 解除。
- **開出 [Issue #47](https://github.com/KLTsai/next-is-you/issues/47)**：把 (B) 寫成一張獨立需求單，帶完整的需求、驗收條件、相依與排除項。
