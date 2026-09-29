# Implementation Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]

**Input**: 功能規格，位於 `/specs/[###-feature-name]/spec.md`

**Note**: 這份模板由 `/speckit-plan` 填寫，執行流程寫在該指令的定義裡。

<!--
  寫作提醒：內文一律用台灣繁體中文，技術名詞可以保留英文。
  直接講做法和理由，不要寫「提供全面的解決方案」這類空話。
-->

## Summary

[從功能規格整理出來：主要需求是什麼，研究後決定用什麼技術做法]

## Technical Context

<!--
  請把這一段換成這個專案實際的技術細節。
  下面的欄位只是參考，幫助你把該想的事情想清楚。
-->

**Language/Version**: [例如 Node.js 20、Python 3.11，或 NEEDS CLARIFICATION]

**Primary Dependencies**: [例如 Express、mysql2，或 NEEDS CLARIFICATION]

**Storage**: [有用到才寫，例如 MySQL、PostgreSQL、檔案，或 N/A]

**Testing**: [例如 Jest、Vitest、pytest，或 NEEDS CLARIFICATION]

**Target Platform**: [例如 Linux 伺服器、Docker、Kubernetes，或 NEEDS CLARIFICATION]

**Project Type**: [例如 library、cli、web-service、mobile-app、desktop-app，或 NEEDS CLARIFICATION]

**Performance Goals**: [依領域而定，例如每秒 1000 個請求、畫面 60 fps，或 NEEDS CLARIFICATION]

**Constraints**: [依領域而定，例如 p95 回應時間低於 200ms、記憶體低於 100MB、要能離線使用，或 NEEDS CLARIFICATION]

**Scale/Scope**: [依領域而定，例如一萬個使用者、50 個畫面，或 NEEDS CLARIFICATION]

## Constitution Check

*GATE：Phase 0 研究開始前要先通過，Phase 1 設計完成後再檢查一次。*

[依照 constitution 檔案列出要檢查的項目]

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # 這份檔案（/speckit-plan 產出）
├── research.md          # Phase 0 產出（/speckit-plan）
├── data-model.md        # Phase 1 產出（/speckit-plan）
├── quickstart.md        # Phase 1 產出（/speckit-plan）
├── contracts/           # Phase 1 產出（/speckit-plan）
└── tasks.md             # Phase 2 產出（/speckit-tasks，不是 /speckit-plan 產生的）
```

### Source Code (repository root)
<!--
  請把下面的目錄樹換成這個功能實際的結構。
  用不到的選項刪掉，選定的結構展開成真實路徑（例如 apps/admin、packages/something）。
  最後交出去的計畫不能留下 Option 標籤。
-->

```text
# [REMOVE IF UNUSED] Option 1: 單一專案（預設）
src/
├── models/
├── services/
├── cli/
└── lib/

tests/
├── contract/
├── integration/
└── unit/

# [REMOVE IF UNUSED] Option 2: Web 應用程式（偵測到 frontend 和 backend 時）
backend/
├── src/
│   ├── models/
│   ├── services/
│   └── api/
└── tests/

frontend/
├── src/
│   ├── components/
│   ├── pages/
│   └── services/
└── tests/

# [REMOVE IF UNUSED] Option 3: 行動裝置加 API（偵測到 iOS 或 Android 時）
api/
└── [跟上面的 backend 一樣]

ios/ or android/
└── [平台專屬結構：功能模組、畫面流程、平台測試]
```

**Structure Decision**: [寫下選了哪種結構，並對應到上面列的實際目錄]

## Complexity Tracking

> **只有在 Constitution Check 有違反項目、而且必須說明理由時才填**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [例如多開第四個專案] | [目前的需求] | [為什麼三個專案不夠] |
| [例如使用 Repository pattern] | [要解決的具體問題] | [為什麼直接存取資料庫不行] |
