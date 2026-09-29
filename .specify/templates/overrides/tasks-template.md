---

description: "功能實作用的任務清單模板"
---

# Tasks: [FEATURE NAME]

**Input**: 設計文件，位於 `/specs/[###-feature-name]/`

**Prerequisites**: plan.md（必要）、spec.md（user story 必要）、research.md、data-model.md、contracts/

**Tests**: 下面的範例有包含測試任務。測試是選配的，功能規格裡有明確要求才加。

**Organization**: 任務依 user story 分組，讓每個 story 都能單獨實作和測試。

<!--
  寫作提醒：任務描述一律用台灣繁體中文，技術名詞和檔案路徑維持英文。
  每個任務寫成一句具體的動作，看得出要改哪個檔案、做完長什麼樣子。
-->

## Format: `[ID] [P?] [Story] Description`

- **[P]**: 可以平行進行（不同檔案，彼此沒有相依）
- **[Story]**: 這個任務屬於哪個 user story（例如 US1、US2、US3）
- 描述裡要寫出完整的檔案路徑

## Path Conventions

- **單一專案**：`src/`、`tests/` 放在 repo 根目錄
- **Web 應用程式**：`backend/src/`、`frontend/src/`
- **行動裝置**：`api/src/`、`ios/src/` 或 `android/src/`
- 下面的路徑以單一專案為例，請依 plan.md 的結構調整

<!--
  ============================================================================
  重要：下面的任務只是範例。

  /speckit-tasks MUST 依照以下內容換成真正的任務：
  - spec.md 裡的 user story 和優先順序（P1、P2、P3……）
  - plan.md 裡的功能需求
  - data-model.md 裡的 entity
  - contracts/ 裡的 endpoint

  任務 MUST 依 user story 分組，讓每個 story 都能：
  - 單獨實作
  - 單獨測試
  - 當成一個 MVP 增量交付

  產出的 tasks.md 不能留下這些範例任務。
  ============================================================================
-->

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: 專案初始化和基本結構

- [ ] T001 依照實作計畫建立專案結構
- [ ] T002 初始化 [language] 專案並安裝 [framework] 相依套件
- [ ] T003 [P] 設定 linting 和格式化工具

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: 所有 user story 開始之前 MUST 先完成的核心基礎

**⚠️ CRITICAL**: 這個階段完成前，任何 user story 都不能開始

基礎任務的範例（請依專案調整）：

- [ ] T004 建立資料庫 schema 和 migration 機制
- [ ] T005 [P] 實作身分驗證和權限控管
- [ ] T006 [P] 建立 API 路由和 middleware 結構
- [ ] T007 建立所有 story 都會用到的基本 model 和 entity
- [ ] T008 設定錯誤處理和 log 機制
- [ ] T009 建立環境設定的管理方式

**Checkpoint**: 基礎完成，user story 可以開始平行進行

---

## Phase 3: User Story 1 - [Title] (Priority: P1) 🎯 MVP

**Goal**: [簡單說明這個 story 會交付什麼]

**Independent Test**: [怎麼單獨確認這個 story 可以用]

### Tests for User Story 1 (OPTIONAL - only if tests requested) ⚠️

> **NOTE: 先寫這些測試，確定它們在實作前會失敗**

- [ ] T010 [P] [US1] 在 tests/contract/test_[name].py 寫 [endpoint] 的 contract 測試
- [ ] T011 [P] [US1] 在 tests/integration/test_[name].py 寫 [使用流程] 的整合測試

### Implementation for User Story 1

- [ ] T012 [P] [US1] 在 src/models/[entity1].py 建立 [Entity1] model
- [ ] T013 [P] [US1] 在 src/models/[entity2].py 建立 [Entity2] model
- [ ] T014 [US1] 在 src/services/[service].py 實作 [Service]（相依 T012、T013）
- [ ] T015 [US1] 在 src/[location]/[file].py 實作 [endpoint 或功能]
- [ ] T016 [US1] 加上輸入驗證和錯誤處理
- [ ] T017 [US1] 幫 user story 1 的操作加上 log

**Checkpoint**: 到這裡，User Story 1 應該已經完整可用，也能單獨測試

---

## Phase 4: User Story 2 - [Title] (Priority: P2)

**Goal**: [簡單說明這個 story 會交付什麼]

**Independent Test**: [怎麼單獨確認這個 story 可以用]

### Tests for User Story 2 (OPTIONAL - only if tests requested) ⚠️

- [ ] T018 [P] [US2] 在 tests/contract/test_[name].py 寫 [endpoint] 的 contract 測試
- [ ] T019 [P] [US2] 在 tests/integration/test_[name].py 寫 [使用流程] 的整合測試

### Implementation for User Story 2

- [ ] T020 [P] [US2] 在 src/models/[entity].py 建立 [Entity] model
- [ ] T021 [US2] 在 src/services/[service].py 實作 [Service]
- [ ] T022 [US2] 在 src/[location]/[file].py 實作 [endpoint 或功能]
- [ ] T023 [US2] 跟 User Story 1 的元件整合（有需要才做）

**Checkpoint**: 到這裡，User Story 1 和 2 都應該能各自運作

---

## Phase 5: User Story 3 - [Title] (Priority: P3)

**Goal**: [簡單說明這個 story 會交付什麼]

**Independent Test**: [怎麼單獨確認這個 story 可以用]

### Tests for User Story 3 (OPTIONAL - only if tests requested) ⚠️

- [ ] T024 [P] [US3] 在 tests/contract/test_[name].py 寫 [endpoint] 的 contract 測試
- [ ] T025 [P] [US3] 在 tests/integration/test_[name].py 寫 [使用流程] 的整合測試

### Implementation for User Story 3

- [ ] T026 [P] [US3] 在 src/models/[entity].py 建立 [Entity] model
- [ ] T027 [US3] 在 src/services/[service].py 實作 [Service]
- [ ] T028 [US3] 在 src/[location]/[file].py 實作 [endpoint 或功能]

**Checkpoint**: 所有 user story 都應該能各自運作

---

[有需要就照同樣格式繼續加 user story 階段]

---

## Phase N: Polish & Cross-Cutting Concerns

**Purpose**: 會影響多個 user story 的改善

- [ ] TXXX [P] 更新 docs/ 裡的文件
- [ ] TXXX 整理程式碼和重構
- [ ] TXXX 針對所有 story 做效能最佳化
- [ ] TXXX [P] 在 tests/unit/ 補單元測試（有要求才做）
- [ ] TXXX 加強安全性
- [ ] TXXX 照 quickstart.md 跑一次驗證

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**：沒有相依，可以馬上開始
- **Foundational (Phase 2)**：要等 Setup 完成，會卡住所有 user story
- **User Stories (Phase 3+)**：都要等 Foundational 完成
  - 之後人手夠的話，user story 可以平行做
  - 不然就照優先順序一個一個來（P1、P2、P3）
- **Polish (最後階段)**：要等想做的 user story 都完成

### User Story Dependencies

- **User Story 1 (P1)**：Foundational（Phase 2）完成後就能開始，不相依其他 story
- **User Story 2 (P2)**：Foundational（Phase 2）完成後就能開始，可能會跟 US1 整合，但要能單獨測試
- **User Story 3 (P3)**：Foundational（Phase 2）完成後就能開始，可能會跟 US1、US2 整合，但要能單獨測試

### Within Each User Story

- 有寫測試的話，MUST 先寫測試，而且要在實作前確認會失敗
- 先做 model，再做 service
- 先做 service，再做 endpoint
- 先完成核心實作，再做整合
- 一個 story 做完，才換下一個優先順序

### Parallel Opportunities

- Setup 裡標 [P] 的任務都可以平行做
- Foundational 裡標 [P] 的任務都可以平行做（限 Phase 2 內）
- Foundational 完成後，人手夠的話所有 user story 都能同時開始
- 同一個 user story 裡標 [P] 的測試都可以平行做
- 同一個 story 裡標 [P] 的 model 都可以平行做
- 不同的 user story 可以由不同的人同時進行

---

## Parallel Example: User Story 1

```bash
# User Story 1 的測試一起開始（有要求測試的話）：
Task: "在 tests/contract/test_[name].py 寫 [endpoint] 的 contract 測試"
Task: "在 tests/integration/test_[name].py 寫 [使用流程] 的整合測試"

# User Story 1 的 model 一起開始：
Task: "在 src/models/[entity1].py 建立 [Entity1] model"
Task: "在 src/models/[entity2].py 建立 [Entity2] model"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. 完成 Phase 1：Setup
2. 完成 Phase 2：Foundational（很重要，會卡住所有 story）
3. 完成 Phase 3：User Story 1
4. **先停下來驗證**：單獨測試 User Story 1
5. 沒問題就部署或展示

### Incremental Delivery

1. 完成 Setup 和 Foundational，基礎就緒
2. 加上 User Story 1，單獨測試，部署或展示（這就是 MVP）
3. 加上 User Story 2，單獨測試，部署或展示
4. 加上 User Story 3，單獨測試，部署或展示
5. 每個 story 都會增加價值，而且不會弄壞前面的 story

### Parallel Team Strategy

有多位開發者時：

1. 大家一起完成 Setup 和 Foundational
2. Foundational 完成後：
   - 開發者 A：User Story 1
   - 開發者 B：User Story 2
   - 開發者 C：User Story 3
3. 各個 story 各自完成、各自整合

---

## Notes

- [P] 任務代表不同檔案、彼此沒有相依
- [Story] 標籤用來對應任務屬於哪個 user story，方便追蹤
- 每個 user story 都要能單獨完成和測試
- 實作前先確認測試會失敗
- 每完成一個任務或一組相關任務就 commit
- 可以在任何 checkpoint 停下來單獨驗證 story
- 要避免：模糊的任務、多個任務改同一個檔案、破壞 story 獨立性的跨 story 相依
