# GitGraph Links Feature - Implementation Checklist

## Overview

Implementing clickable link support for gitGraph diagrams (mermaid-js/mermaid#5599)

**Approach:** Click statement syntax only (matching flowchart pattern)

```
gitGraph
    commit id: "c1"
    commit id: "c2"
    click "c1" "https://example.com"
    click "c2" "https://example.com" "Tooltip" _blank
```

---

## Task Dependency Order

```
1.1 → 1.2 → 2.1 → 2.2 → 3.1 → 3.2 → 3.3 → 4.1 → 5.1 → 5.2 → 5.3 → 6.1
```

---

## Phase 1: Foundation

- [x] **Task 1.1** - Add GitGraphLink type definition
- [x] **Task 1.2** - Add link state management (setLink, getLink, getLinks, clear)

## Phase 2: Parser

- [x] **Task 2.1** - Add lexer tokens (CLICK, LINK_TARGET)
- [x] **Task 2.2** - Add clickStatement grammar rule

## Phase 3: Renderer

- [ ] **Task 3.1** - Add data-commit-id attribute to commit elements
- [ ] **Task 3.2** - Implement setupClickEvents function
- [ ] **Task 3.3** - Integrate bindFunctions into draw()

## Phase 4: Styles

- [ ] **Task 4.1** - Add clickable commit CSS styles

## Phase 5: Testing

- [ ] **Task 5.1** - Unit tests for link state management
- [ ] **Task 5.2** - Parser tests for click statement syntax
- [ ] **Task 5.3** - Cypress visual regression tests

## Phase 6: Documentation

- [ ] **Task 6.1** - Update gitgraph.md with click syntax docs

---

## Acceptance Criteria

### Functional

- [ ] `click "id" "url"` works
- [ ] `click "id" "url" "tooltip"` works
- [ ] `click "id" "url" _blank` works
- [ ] `click "id" "url" "tooltip" _blank` works
- [ ] Links work on commits referenced by id
- [ ] Works with all orientations (LR, TB, BT)

### UI/UX

- [ ] Cursor pointer on linked commits
- [ ] Hover visual feedback
- [ ] Tooltip displays on hover
- [ ] Keyboard accessible (Tab + Enter)
- [ ] Focus indicator visible

### Security

- [ ] javascript: URLs blocked
- [ ] data: URLs blocked
- [ ] Sandbox mode uses postMessage
- [ ] \_blank uses noopener,noreferrer

### Tests

- [ ] Unit tests pass
- [ ] Parser tests pass
- [ ] Cypress tests pass
