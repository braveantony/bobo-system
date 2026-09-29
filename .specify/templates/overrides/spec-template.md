# Feature Specification: [FEATURE NAME]

**Feature Branch**: `[###-feature-name]`

**Created**: [DATE]

**Status**: Draft

**Input**: 使用者描述："$ARGUMENTS"

## User Scenarios & Testing *(mandatory)*

<!--
  寫作提醒：內文一律用台灣繁體中文，技術名詞可以保留英文。
  用平常講話的方式寫，一句講一件事，不要官腔和 AI 套話。

  User story 請依重要性排序。每個 story 都要能單獨測試，
  也就是只做其中一個，也能交出一個可以用的最小版本。

  優先順序用 P1、P2、P3 標示，P1 最重要。每個 story 都要能：
  - 單獨開發
  - 單獨測試
  - 單獨部署
  - 單獨展示給使用者看
-->

### User Story 1 - [簡短標題] (Priority: P1)

[用白話描述使用者要完成的事情]

**Why this priority**: [說明它的價值，以及為什麼排這個順序]

**Independent Test**: [說明怎麼單獨測試，例如「做完 [某個操作] 就能確認 [某個結果]」]

**Acceptance Scenarios**:

1. **Given** [一開始的狀態]，**When** [使用者做了什麼]，**Then** [應該看到的結果]
2. **Given** [一開始的狀態]，**When** [使用者做了什麼]，**Then** [應該看到的結果]

---

### User Story 2 - [簡短標題] (Priority: P2)

[用白話描述使用者要完成的事情]

**Why this priority**: [說明它的價值，以及為什麼排這個順序]

**Independent Test**: [說明怎麼單獨測試]

**Acceptance Scenarios**:

1. **Given** [一開始的狀態]，**When** [使用者做了什麼]，**Then** [應該看到的結果]

---

### User Story 3 - [簡短標題] (Priority: P3)

[用白話描述使用者要完成的事情]

**Why this priority**: [說明它的價值，以及為什麼排這個順序]

**Independent Test**: [說明怎麼單獨測試]

**Acceptance Scenarios**:

1. **Given** [一開始的狀態]，**When** [使用者做了什麼]，**Then** [應該看到的結果]

---

[有需要就繼續加 user story，每個都要標優先順序]

### Edge Cases

<!--
  這一段是佔位內容，請換成這個功能真正會遇到的邊界狀況。
-->

- [某個邊界條件] 發生時會怎樣？
- 系統遇到 [某種錯誤] 時怎麼處理？

## Requirements *(mandatory)*

<!--
  這一段是佔位內容，請換成這個功能真正的功能需求。
  每一條都要具體到可以拿來測試。
-->

### Functional Requirements

- **FR-001**: 系統 MUST [具體能力，例如「讓使用者建立帳號」]
- **FR-002**: 系統 MUST [具體能力，例如「檢查 email 格式是否正確」]
- **FR-003**: 使用者 MUST 能夠 [主要操作，例如「重設密碼」]
- **FR-004**: 系統 MUST [資料需求，例如「記住使用者的偏好設定」]
- **FR-005**: 系統 MUST [行為，例如「記錄所有跟安全有關的事件」]

*還不確定的需求這樣標：*

- **FR-006**: 系統 MUST 用 [NEEDS CLARIFICATION: 還沒決定登入方式，email 加密碼、SSO 還是 OAuth？] 驗證使用者
- **FR-007**: 系統 MUST 保留使用者資料 [NEEDS CLARIFICATION: 還沒決定要保留多久]

### Key Entities *(include if feature involves data)*

- **[Entity 1]**: [它代表什麼，有哪些重要屬性，不用寫實作細節]
- **[Entity 2]**: [它代表什麼，跟其他 entity 有什麼關係]

## Success Criteria *(mandatory)*

<!--
  請寫可以量測的成功標準，不要綁定特定技術。
-->

### Measurable Outcomes

- **SC-001**: [可量測的指標，例如「使用者兩分鐘內就能建好帳號」]
- **SC-002**: [可量測的指標，例如「同時一千人在線，速度不會變慢」]
- **SC-003**: [使用者滿意度指標，例如「九成使用者第一次就能完成主要操作」]
- **SC-004**: [業務指標，例如「跟 [X] 有關的客服單減少一半」]

## Assumptions

<!--
  這一段是佔位內容。功能描述沒講清楚的地方，先用合理的預設值帶過，
  並寫在這裡讓大家知道。
-->

- [關於目標使用者的假設，例如「使用者的網路連線穩定」]
- [關於範圍的假設，例如「第一版不支援手機」]
- [關於資料或環境的假設，例如「沿用現有的登入機制」]
- [對既有系統或服務的依賴，例如「需要能呼叫現有的使用者資料 API」]
