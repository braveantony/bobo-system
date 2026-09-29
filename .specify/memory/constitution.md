# bobo Constitution

## Core Principles

### I. Taiwan Traditional Chinese

spec-kit 產出的所有文件 MUST 用台灣繁體中文撰寫。包含 constitution、spec、plan、tasks、
checklist、分析報告，以及轉成 GitHub issue 的文字。

- 用詞 MUST 採台灣慣用說法，例如專案、程式碼、資料、設定、使用者、伺服器。
- MUST NOT 使用中國大陸用詞，例如項目、代碼、數據、配置、用戶、服務器。
- 標題、欄位名稱和技術名詞可以維持英文，例如 User Story、Acceptance Scenarios、API、commit。
- spec-kit 需要比對的固定字串 MUST 維持原樣，例如 NEEDS CLARIFICATION、Given、When、Then、
  FR-001、T001、[P]、[US1]、CHK001。

理由：這個專案的使用者和維護者都在台灣，文件要讓人一看就懂。

### II. Human Voice

文件 MUST 讀起來像同事寫的，不能有 AI 感。

- 一句話講一件事，直接講重點，不打官腔。
- MUST NOT 使用套話，例如「值得注意的是」「綜上所述」「賦能」「閉環」「無縫整合」
  「提供全面性的解決方案」。
- 有幾點就寫幾點，不為了看起來完整硬湊三點或五點。
- 粗體和 emoji 只在真的需要時才用，內容短就寫成段落，不要每段都下小標題。

理由：制式的 AI 文字讀起來費力，重點容易被淹沒，也會讓人懷疑內容有沒有經過思考。

### III. Concrete and Testable

需求、驗收條件和任務 MUST 寫到可以拿來測試或驗證。

- 寫具體行為和數字，例如「密碼連續錯三次，帳號鎖定十分鐘」。
  不要寫「提供完善的安全機制」這種看不出怎麼驗收的句子。
- 不確定的地方 MUST 直接標出來，用 NEEDS CLARIFICATION 或「待確認：……」，
  不能用模糊的形容詞帶過。

理由：寫不具體的需求沒辦法驗收，最後只會在實作時才發現大家想的不一樣。

## Terminology

完整的用詞對照表放在 repo 根目錄的 AGENT.md。新增常用詞時更新那份表，
不要在各個 spec 裡各自定義。

## Review Process

每份 spec-kit 產出在 commit 前，作者 MUST 檢查三件事：

- 有沒有中國大陸用詞。
- 有沒有 Human Voice 列出的套話。
- 每條需求和驗收條件能不能拿來測試。

有任何一項不符合就先改掉再 commit。

## Governance

這份 constitution 的規則優先於其他寫作習慣。修改時 MUST 用 `/speckit-constitution`，
並依以下規則調整版本號：

- MAJOR：刪除或重新定義既有原則。
- MINOR：新增原則或章節，或大幅擴充既有規則。
- PATCH：修正文字、錯字，不影響規則本身。

所有 PR 和 review 都要檢查文件有沒有符合這份 constitution。日常的寫作細節請看 AGENT.md。

**Version**: 1.0.0 | **Ratified**: 2026-09-29 | **Last Amended**: 2026-09-29
