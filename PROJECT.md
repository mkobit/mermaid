# GitGraph Links Feature - Implementation Checklist

## Overview

Implementing clickable link support for gitGraph diagrams (mermaid-js/mermaid#5599)

**Approach:**
- `click "id" "url"` (Commits, Merges, Cherry-picks - Default)
- `click commit "id" "url"` (Commits - Explicit)
- `click branch "name" "url"` (Branches)
- `click tag "name" "url"` (Tags)

---

## Task Dependency Order

```
1.1 → 1.2 → 2.1 → 2.2 → 3.1 → 3.2 → 3.3 → 4.1 → 5.1 → 5.2 → 5.3 → 6.1
(New Tasks: 1.3, 2.3, 2.4, 3.4, 3.5, 3.6, 5.4)
```

---

## Phase 1: Foundation

- [x] **Task 1.1** - Add GitGraphLink type definition
- [x] **Task 1.2** - Add link state management (setLink, getLink, getLinks, clear)
- [x] **Task 1.3** - Update GitGraphLink to support link types (commit, branch, tag)

## Phase 2: Parser

- [x] **Task 2.1** - Add lexer tokens (CLICK, LINK_TARGET)
- [x] **Task 2.2** - Add clickStatement grammar rule
- [x] **Task 2.3** - Update grammar to support `click branch` and `click tag`
- [x] **Task 2.4** - Update parser mapping to handle new link types

## Phase 3: Renderer

- [x] **Task 3.1** - Add data-commit-id attribute to commit elements
- [x] **Task 3.2** - Implement setupClickEvents function
- [x] **Task 3.3** - Integrate bindFunctions into draw()
- [x] **Task 3.4** - Add data-branch-name attribute to branch elements
- [x] **Task 3.5** - Add data-tag-name attribute to tag elements
- [x] **Task 3.6** - Update setupClickEvents to handle multiple link types

## Phase 4: Styles

- [x] **Task 4.1** - Add clickable commit CSS styles

## Phase 5: Testing

- [x] Task 5.1 - Unit tests for link state management
- [x] Task 5.2 - Parser tests for click statement syntax
- [x] Task 5.3 - Cypress visual regression tests
- [x] Task 5.4 - Tests for branch and tag links (Parser and Unit)

## Phase 6: Documentation

- [ ] **Task 6.1** - Update gitgraph.md with click syntax docs

---

## Acceptance Criteria

### Functional

- [x] `click "id" "url"` works (Commits)
- [x] `click commit "id" "url"` works
- [x] `click branch "name" "url"` works
- [x] `click tag "name" "url"` works
- [ ] Links work on commits referenced by id
- [ ] Works with all orientations (LR, TB, BT)

### UI/UX

- [x] Cursor pointer on linked elements (commits, branches, tags)
- [ ] Hover visual feedback
- [ ] Tooltip displays on hover
- [ ] Keyboard accessible (Tab + Enter)

### Security

- [ ] javascript: URLs blocked
- [ ] data: URLs blocked
- [ ] Sandbox mode uses postMessage
- [ ] \_blank uses noopener,noreferrer
