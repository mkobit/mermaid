# GitGraph Links Feature - Implementation Tasks

## Overview
Implementing clickable link support for gitGraph diagrams (mermaid-js/mermaid#5599)

---

## Phase 1: Foundation

### Task 1.1: Add Type Definitions
**File:** `packages/mermaid/src/diagrams/git/gitGraphTypes.ts`

**Objective:** Add `GitGraphLink` interface

**Implementation:**
```typescript
export interface GitGraphLink {
  id: string;
  link: string;
  tooltip?: string;
  target: '_self' | '_blank' | '_parent' | '_top';
}
```

**Acceptance:**
- [ ] Interface exported from gitGraphTypes.ts
- [ ] TypeScript compiles without errors

---

### Task 1.2: Add Link State Management to AST
**File:** `packages/mermaid/src/diagrams/git/gitGraphAst.ts`

**Objective:** Add link storage and management functions

**Implementation:**
1. Add module-level `links: Map<string, GitGraphLink>`
2. Implement `setLink(id, link, tooltip?, target?)`
3. Implement `getLinks()` returning Map copy
4. Implement `getLink(id)` returning single link or undefined
5. Modify `clear()` to reset links Map
6. Export all three functions in default export object

**Acceptance:**
- [ ] `setLink` stores link with default target `_self`
- [ ] `getLinks` returns defensive copy
- [ ] `clear()` resets links
- [ ] Functions exported in default object

---

### Task 1.3: Extend Commit Function for Inline Links
**File:** `packages/mermaid/src/diagrams/git/gitGraphAst.ts`

**Objective:** Handle inline link options in commit()

**Implementation:**
Modify `commit(options)` to call `setLink()` when `options.link` exists, passing `options.tooltip` and `options.linkTarget`

**Acceptance:**
- [ ] `commit({ id: "x", link: "url" })` creates link entry
- [ ] tooltip and linkTarget passed through when present

---

## Phase 2: Parser

### Task 2.1: Add Lexer Tokens
**File:** `packages/mermaid/src/diagrams/git/parser/gitGraph.jison`

**Objective:** Add lexer rules for link-related keywords

**Implementation (in %lex section):**
```jison
"link"          return 'LINK';
"click"         return 'CLICK';
"tooltip"       return 'TOOLTIP';
"target"        return 'TARGET';
"_blank"        return 'LINK_TARGET';
"_self"         return 'LINK_TARGET';
"_parent"       return 'LINK_TARGET';
"_top"          return 'LINK_TARGET';
```

**Acceptance:**
- [ ] Tokens recognized by lexer
- [ ] No conflicts with existing tokens

---

### Task 2.2: Extend commitOpt Grammar Rule
**File:** `packages/mermaid/src/diagrams/git/parser/gitGraph.jison`

**Objective:** Add link/tooltip/target options to commitOpt

**Implementation:**
Add to `commitOpt` rule:
```jison
| LINK COLON STR        { $$ = { link: $3 } }
| TOOLTIP COLON STR     { $$ = { tooltip: $3 } }
| TARGET COLON LINK_TARGET  { $$ = { linkTarget: $3 } }
```

**Acceptance:**
- [ ] `commit id: "x" link: "url"` parses
- [ ] `commit id: "x" link: "url" tooltip: "tip" target: "_blank"` parses
- [ ] Options merge correctly with existing commitOpt handling

---

### Task 2.3: Add clickStatement Grammar Rule
**File:** `packages/mermaid/src/diagrams/git/parser/gitGraph.jison`

**Objective:** Add click statement syntax

**Implementation:**
```jison
clickStatement
    : CLICK STR STR                     { yy.setLink($2, $3) }
    | CLICK STR STR STR                 { yy.setLink($2, $3, $4) }
    | CLICK STR STR LINK_TARGET         { yy.setLink($2, $3, undefined, $4) }
    | CLICK STR STR STR LINK_TARGET     { yy.setLink($2, $3, $4, $5) }
    ;
```

Add `clickStatement` to `statement` rule.

**Acceptance:**
- [ ] `click "id" "url"` parses
- [ ] `click "id" "url" "tooltip"` parses
- [ ] `click "id" "url" _blank` parses
- [ ] `click "id" "url" "tooltip" _blank` parses

---

## Phase 3: Renderer

### Task 3.1: Add data-commit-id Attribute
**File:** `packages/mermaid/src/diagrams/git/gitGraphRenderer.ts`

**Objective:** Add data attribute to commit elements for selection

**Implementation:**
Find commit group creation and add:
```typescript
commitGroup.attr('data-commit-id', commit.id);
```

**Acceptance:**
- [ ] Rendered commits have `data-commit-id` attribute
- [ ] Attribute value matches commit id

---

### Task 3.2: Implement Click Event Setup Function
**File:** `packages/mermaid/src/diagrams/git/gitGraphRenderer.ts`

**Objective:** Create `setupClickEvents()` function

**Implementation:**
1. Import `sanitizeUrl` from utils
2. Create function that:
   - Gets links from db
   - Returns early if no links
   - Returns a function that binds click events
   - For each link: select element, sanitize URL, add classes/attributes, bind click/keydown handlers
   - Respect securityLevel for navigation behavior
   - Use `noopener,noreferrer` for `_blank`

**Acceptance:**
- [ ] Returns undefined when no links
- [ ] Returns bind function when links exist
- [ ] Sanitizes URLs before use
- [ ] Handles sandbox mode via postMessage
- [ ] Keyboard accessible (Enter/Space)

---

### Task 3.3: Integrate Click Events into Draw Function
**File:** `packages/mermaid/src/diagrams/git/gitGraphRenderer.ts`

**Objective:** Return bindFunctions from draw()

**Implementation:**
Call `setupClickEvents()` and include result in return object:
```typescript
return {
  svg: svg.node(),
  bindFunctions,
};
```

**Acceptance:**
- [ ] draw() returns object with bindFunctions
- [ ] Existing rendering unaffected

---

## Phase 4: Styles

### Task 4.1: Add Clickable Commit Styles
**File:** `packages/mermaid/src/diagrams/git/styles.ts`

**Objective:** Add hover/focus styles for linked commits

**Implementation:**
Add to getStyles():
```css
.commit.clickable { cursor: pointer; }
.commit.clickable:hover .commit-label-bkg,
.commit.clickable:focus .commit-label-bkg { stroke-width: 4px; }
.commit.clickable:hover text.commit-label,
.commit.clickable:focus text.commit-label { text-decoration: underline; }
.commit.clickable:focus { outline: 2px solid ${options.git0 || '#000'}; outline-offset: 2px; }
```

**Acceptance:**
- [ ] Cursor changes on hover
- [ ] Visual feedback on hover/focus
- [ ] Focus indicator visible

---

## Phase 5: Testing

### Task 5.1: Unit Tests - Link State Management
**File:** `packages/mermaid/src/diagrams/git/__tests__/gitGraphAst.spec.ts`

**Objective:** Test setLink, getLink, getLinks, clear

**Tests:**
- setLink stores basic link
- Default target is _self
- Tooltip stored correctly
- Target stored correctly
- Overwrite existing link
- clear() removes all links
- commit with inline link creates entry

**Acceptance:**
- [ ] All tests pass
- [ ] Edge cases covered

---

### Task 5.2: Parser Tests - Link Syntax
**File:** `packages/mermaid/src/diagrams/git/__tests__/gitGraphParser.spec.ts`

**Objective:** Test all link syntax variations

**Tests:**
- Inline link parses
- Inline link + tooltip parses
- Inline link + target parses
- Inline link + tooltip + target parses
- click statement parses
- click + tooltip parses
- click + target parses
- click + tooltip + target parses

**Acceptance:**
- [ ] All syntax variations tested
- [ ] Tests pass

---

### Task 5.3: Visual Regression Tests
**File:** `cypress/integration/rendering/gitGraph.spec.js`

**Objective:** Add snapshot tests for linked commits

**Tests:**
- Commit with inline link renders
- Commit with click statement renders
- Branches with links render
- All orientations (LR, TB, BT) work
- Special characters in commit id handled

**Acceptance:**
- [ ] Snapshots generated
- [ ] Tests pass in CI

---

## Phase 6: Documentation

### Task 6.1: Update gitgraph.md Documentation
**File:** `docs/syntax/gitgraph.md`

**Objective:** Document link syntax

**Content:**
- Interactive Links section
- Inline syntax examples
- Click statement syntax examples
- Combined usage example
- Note about security (sandbox mode)

**Acceptance:**
- [ ] Examples render correctly
- [ ] All syntax variations documented

---

## Completion Checklist

### Functional
- [ ] Inline `link:` works
- [ ] Inline `tooltip:` works
- [ ] Inline `target:` works
- [ ] Click statement works
- [ ] Works on NORMAL/REVERSE/HIGHLIGHT commits
- [ ] Works on merge commits
- [ ] Works with all orientations
- [ ] Works with tags
- [ ] Click statement can override inline

### UI/UX
- [ ] Cursor pointer on linked commits
- [ ] Hover feedback
- [ ] Tooltip displays
- [ ] Keyboard accessible
- [ ] Focus indicator

### Security
- [ ] javascript: blocked
- [ ] data: blocked
- [ ] Sandbox uses postMessage
- [ ] _blank uses noopener,noreferrer

### Tests
- [ ] Unit tests pass
- [ ] Parser tests pass
- [ ] Cypress tests pass
- [ ] CI green

### Documentation
- [ ] Syntax documented
- [ ] Examples work
