---
name: git-conventional-commit
description: >-
  依 Conventional Commits v1.0.0 規範撰寫、檢查、修正 git commit 訊息。只要使用者要
  commit、寫 commit message、整理 commit 歷史、設定 commitlint / semantic-release，
  或問「這則 commit 訊息對不對」「這次變更該用哪個 type」「要不要標 BREAKING CHANGE」，
  就使用本 skill。即使使用者沒講出「conventional commits」這個詞，只要牽涉到 git 提交訊息
  的格式與語意，也要主動套用。
---

# Conventional Commits（慣例式提交）

一套疊加在 git commit 訊息上的輕量約定，讓訊息「人類可讀、機器也可讀」，並與 Semantic
Versioning（SemVer）直接對應。遵守它，工具就能自動產生 CHANGELOG、自動決定版本號遞增、
觸發 build／publish，團隊也更容易讀懂歷史。

本 skill 已依官方 v1.0.0 規格整理驗證；完整條文、FAQ、原始出處見 `references/specification.md`。

## 核心格式

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

精確規則（標點與空行不可省）：

- `<type>` 後可接括號 scope、可接 `!`，然後**必須**是冒號加一個空白 `: `。
- `<description>` 緊接 `: ` 之後，同一行，精簡說明「這次變更做了什麼」。
- description 與 body 之間空**一個**空白行；body 與 footer 之間也空一個空白行。
- footer 每則為 `token: value` 或 `token #value`（後者常用來連 issue）。
- footer token 以 `-` 取代空白（例 `Reviewed-by`、`Refs`）。唯一例外是 `BREAKING CHANGE`。

## 撰寫步驟

1. **看清這次變更的本質**，再決定 type。先確認是「新功能／修 bug／其他」，再看是否破壞相容性。
   如果一次變更同時符合多個 type，優先**拆成多個 commit**，讓每則 commit 聚焦。
2. **選 type**（見下表）。規格只明定 `feat` / `fix`，其餘為業界慣例。
3. **判斷是否為破壞性變更**（breaking change）。是的話一定要標示（兩種寫法見下）。
4. **寫 description**：祈使句、精簡、聚焦變更本身。把訊息當成「產品已經發布、別人會依賴它」來寫。
5. **需要時補 body**：說明動機、對比、為什麼這樣改（不是重述改了什麼）。
6. **需要時補 footer**：如 `Refs: #123`、`Reviewed-by: Z`，或 `BREAKING CHANGE:`。
7. **自我檢查**：冒號後有空白？description 非空？breaking 有標？type 拼對？大小寫一致？

## type 對照表

| type | 用途 | SemVer |
|------|------|--------|
| `feat` | 新增功能（規格明定） | MINOR |
| `fix` | 修補 bug（規格明定） | PATCH |
| `docs` | 僅文件變更 | - |
| `style` | 不影響邏輯的格式變更（空白、排版、分號） | - |
| `refactor` | 不修 bug 也不加功能的重構 | - |
| `perf` | 提升效能 | - |
| `test` | 新增或修正測試 | - |
| `build` | 影響建置系統或外部相依（npm、webpack 等） | - |
| `ci` | 變更 CI 設定與腳本（GitHub Actions 等） | - |
| `chore` | 其他不動 src/test 的雜務 | - |
| `revert` | 還原先前 commit，footer 以 `Refs:` 列 SHA | - |

> 任何 type 只要帶破壞性變更，一律對應 SemVer **MAJOR**（含 `chore!`、`refactor!`）。

## 破壞性變更的標示（兩種，可併用）

**方式一 - footer（完整寫法，必須全大寫）：**
```
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files
```

**方式二 - `!` 前綴（簡寫，放在冒號之前）：**
```
feat(api)!: send an email to the customer when a product is shipped
```

用 `!` 時可省略 footer 的 `BREAKING CHANGE:`，此時 description 就是破壞性變更的說明。
footer token 也可寫 `BREAKING-CHANGE:`（連字號），視為同義。

## 範例

**Example 1 - 帶 body 與多個 footer：**
Input: 加入 request id 與 latest request 參照來避免請求競態，並移除已無用的 timeout
Output:
```
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Reviewed-by: Z
Refs: #123
```

**Example 2 - 含 scope 的功能新增：**
Input: 為語言模組新增波蘭文
Output: `feat(lang): add Polish language`

**Example 3 - 文件修正（無 body）：**
Input: 修正 CHANGELOG 的拼字
Output: `docs: correct spelling of CHANGELOG`

**Example 4 - 破壞性變更（`!` + footer 併用）：**
Input: 不再支援 Node 6
Output:
```
chore!: drop support for Node 6

BREAKING CHANGE: use JavaScript features not available in Node 6.
```

## 規則重點與例外

- **大小寫**：除 `BREAKING CHANGE` 必須全大寫外，其餘元素不區分大小寫；但團隊宜選定一種並一致。
- **拼錯 type**：若是規格內 type 用錯（該 `feat` 寫成 `fix`）且尚未 merge/release，用
  `git rebase -i` 修正；用了不存在的 type（如 `feet`）不會出錯，只是工具不認得這則 commit。
- **squash 工作流**：臨時貢獻者不必人人遵守，維護者可在 merge/squash 時重寫最終訊息。
- **revert**：規格未強制定義，建議用 `revert` type 搭配 footer 列出被還原的 SHA。
- 導入自動化時常用 `@commitlint/config-conventional` + husky hook 驗證格式，搭配
  `semantic-release` / `conventional-changelog` 自動發版與產生 changelog。

完整規格 1-16 條、FAQ、SemVer 對應與原始出處：見 `references/specification.md`。
