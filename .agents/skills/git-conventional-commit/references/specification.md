# Conventional Commits v1.0.0 - 完整規格與出處

> 本檔保留完整條文、FAQ 與資料來源，供回去核對。摘要版在 `../SKILL.md`。

## 目錄
- [規格 1-16 條](#規格-1-16-條)
- [術語定義](#術語定義)
- [與 SemVer 的對應](#與-semver-的對應)
- [FAQ 重點](#faq-重點)
- [資料來源](#資料來源)

## 規格 1-16 條

條文使用 RFC 2119 關鍵字（MUST / MUST NOT / REQUIRED / SHALL / SHOULD / MAY）。

1. 每則 commit **MUST** 以一個 type 開頭，type 為名詞（如 `feat`、`fix`），後接選填 scope、選填 `!`，以及必填的冒號加空白 `: `。
2. 新增功能時 type **MUST** 使用 `feat`。
3. 修補 bug 時 type **MUST** 使用 `fix`。
4. scope **MAY** 跟在 type 之後提供額外脈絡，為括號包住的名詞，例如 `fix(parser):`。
5. description **MUST** 緊接在 type/scope 前綴的冒號加空白之後，是對變更的簡短摘要。
6. 在 description 之後 **MAY** 提供較長 body，body **MUST** 以一個空白行為開始。
7. body 為自由格式，**MAY** 由任意數量、以空白行分隔的段落組成。
8. 在 body 之後 **MAY** 提供一或多個 footer，彼此以空白行隔開；每個 footer **MUST** 由 word token + 分隔符（`: ` 或 ` #`）+ 字串 value 組成（源自 git trailer 慣例）。
9. footer 的 token **MUST** 以 `-` 取代空白（例 `Acked-by`），唯一例外 `BREAKING CHANGE` **MAY** 當作 token（可含空白）。
10. footer 的 value **MAY** 含空白與換行；解析 **MUST** 在偵測到下一個合法 footer token/分隔符時終止。
11. 破壞性變更 **MUST** 被標示：標在 type/scope 前綴，或作為一個 footer。
12. 以 footer 標示時 **MUST** 由大寫 `BREAKING CHANGE` + 冒號 + 空白 + 描述組成。
13. 以前綴標示時 **MUST** 以 `!` 緊接在 type/scope 的冒號之前；用 `!` 時 footer **MAY** 省略 `BREAKING CHANGE:`，description **SHALL** 用來描述破壞性變更。
14. type **MAY** 使用 `feat`／`fix` 以外的類型，例如 `docs:`。
15. 各單元實作者 **MUST NOT** 視為大小寫敏感，唯一例外 `BREAKING CHANGE` **MUST** 為大寫。
16. footer 的 `BREAKING-CHANGE`（連字號）**MUST** 與 `BREAKING CHANGE` 視為同義。

## 術語定義

| 術語 | 說明 |
|------|------|
| type | commit 最前的分類名詞，必填。 |
| scope | type 之後括號內的名詞，標示影響範圍，選填。 |
| description | 冒號空白後同一行的簡短摘要，必填。 |
| body | description 後空一行開始的自由格式內文，選填。 |
| footer | body 後空一行的 metadata，格式 `token: value` 或 `token #value`。 |
| footer token | footer 開頭標籤，以 `-` 取代空白；例外 `BREAKING CHANGE`。 |
| footer separator | `: `（冒號空白）或 ` #`（空白井字號）。 |
| BREAKING CHANGE | 破壞性變更關鍵字，footer 須全大寫；或用 `!` 前綴。對應 SemVer MAJOR。 |
| `!` | 放在冒號前的破壞性變更簡寫，例 `feat!:`、`feat(api)!:`。 |

## 與 SemVer 的對應

SemVer 版本號 `MAJOR.MINOR.PATCH`：MAJOR 為不相容 API 變更、MINOR 為向下相容新功能、PATCH 為向下相容 bug 修正。

| Commit | SemVer |
|--------|--------|
| `fix:` | PATCH（1.0.0 → 1.0.1） |
| `feat:` | MINOR（1.0.0 → 1.1.0） |
| 任何帶 `BREAKING CHANGE` / `!`（不論 type） | MAJOR（1.0.0 → 2.0.0） |

Conventional Commits「深受 Angular Commit Guidelines 啟發且大量以其為基礎」。常用 type 清單（docs/style/refactor/perf/test/build/ci/chore）源自 Angular convention 與 `@commitlint/config-conventional`，非規格強制。

## FAQ 重點

1. **初期開發怎麼寫？** 當成產品已發布、別人會依賴它來寫。
2. **type 大寫還小寫？** 皆可，但團隊選一種並一致。
3. **一個 commit 符合多個 type？** 盡量拆成多個 commit。
4. **會拖慢迭代嗎？** 它阻止的是雜亂地快，目的是長期持續地快。
5. **會侷限思考嗎？** 可自訂 type 並隨時間演進。
6. **與 SemVer 關係？** 見上表。
7. **擴充怎麼版本化？** 用 SemVer 管理自己的擴充。
8. **打錯 type？** 規格內用錯且未 merge/release → `git rebase -i` 修正；用了不存在的 type → 工具不認得但非災難。
9. **所有人都要遵守嗎？** 不用，squash 工作流下維護者可重寫最終訊息。
10. **revert 怎麼處理？** 規格未強制；建議 `revert` type + footer 列被還原 SHA。

## 資料來源

實際讀取的頁面：
- https://www.conventionalcommits.org/en/v1.0.0/ （主規格頁）
- https://www.conventionalcommits.org/en/v1.0.0/#specification （Specification 1-16 條）
- https://www.conventionalcommits.org/en/v1.0.0/#faq （FAQ）
- https://www.conventionalcommits.org/en/about/ （起源、與 Angular 關係、授權 CC BY 3.0）

方法論背景（頁面引用）：
- https://semver.org/ （Semantic Versioning）
- Angular Commit Guidelines（規格所本）
- `@commitlint/config-conventional`（常用 type 清單來源）
- RFC 2119（MUST/SHOULD/MAY 定義）

> 註：官方規格強制的 type 僅 `feat`／`fix`；其餘為業界慣例（Angular / commitlint），使用時宜區分「規格明定」與「業界慣例」。
