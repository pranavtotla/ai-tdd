---
name: ai-tdd
version: 2.0.0
description: This skill should be used when the user asks to "build with TDD", "write tests first", "ai-tdd", "test-driven development", "implement with tests", "tests before code", "TDD workflow", "write failing tests first", "add e2e tests", "playwright tests", or wants AI to write tests before implementation code. Enforces a 5-step workflow (optionally 7 steps with E2E) where AI writes tests, human approves them, then AI writes code constrained by those tests.
---

# AI-TDD

## The Alignment Problem

When AI writes both tests and code, human intent enters the system at exactly two points: the initial requirement and the review. Everything between is the AI's interpretation.

If AI writes code first, then tests, the tests align to the code — not to the requirement. The AI tests what it built, not what was asked for. Edge cases it missed stay undetected because the tests are optimized to confirm the implementation, not challenge it.

If AI writes tests first, then code, the tests align to the requirement while the code aligns to the tests. The human reviews the tests — the specification — before any implementation exists. This is the only point in the workflow where the human can correct the AI's understanding of the requirement before it compounds into code.

The human review of tests is the alignment mechanism. Skip it and the AI is optimizing against its own interpretation in a closed loop.

## Workflow

```
Step 1       Step 2         Step 3          Step 4        Step 5
Plan    →  Write Tests  →  Review Tests  →  Write Code  →  Review Code
 AI          AI             HUMAN (gate)      AI            HUMAN (gate)

                              ↓ (optional, after Step 5 approval)

                    Step 6           Step 7
                  Write E2E     →  Review E2E
                    AI              HUMAN (gate)
```

Strict ordering. Steps 3, 5, and 7 are hard gates — do not proceed without explicit human approval.

Steps 6-7 are **optional** — offer them after Step 5 approval for features that benefit from end-to-end testing (UI features, user workflows, integration points). Skip for pure backend logic or utilities.

### Step 1: Plan

Identify affected files, existing test conventions, dependencies, and relevant data flow. Understand the requirement. Produce a plan covering:

- **What will be built** — components affected, approach, data flow
- **What will be tested** — test types (unit, integration, E2E), key scenarios, where test files go
- **What will not be built** — explicit scope boundary
- **E2E scope** — if applicable, identify user workflows that need E2E coverage

The plan makes the AI's interpretation visible before any code is written. Present it to the human. If the interpretation is wrong, this is the cheapest place to correct it.

**Adaptive chunking:** If the plan identifies 3 or more independent components, propose splitting into chunks. Each chunk cycles through Steps 2–5 (and optionally 6–7) independently. Present the proposed chunks and let the human decide. Default: chunk when components are independent, batch when tightly coupled.

### Step 2: Write Tests

Detect the project's test framework and conventions. Write tests following existing patterns.

Tests should read as a **specification of behavior** — what the system does, not how it does it. A test that describes internal implementation details is a brittle test that constrains the implementation without adding confidence.

Write tests for:
- Expected behavior under normal conditions
- Edge cases and boundary conditions
- Error states and failure modes

Every test for new behavior must fail — no implementation exists yet. If a test passes, it is testing existing behavior. Separate it from new-behavior tests and flag it during review so the human sees the distinction clearly.

New-behavior tests must fail for expected assertion reasons (wrong output, wrong state, missing side effect), not because the test harness is broken. Import errors, syntax errors, configuration errors, or missing setup are harness failures and must be fixed before Step 3.

Name each test as a plain-language statement of the behavior it verifies. The test name is the specification line item the human reviews at the gate — it must be self-explanatory without reading the test body. Each test should verify one behavior.

Do not write production implementation code in Step 2. Not app/runtime stubs, business-logic helpers, or product scaffolding. The moment implementation thinking enters the test-writing phase, tests start optimizing for ease of implementation rather than correctness of specification.

Test-only support code is allowed when needed to express behavior: fixtures, mocks, factories, and test-data setup. Keep this code under test paths and do not embed product logic in it.

### Step 3: Review Tests — HARD GATE

This is where alignment happens. The human is not reviewing "test quality" — they are reviewing whether the AI understood the requirement correctly. The tests are the specification. If the specification is wrong, the implementation will be precisely, confidently wrong.

Present the tests. Show what was written, what scenarios are covered, and confirm new-behavior tests fail for expected assertion reasons. If failures are due to harness/setup issues, fix those first and re-run before presenting. Then stop. See **`references/review-protocol.md`** for presentation format.

**Do not proceed until the human explicitly approves.**

On rejection: ask targeted questions about what to change. Do not guess at intent — ambiguity caused the problem, more ambiguity will not fix it. Propose revisions before making them. Re-present after revising.

After 3 revision rounds without resolution, the misalignment is deeper than test-level fixes can address. Suggest the human edit tests directly, pair on the issue, or revisit the plan.

### Step 4: Write Code

The tests are the contract. Implement code to satisfy them.

Run the approved tests to verify they pass, then run the broader project test suite to check for regressions. The approved tests define the scope — do not add behavior beyond what they require. If something is missing, it goes into a new cycle, not into this implementation.

If a test cannot be passed without modifying the test itself, **stop and flag it**. Explain why. The human approved the specification — changing it unilaterally defeats the purpose of the review gate. The human decides whether to adjust the test or the requirement.

#### UI Testability Requirements

For UI components, **add `data-testid` attributes** to all interactive and state-displaying elements. This is not optional — it is a testability contract that enables reliable E2E testing.

**Required locators:**
- Buttons, links, and clickable elements
- Input fields, dropdowns, and form controls
- Panels, modals, and containers that show/hide
- List items that need individual selection
- State indicators (loading, error, success)
- Dynamic content areas

**Naming convention:** `{component}-{element}-{qualifier}`
```tsx
// Examples
data-testid="space-tag-category-dropdown-DEPARTMENT"
data-testid="create-tag-apply-button"
data-testid="view-settings-color-mode-select"
data-testid="object-properties-panel"
```

**Do not rely on:**
- Text content (changes with i18n, updates)
- CSS classes (change with styling)
- DOM structure (fragile to refactoring)
- XPath (brittle, hard to maintain)

If implementing a feature that will need E2E tests, the `data-testid` attributes are part of the implementation — not an afterthought.

### Step 5: Review Code — HARD GATE

Present the implementation. Show test results, what was built, the diff in existing codebases, and any concerns. Then stop.

**Do not proceed until the human explicitly approves.**

On rejection: same collaborative loop as Step 3.

After approval, **offer E2E testing** if the feature involves:
- User-facing UI components
- Multi-step user workflows
- Integration with other features
- Visual state changes (color modes, visibility toggles)
- 2D/3D view interactions

Present: "Would you like to add E2E (Playwright) tests for this feature? This covers user workflows and visual regression."

---

## Optional E2E Testing (Steps 6-7)

### Step 6: Write E2E Tests

Write Playwright E2E tests following the project's existing patterns. See **`references/playwright-e2e.md`** for complete conventions.

#### Prerequisites: Repository Configuration

**First, gather the E2E test environment:**

```
═══ E2E CONFIGURATION ══════════════════════════════════

Before writing E2E tests, I need to know:

1. Playwright test repository location:
   [ ] Same repo (path: _______________)
   [ ] Separate repo (path: _______________)

2. Test file location pattern:
   Example: src/tests/[feature]/[feature].spec.ts

3. Existing test files to reference:
   (I'll explore these to match your patterns)

════════════════════════════════════════════════════════
```

**After receiving configuration:**
1. Explore the test repository structure
2. Read 2-3 existing test files to understand patterns
3. Identify POMs, fixtures, and helpers available
4. Note the test ID convention (e.g., `TC_FEATURE_XXX`)

#### MANDATORY: Read Existing Tests in the Same Area

**Before writing ANY new E2E test file, you MUST:**

1. **Identify the target folder** — Is this an existing folder or a new folder?
   - **Existing folder** → Read ALL test files in that folder to understand patterns
   - **New folder** → Read tests from a similar feature area as reference

2. **Read the primary reference test for the feature area:**
   - **Program Mode** → Read `programMode/basic.spec.ts` first — this is the canonical example for Program Mode test patterns
   - **Other feature areas** → Read `[feature]/basic.spec.ts` or the first test file in that folder

3. **Read common/shared files to avoid redundancy:**
   ```bash
   # Check for existing common methods
   ls -la [playwright-repo]/src/common/
   cat [playwright-repo]/src/common/project.ts
   cat [playwright-repo]/src/common/geometry.ts
   cat [playwright-repo]/src/common/locators.ts
   ```

4. **Before writing ANY helper function, check if it already exists:**
   ```bash
   # Search for existing helper methods
   grep -r "function methodName" [playwright-repo]/src/common/
   grep -r "async function" [playwright-repo]/src/common/*.ts
   ```

**Do NOT:**
- Write a new helper function if one already exists in common files
- Create patterns that conflict with existing tests in the same folder
- Ignore the established conventions from `basic.spec.ts` or similar reference files

**Present what you found:**
```
═══ EXISTING TEST ANALYSIS ═════════════════════════════

Target folder: [path/to/feature/]
Reference tests read:
  • [basic.spec.ts] — [patterns observed]
  • [other.spec.ts] — [patterns observed]

Common methods available:
  • ensureDrawMode() — from common/project.ts
  • setCameraPosition() — from common/camera.ts
  • clearSelection() — from common/geometry.ts
  • [list other relevant methods]

Patterns I will follow:
  • Camera position: [observed pattern]
  • Test ID format: [observed format]
  • beforeEach setup: [observed setup]
  • Helper location: [where helpers should go]

⏸ Confirm these patterns, or point me to different references.
════════════════════════════════════════════════════════
```

#### Check Existing Related Tests

Before writing new tests, find and run existing tests related to the feature:

```bash
# Find tests related to the feature
grep -r "feature-name" [playwright-repo]/src/tests/
npx playwright test --list | grep -i "feature"

# Run existing related tests
npx playwright test [related-test-files] --timeout=120000
```

**If existing tests fail due to your implementation changes:**

1. **Analyze the failure** — Is it a locator change, flow change, or behavior change?
2. **Categorize the update needed:**
   - **Locator update** — Element ID/testid changed
   - **Flow update** — New steps required, old steps removed
   - **Assertion update** — Expected behavior changed (intentional)
   - **New edge case** — Implementation revealed missing coverage

3. **Present updates for review:**

```
═══ EXISTING TEST UPDATES ══════════════════════════════

Affected tests: [count]
Files: [file paths]

Updates needed:
  • [TC_XXX_001] — Locator change: #old-id → #new-id
  • [TC_XXX_003] — Flow change: Added step for new modal
  • [TC_XXX_005] — Assertion change: Now expects 3 items instead of 2

Reason: [Brief explanation of why implementation caused these changes]

⏸ Approve updates, or tell me what to change.
════════════════════════════════════════════════════════
```

**Do not silently update existing tests.** Present all changes for review — the human approved those tests previously and must approve modifications.

#### Prerequisites: Verify Locators Exist

Before writing tests, verify the UI components have `data-testid` attributes on:
- Buttons, dropdowns, and interactive elements
- Form inputs and controls
- Panels, modals, and containers
- List items and selection targets

**If locators are missing — STOP.** Do not use workarounds (XPath, text matching, CSS structure). Instead:

1. Report the missing locator with suggested `data-testid`
2. Request the human add it to the component
3. Resume E2E test writing after locator is added

See `references/playwright-e2e.md` → "Handling Missing Locators" for the reporting format.

#### E2E Test Structure

1. **Explore existing E2E tests** in the playwrighttesting repository
2. **Identify patterns** — POMs, fixtures, helpers, test file structure
3. **Verify required locators exist** — stop and request if missing
4. **Write exhaustive E2E tests** covering:

**Happy Paths:**
- Basic user workflow works end-to-end
- UI reflects all state changes correctly
- Data persists across operations

**Edge Cases:**
- Empty states (no data)
- Single item vs multiple items
- Maximum items / overflow (collapsed sections)
- Special characters and unicode in inputs
- Very long strings (truncation)

**View Modes:**
- 2D view behavior
- 3D view behavior
- Switching between views mid-workflow

**Panel/Modal States:**
- Panels open vs closed
- Collapsed vs expanded sections
- Modal interruptions (unsaved changes, confirmations)
- Dropdown overflow handling

**Selection States:**
- Single selection
- Multi-selection
- No selection
- Mixed selection (varies state)

**Integration Points:**
- Feature interaction with other features
- Right-click context menus (RCCM)
- Toolbar and panel coordination

#### E2E Test File Template

```typescript
import { expect, test } from '../../common/fixtures';
import { ensurePropertiesPanelIsOpen, ensureDrawMode, toggle2D, toggle3D } from '../../common/project';
import { setCameraPosition } from '../../common/camera';
import { space, floor, wall } from '../helper/bimObject';
import { ensureViewsTabDocked } from '../../common/locators';
import * as allure from 'allure-js-commons';
import { clearSelection } from '../../common/geometry';

const camPos2D = { /* camera config */ };

// Helpers at top
function generateRandomString(length: number): string { /* ... */ }

async function expandSectionIfNeeded(page: any) {
  const showAllButton = page.locator('text=/Show all \\d+ items/');
  if (await showAllButton.isVisible({ timeout: 2000 }).catch(() => false)) {
    await showAllButton.click();
    await page.waitForTimeout(300);
  }
}

async function dismissModalIfPresent(page: any) {
  const modal = page.getByText('modal text');
  if (await modal.isVisible({ timeout: 2000 }).catch(() => false)) {
    await page.getByRole('button', { name: 'Cancel' }).click();
    await page.waitForTimeout(300);
  }
}

test.beforeEach(async ({ page }) => {
  await setCameraPosition(page, camPos2D);
});

/**
 * @id TC_FEATURE_XXX
 * @description Description of test
 *
 * @steps
 * 1) Step one
 * 2) Step two
 *
 * @expected Expected outcome
 */
test('TC_FEATURE_XXX', async ({ page }) => {
  await allure.description(`Test Description: ...`);
  // Test implementation
});
```

#### Locator Best Practices

1. **Prefer stable selectors:**
   - `page.getByTestId('element-id')` — best
   - `page.locator('#element-id')` — good
   - `page.getByRole('button', { name: 'Submit' })` — good
   - `page.getByText('Label', { exact: true })` — acceptable
   - XPath — last resort

2. **Scope locators to containers:**
   ```typescript
   // Avoid ambiguous matches
   const dropdown = page.getByTestId('category-dropdown-listbox');
   await dropdown.getByRole('button', { name: 'Add new' }).click();
   ```

3. **Wait for elements before interaction:**
   ```typescript
   await element.waitFor({ state: 'visible', timeout: 10000 });
   await element.click();
   ```

#### POM Updates

If new UI elements need testing, update or create Page Object Models in `src/common/pom/`:

```typescript
// New locators for feature
this.newElement = page.getByTestId('new-element-id');
this.newButton = page.locator('#new-button');
```

### Step 7: Review E2E Tests — HARD GATE

Present the E2E tests. Show:
- New tests: count and coverage breakdown
- Updated tests: what changed and why
- All test scenarios as plain descriptions
- Test execution results
- Any POM updates made
- Snapshot status (new vs updated)

```
═══ E2E TEST REVIEW ════════════════════════════════════

NEW TESTS: [count]
Files: [file paths]

Scenarios:
  • [TC_XXX_001] [description of user workflow tested]
  • [TC_XXX_002] [description of edge case tested]
  • ...

Coverage:
  ✓ Happy path workflows
  ✓ 2D view behavior
  ✓ 3D view behavior
  ✓ Edge cases: [list]
  ✓ Modal/panel states

────────────────────────────────────────────────────────

UPDATED EXISTING TESTS: [count] (or "none")
Files: [file paths]

Changes:
  • [TC_YYY_001] — Locator: #old-id → #new-id
  • [TC_YYY_003] — Added wait for new modal
  • [TC_YYY_005] — Updated assertion: expects 3 items (was 2)

Reason: [Why implementation required these updates]

────────────────────────────────────────────────────────

Results: [X/X passing] (new + updated)
Command: npx playwright test [files]

POM Updates: [none / list of changes]
Snapshots: [X new / Y updated]

⏸ Approve, or tell me what to change.
════════════════════════════════════════════════════════
```

**Do not proceed until the human explicitly approves.**

#### Running E2E Tests

After approval, offer to run:

1. **Functional verification:**
   ```bash
   npx playwright test [test-file] --timeout=120000
   ```

2. **Snapshot update (if visual tests):**
   ```bash
   npx playwright test [test-file] --update-snapshots
   ```

3. **Debug mode (if failures):**
   ```bash
   PWDEBUG=1 npx playwright test --grep "TC_XXX"
   ```

#### Updating Existing E2E Tests

If implementation changes break existing E2E tests:

1. **Identify affected tests** — run full E2E suite
2. **Analyze failures** — locator changes, behavior changes, timing issues
3. **Propose updates:**
   - Locator fixes (element IDs changed)
   - Flow changes (new steps required)
   - Snapshot updates (visual changes)
4. **Present changes for review** before applying

---

## Edge Cases

**New feature (clean slate)** — Run the standard cycle without exceptions: plan the scope, write failing tests for new behavior, pass Step 3 review, implement only what approved tests require, run approved tests plus broader regression checks, then pass Step 5 review. Optionally continue to Steps 6-7 for E2E.

**Chunked delivery (independent components)** — If chunking is approved in Step 1, each chunk must complete a full Step 2 → Step 5 (→ Step 7) cycle before starting the next chunk. Do not batch multiple unreviewed chunks into a single gate.

**Existing behavior already covered** — Treat already-passing tests as baseline coverage, then add failing tests only for missing behavior in scope. Present both groups clearly at Step 3 so the human can distinguish baseline vs new specification.

**Bug fixes** — Same workflow. Plan describes the bug. Tests reproduce it (they fail against current code). Code fixes it. Tests go green. E2E tests verify the fix in context.

**Refactoring** — Write tests that lock current behavior. Approve them through Step 3. Refactor in Step 4. Tests stay green. If behavior needs to change, that is a feature — start a new cycle.

**Existing code with no tests** — Plan identifies what existing behavior needs coverage. Step 2 writes tests for existing behavior alongside new behavior. Separate the two groups clearly when presenting for review.

**Test infrastructure** — If tests need databases, mocks, or fixtures, identify this in the plan and set up infrastructure before writing test cases.

**Tests that cannot be passed** — Do not silently modify approved tests. Stop. Flag. The human decides.

**E2E test failures after code changes** — If implementation changes break E2E tests, do not silently fix them. Present the failures and proposed fixes for review.

**Snapshot mismatches** — Visual regressions may be intentional (feature change) or bugs. Present both the old and new state. Human decides whether to update snapshots or fix the code.

**Missing locators for E2E** — If UI elements lack `data-testid` attributes, do not write hacky workarounds. Stop and request the locator be added to the component. Resume after it's added. This ensures tests remain stable.

## Anti-Patterns

**Writing tests without reading existing tests in the same folder.** Before adding `tagsCrud.spec.ts` to `programMode/`, you must first read `programMode/basic.spec.ts` and other tests in that folder. Skipping this results in inconsistent patterns, conflicting conventions, and duplicated code.

**Writing redundant helper methods.** Before creating any helper function, search the `common/` directory to check if it already exists. Duplicating `ensureDrawMode()`, `setCameraPosition()`, or similar utilities wastes effort and creates maintenance burden. Always reuse existing common methods.

**Code before tests.** The core violation. Discard AI-authored implementation code created in the current cycle and restart from Step 2. Do not delete or rewrite pre-existing user code or repository history. Code-first implementation cannot be kept as "reference" — its existence biases test writing toward confirming what was already built.

**Tests that pass immediately.** Either testing existing behavior or testing nothing. Investigate before proceeding.

**Tests that describe implementation.** A test asserting that a specific internal method is called, or that data flows through a specific path, is not a specification — it is a constraint on implementation. Tests should describe observable behavior: given this input, expect this output or side effect.

**Exceeding test scope.** The approved tests define what to build. Additional features, optimizations, or "improvements" go into a new cycle. The human approved a specific scope — respect it.

**Modifying approved tests without review.** Tests are the contract the human signed off on. Changing them without going back through the gate breaks the alignment mechanism.

**Testing trivial operations.** Tests for getters, identity mappings, or framework defaults add maintenance cost without adding confidence. Focus on behavior that could plausibly be wrong.

**Flaky E2E tests.** E2E tests that fail intermittently due to timing issues are worse than no tests — they erode trust. Use proper waits, not arbitrary timeouts. Debug flakiness before approving.

**Hardcoded coordinates without context.** Canvas click coordinates should be documented or use relative positioning when possible. Magic numbers make tests brittle.

**Workaround locators instead of requesting testids.** When `data-testid` is missing, do not use XPath, text matching, or CSS structure as workarounds. These are fragile — tooltips match text, DOM structure changes, classes get renamed. Stop and request the locator be added. The small delay is worth the test stability.

## The Rule

```
Tests are the specification. Human-approved.
Code fulfills the specification. Nothing more.
E2E tests verify the user experience. Exhaustively.
Gates are not optional.
```

## References

- **`references/review-protocol.md`** — Presentation format for review gates and the collaborative fix loop
- **`references/playwright-e2e.md`** — Playwright E2E testing patterns, conventions, and exhaustiveness checklist
