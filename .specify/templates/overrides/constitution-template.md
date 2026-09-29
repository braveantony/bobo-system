# [PROJECT_NAME] Constitution
<!-- 範例：Spec Constitution、TaskFlow Constitution -->
<!-- 寫作提醒：內文一律用台灣繁體中文，技術名詞可以保留英文。原則要寫到可以檢查，不要寫口號。 -->

## Core Principles

### [PRINCIPLE_1_NAME]
<!-- 範例：I. Library-First -->
[PRINCIPLE_1_DESCRIPTION]
<!-- 範例：每個功能都先做成獨立的 library。library 要能自己運作、單獨測試，也要有文件。每個 library 都要有明確用途，不能只是為了整理程式碼而拆。 -->

### [PRINCIPLE_2_NAME]
<!-- 範例：II. CLI Interface -->
[PRINCIPLE_2_DESCRIPTION]
<!-- 範例：每個 library 都透過 CLI 提供功能。輸入從 stdin 或參數進來，結果輸出到 stdout，錯誤輸出到 stderr。同時支援 JSON 和人看得懂的格式。 -->

### [PRINCIPLE_3_NAME]
<!-- 範例：III. Test-First (NON-NEGOTIABLE) -->
[PRINCIPLE_3_DESCRIPTION]
<!-- 範例：一定要 TDD。先寫測試，使用者確認過，確定測試會失敗，才開始實作。嚴格走 Red-Green-Refactor。 -->

### [PRINCIPLE_4_NAME]
<!-- 範例：IV. Integration Testing -->
[PRINCIPLE_4_DESCRIPTION]
<!-- 範例：以下情況一定要寫整合測試：新 library 的 contract 測試、contract 有變動、服務之間的溝通、共用的 schema。 -->

### [PRINCIPLE_5_NAME]
<!-- 範例：V. Observability、VI. Versioning & Breaking Changes、VII. Simplicity -->
[PRINCIPLE_5_DESCRIPTION]
<!-- 範例：文字輸入輸出讓除錯更容易，log 要結構化。或是：版本用 MAJOR.MINOR.BUILD。或是：先從簡單的做起，遵守 YAGNI。 -->

## [SECTION_2_NAME]
<!-- 範例：Additional Constraints、Security Requirements、Performance Standards -->

[SECTION_2_CONTENT]
<!-- 範例：技術選型的要求、法規遵循標準、部署政策 -->

## [SECTION_3_NAME]
<!-- 範例：Development Workflow、Review Process、Quality Gates -->

[SECTION_3_CONTENT]
<!-- 範例：code review 的要求、測試關卡、部署的核准流程 -->

## Governance
<!-- 範例：constitution 的效力高於其他所有做法。要修改必須留下紀錄、經過核准，並附上遷移計畫。 -->

[GOVERNANCE_RULES]
<!-- 範例：所有 PR 和 review 都要檢查有沒有符合 constitution。增加複雜度一定要說明理由。開發時的日常指引請看 [GUIDANCE_FILE]。 -->

**Version**: [CONSTITUTION_VERSION] | **Ratified**: [RATIFICATION_DATE] | **Last Amended**: [LAST_AMENDED_DATE]
<!-- 範例：Version: 2.1.1 | Ratified: 2025-06-13 | Last Amended: 2025-07-16 -->
