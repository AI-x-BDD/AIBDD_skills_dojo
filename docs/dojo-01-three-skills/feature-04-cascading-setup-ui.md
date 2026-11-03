# 功能四：報到設定改成用選的，只剩看診號碼要手填

| 項目 | 內容 |
|---|---|
| 開發日期 | 2026-10-31 |
| 實作專案 | [next-is-you](https://github.com/KLTsai/next-is-you) |
| 需求來源 | [Issue #47](https://github.com/KLTsai/next-is-you/issues/47)（前端切片，依賴功能三的名冊後端） |
| 產出 | [PR #54](https://github.com/KLTsai/next-is-you/pull/54) ✅ 已 squash merge 至 main：[`73260e2`](https://github.com/KLTsai/next-is-you/commit/73260e2)<br>原始四則 commit（分支已刪，PR 內可查）：[`03164c2`](https://github.com/KLTsai/next-is-you/commit/03164c2) [`00b23fa`](https://github.com/KLTsai/next-is-you/commit/00b23fa) [`cbe60df`](https://github.com/KLTsai/next-is-you/commit/cbe60df) [`a2dbff6`](https://github.com/KLTsai/next-is-you/commit/a2dbff6) |
| 分支 | `feat/47b-cascading-setup-ui`（短命分支，同日開分支、commit、開 PR、合入 main） |
| 測試結果 | 269 passed（含重寫的瀏覽器驗收旅程與 rehearsal UI E2E）、ruff format/check、tsc、test:ui 13 passed、vite build 全通過 |
| 三支 skill 怎麼被使用 | [見道館總覽](./README.md#三支-skill-如何在本道館被使用)（本道館所有功能共用） |

## 這次做了什麼

把設定表單從「每一格都手打」換成「串接式選取」：

- 醫院清單、顯示名稱、支援科別、capability 全部來自後端型錄（`GET /api/organizer/hospitals`）。前端寫死的 `hospitalNames`、`ntuh-main` 預設值、台大專屬的說明文字全部移除。
- 流程變成：醫院 → 科別＋日期 → （自動抓名冊，附載入狀態）→ 醫師／診間下拉（時段自動帶出）→ 只填看診號碼。
- 五種狀態都有明確畫面：完整名冊、載入中、降級手動輸入、醫院尚未支援、送出前驗證。
- 名冊不可用時（#22 試點前讀取被 held、約診日不是今天、讀取失敗）降級為手動輸入，並附來源進度頁的參考連結與即時解析預覽。
- 送出前驗證：日期不能選過去；名冊模式下選到的醫師／診間必須真的在當天名冊裡。

## 原本預期做什麼？為什麼實際做出來的不一樣？

這一輪動工前，先用可互動原型（Lavish）跑一次 UX 審閱，把六項決策鎖定後才寫程式 - 見下面「人類介入」第二、三點（決策內容記錄在 [#47 的實作規格留言](https://github.com/KLTsai/next-is-you/issues/47#issuecomment-6538923721)）。

其中第五項決策（手動輸入怎麼輔助）是這一輪最大的轉折：原型一開始只在「單一欄加格式提示」與「拆成三欄」之間讓人選，人類的回饋把問題改寫成「使用者怎麼知道該輸入什麼」。

## Double Diamond 四階段實際發生的事

### Discover

沒有再對來源站取證（名冊能力在功能三已經驗證）。這一輪 Discover 發生在原型審閱裡，**問題多半是人類提的**：

- 為什麼科別只有眼科？（答：讀取器只對眼科有來源實證，其他科別需要各自的證據與讀取器）
- 「選診間」在語意上到底是在選什麼？（答：診間＋醫師＋門診別的組合）
- 比對是精確還是關鍵字？（答：逐欄精確，大小寫／空白／標點不敏感）

這些答案被寫進原型後，人類在手動輸入那一題（Q5）給了最重要的一則回饋，見「人類介入」第三點。

### Define

> 送出前要能分辨兩種模式：**名冊模式**（使用者只選，不打字）與**手動模式**（名冊不可用，使用者打字但要有辦法知道該打什麼）。兩種模式的驗證邊界不同，而手動模式輸入值的格式正確性**不是前端的責任** - 格式是醫院專屬知識，前端不得帶。

### Develop

六項決策在寫程式前定案：版面不重新設計（只做漸進揭露）、醫院清單全列出但不支援的灰掉標註「尚未支援」、約診日不是今天就轉手動輸入並說明原因、選完科別＋日期自動抓名冊不用按鈕、手動輸入採「單一欄＋參考連結＋解析預覽」、科別只列後端宣告支援的。

每項決策的選項與理由以 [#47 的規格留言](https://github.com/KLTsai/next-is-you/issues/47#issuecomment-6538923721) 為準，這裡不重述。

### Deliver

除了測試全綠，這一輪在 Deliver 階段**自己抓到三個問題**，值得分開記，因為它們證明 E2E 真的在瀏覽器裡跑過，不是只跑單元測試：

1. **第一版把台大的三段格式硬寫進前端驗證** - 直接違反本功能自己的需求（前端不得帶醫院知識），而且會擋掉既有已儲存的資料。移除前端格式攔阻，欄位標籤改由後端型錄提供，解析預覽只做提示不做攔阻。
2. **每次按鍵都重新 render 整個表單**，導致手動輸入欄位 blur 時 Save 按鈕被重建，點擊沒有送出。改成只有「會改變後續選項」的欄位（醫院、科別、日期、名冊選項）才重新 render，手動輸入欄只更新預覽節點。這個 bug 在單元測試裡看不到，是在瀏覽器裡點才發現的。
3. **CI 專屬的偶發失敗**：儲存成功訊息會先出現，但列表還在重新載入，一次性斷言跟卡片更新搶跑；本機跑不出來。用 CPU 節流 6 倍重現，改用 Playwright 會自動重試的 `expect()` 取代一次性斷言，修好後在節流下連續驗證 3/3 才移除節流。

## 人類介入

**一、先確認範圍不含視覺重設計。** 人類問「若要進 47b 是有要連 UI 重新設計嗎？」agent 的回答是：需要的是 UX 工作（狀態與流程），不是視覺改版或新設計系統。這個確認讓決策 1 成立，避免範圍滑向改版。

**二、原型審閱是人提議的。** 人類提議：「既然 47b 要換成串接式選取＋狀態處理，那 web app 裡的 layout 跟 UX 不就是要一起討論嗎？或許可使用 lavish 來輔助規劃 UI/UX?」（他在同一段時間內先後提了兩次，理由之一是用文字描述容易有偏差）。這把 Develop 從「用文字討論」換成「先做可互動原型、逐條鎖決策」。六項決策因此在動工前定案，而不是在 code review 時才被挑戰。

**三、最關鍵的一次：人類指出降級路徑的體驗很差。**

原型的 Q5 題目是這樣問的：手動輸入時，後端是逐欄精確比對（診間、醫師、門診別三欄都要對），所以要選「維持單一『診間代碼』欄加格式提示」，還是「拆成三個獨立欄位」。這兩個選項都預設使用者知道該打什麼。

人類沒有在這兩個選項裡選，而是回饋了真正的問題，原話是：「我怎知道要怎麼輸入希望能透過連結可以連到來源網頁參考，輸入什麼內容，通常在意的點就是輸入的內容對應不到，要一次不差才能，這樣操作體驗很差」。

這一句把題目從「欄位怎麼排」改寫成「使用者怎麼知道該輸入什麼」。agent 的回應是：加一個指向院方即時進度頁的參考連結（app 本來就有這個來源連結，搬到手動輸入處）、加一個即時解析預覽（使用者一邊打，畫面一邊顯示系統讀到的診間／醫師／門診別三欄）、並誠實面對「進度頁只顯示當天，所以未來預約對不到」。最後的決策是「單一欄＋參考連結＋解析預覽」。

這跟功能一的轉折是同一個性質：agent 提出的選項在邏輯上自洽，但站在使用者的處境上漏掉了更根本的問題，是人把它拉回來的。

**四、中途暫停與續做。** 人類要求做到某個步驟先暫停（要關機）。agent 先把進度 commit 成 checkpoint 並寫下續做步驟，幾個小時後（同一天）以「繼續 47b」接回。分支從開出到合入不到一天。

**五、合入由人核准**（「ok merge」）。

## 最終的 commit 紀錄

**審查時的四則 commit**：

- [`03164c2`](https://github.com/KLTsai/next-is-you/commit/03164c2) `chore: ignore the .lavish agent review directory`
- [`00b23fa`](https://github.com/KLTsai/next-is-you/commit/00b23fa) `feat(next_is_you): add supported departments to the hospital catalog`
- [`cbe60df`](https://github.com/KLTsai/next-is-you/commit/cbe60df) `feat(frontend): catalog-driven cascading reminder-setup form`
- [`a2dbff6`](https://github.com/KLTsai/next-is-you/commit/a2dbff6) `test: wait for the list reload before asserting the saved session`

**合入 main 的 squash commit**：

- [`73260e2`](https://github.com/KLTsai/next-is-you/commit/73260e2) `feat(frontend): catalog-driven cascading reminder-setup UI (#47) (#54)`

type 與拆分的理由：

- `chore` 給 `.lavish` 的忽略規則：原型審閱的產物不進 repo，純雜務。
- **兩則 `feat` 刻意拆開**：`00b23fa` 是後端型錄多帶科別清單（scope `next_is_you`），`cbe60df` 是前端串接式表單（scope `frontend`）。兩則 type 相同，但主題與 scope 都不同 - 正是[功能一提出的那條建議](./feature-01-watch-window.md#4-git-conventional-commit--相同-type-但主題不同的多件事也要拆要寫得更明確)要處理的情況，這次照建議拆了。
- `test` 給那則 CI 偶發失敗的修復：它跟本功能的需求無關（是既有測試對非同步時序的假設過窄，被這次多出的型錄請求放大），獨立成則，之後回溯時才分得出「這是功能、那是測試穩定性」。
- squash commit 選 `feat(frontend)`：使用者可見的淨變化是前端的新體驗。
- footer 用 `Refs: #47`（PR 本體以 `Closes #47` 收尾需求單）。

### 為什麼合入時用 squash

同前兩輪，跟著 repo 的 house style（main 全是 squash、尾綴 `(#NN)`）。這一輪四則 commit 的拆解比前兩輪更有資訊量（功能／雜務／測試三類分明），但那份拆解保留在 PR #54 裡仍然查得到，trunk 上維持 linear history。

## Trunk-Based 執行核對

| 規則 | 做到了嗎 | 證據 |
|---|---|---|
| 短命分支 | ✅ | 同一天內開分支、暫停、續做、合入（中途暫停有 checkpoint commit），合入後刪除 |
| push 前跑完整本機 build | ✅ | 269 測試 + ruff + tsc + test:ui + vite build |
| 每個 commit 都不弄壞 build | ✅ | 四則各自獨立通過 |
| 保持 trunk 可發布 | ✅ | CI 綠燈才請求 merge |
| 壞掉就快速處理 | ✅ | CI 偶發失敗沒有用重跑帶過：節流重現 → 修根因 → 連續驗證 3/3 |
| 不相關變更拆開提交 | ✅ | 雜務／後端／前端／測試四則分開 |
| 整合前先同步 trunk | 未查證 | 本輪沒有特別核對 fetch 的紀錄 |
| 道館協定：一功能一輪（不是 skill 的規則） | ❌ | 本輪接在功能三之後，同一個 session 連續做完，見下面的共同觀察 |

## 這次暴露的 skill 可改進點

### 1. `/double-diamond-principle` - 不要把系統的嚴格要求直接轉嫁給使用者

**從哪看出**：後端比對需要精確字串，原型的 Q5 把題目設成「單一欄加格式提示，還是拆三欄」，兩個選項都預設使用者知道該打什麼。人類的回饋（「我怎知道要怎麼輸入…要一次不差才能」）指出了漏掉的那一問：使用者有沒有辦法知道正確答案？

**skill 現況**：Define 階段要求從議題出發重新框定，「Put people first」是原則之一，但沒有具體到「輸入的正確值由外部來源決定時要怎麼辦」。

**建議新增規則（可併入「Put people first」的落實方式）**：

> 當某個輸入的正確值由外部來源決定時，不要只做驗證。要同時提供使用者**取得正確值**的路徑 - 參考來源連結、即時解析預覽，或乾脆改成從清單選取。擋住錯誤輸入不等於幫助使用者輸入正確。

### 2. `/trunk-based-development` - 「不穩的測試」不能跟「慢的測試」一樣處理

**從哪看出**：這次的 CI 偶發失敗（本機 15 次全過、CI 上必紅）處理方式是用 CPU 節流 6 倍在本機重現，修根因，再在節流下連續驗證才收工。這個做法來自使用者全域規則（看到測試不穩定就要修），不是來自 skill - skill 的文字甚至指向另一個方向。

**skill 現況**：「保持 green trunk」那條寫「慢 / 不穩的測試移到 pipeline 後段」，把「慢」和「不穩」綁在一起給同一個處方。但把不穩的測試往後搬，它還是不穩，只是晚一點紅。

**建議修改**：

> 區分「慢」與「不穩」：慢的測試可以移到 pipeline 後段；**不穩的測試不行** - 它在 pipeline 的任何位置都會讓紅燈失去意義，團隊會開始用「重跑一次」當作處理方式，trunk 的綠燈就不再代表任何事。偶發失敗要當成壞掉的 build 處理：優先重現（例如用 CPU／網路節流放大時序競態），修根因，並在修好後於放大條件下連續驗證數次才收工。

### 3. `/double-diamond-principle` - 沒有要求完成時回報「怎麼用了這個流程」（三支 skill 皆適用）

**從哪看出**：作業要觀察「每次功能完成後，Agent 能簡短指出這次如何使用 Double Diamond、測試結果、branch 與整合狀態，以及實際使用的 commit message」。看功能二、三、四完成時 agent 的回報：三次都有測試結果與分支／整合狀態；功能二提了一句「先定義對問題（雙鑽石）」，功能三與功能四完全沒提 Double Diamond 做了什麼；三次都沒有引用 commit message 原文（功能二只給 hash）；功能三的回報結尾還主動提議開做下一個功能。

**skill 現況**：三支 skill 的 `SKILL.md` 都沒有要求「完成時要回報什麼」。agent 做到的部分（測試、分支）是它自己的習慣，沒做到的部分（流程怎麼用、commit message）在 skill 裡找不到依據。

**建議新增規則（加在 Deliver 結尾，三支 skill 共用）**：

> 功能完成時，用四句話回報：(1) Double Diamond 四個階段各做了什麼、中途改了哪個判斷；(2) 測試結果與範圍；(3) 分支與整合狀態；(4) 實際使用的 commit message（原文）。回報完就停，不提議下一個功能。

## 衍生產出

- **[#47 的實作規格留言](https://github.com/KLTsai/next-is-you/issues/47#issuecomment-6538923721)**：六項 UX 決策連同理由，寫在動工前，讓需求單自己帶著「為什麼長這樣」。
- **[#47 的收尾留言](https://github.com/KLTsai/next-is-you/issues/47#issuecomment-6559348742)**：誠實列出三個仍未補的缺口 - 前端表單邏輯（`validateForm`、解析預覽、重新 render 規則）沒有單元測試、只被瀏覽器 E2E 間接覆蓋；合成醫院的降級只在 registry 層被證明，沒有 UI 層的證明；手動模式下「科別／醫師／診間不符」的檢查只在名冊模式生效，手動模式仍只靠後端 Start。
- **E2E 測試重寫**：`tests/test_local_acceptance.py` 的瀏覽器旅程與 `tests/test_rehearsal.py` 的 UI 測試都改寫成走串接式表單。
- **互動原型**（`.lavish/`，已加入 `.gitignore`，不進 repo）。

## 功能 2/3/4 的共同觀察：一個 session 連著跑完三輪

這一條寫在這裡統一講，因為它跨越三個功能。

本道館的設計是**一個功能一輪，一輪結束才開下一輪，每輪在新的 session 裡進行**，這樣每一輪的 skill 遵循狀況才能被分開觀察。實際上，功能二、三、四是在**同一個 session**（2026-10-29 開始，2026-10-31 合入最後一個 PR）裡做完的：

1. 功能二的 PR（#48）開出、CI 還在跑時，agent 主動問「你想等 CI 綠就先停，還是 CI 跑的同時我就先去規劃 #47？」，人類選了後者。本輪的 Deliver 還沒收尾，下一輪的 Discover 已經開始。
2. 功能三完成後緊接著做功能四，中間沒有停下來分段。
3. agent 是順著 issue 之間的 `Refs` / `Depends on` 連結往前推進（#45 → #47 → 後端切片 → 前端切片）。**這個 session 的開場 prompt 沒有寫「做完就停、下一個功能開新 session」，agent 也沒有讀過道館的功能表**（對話紀錄裡沒有任何一處提到道館 repo），所以它不知道 #45 在道館裡被拆成功能 2～5。「新 session」這個想法，人類在功能一的 session 結尾說過（「之後會開新的session來製作剩下的功能」），但沒有被帶進這個 session。issue 的相依邊界與道館的觀察邊界是兩件不同的事，而這個 session 只拿得到前者。

**連帶後果：功能五沒有被涵蓋。** 功能五（送出前擋掉填錯的看診資料）不是被合併進前三輪，是整個沒被當成題目處理過。PR #54 確實附帶加了兩條驗證（日期不能選過去、名冊模式下選項必須存在），但那是**本功能選單模式的必要配套**，不是功能五的範圍。功能五真正要求的 - 手動輸入模式下的錯誤攔阻、科別／醫師／診間組合不符的獨立驗證 - 至今沒有經過 Discover，而且手動模式的格式攔阻目前是**刻意不做**的（#47 的規格要求前端不得帶醫院格式知識）。這三個缺口記錄在 [#47 的收尾留言](https://github.com/KLTsai/next-is-you/issues/47#issuecomment-6559348742)，等它自己那一輪處理。

這個現象不是三支 skill 內容本身的問題 - 三支 skill 在每一輪的單一功能範圍內都有被遵循的證據（見三份文件的核對表），而「一功能一 session」是道館自己的實驗協定，沒有寫在任何一支 skill 裡，也沒有寫進這個 session 的 prompt。skill 能不能幫 agent 守住這個要求，是另一件事，記在功能二的[提案 2](./feature-02-hospital-adapter-boundary.md#2-double-diamond-principle---deliver-沒有交付完就停的規則agent-自己會提議下一個功能)。這裡只記錄協定沒有被帶進 session 的事實，以及下一輪的做法：

**給下一輪的執行規則**：每個功能開一個新 session，開場 prompt 寫明道館的功能編號、對應的需求單，以及「做完這一個就停，不規劃下一個」。該功能合入 main 並寫完觀察紀錄、更新道館表格之後，才在新 session 開始下一個編號。agent 若發現某張 issue 可以再切成更小的片，只回報切法，等人指派編號再動工。
