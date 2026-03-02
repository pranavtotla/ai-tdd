# Playwright E2E Testing Reference

Patterns and conventions for writing exhaustive Playwright E2E tests.

## Getting Started: Repository Discovery

Before writing E2E tests, you must know where the Playwright tests live. Ask the user:

```
Where is your Playwright test repository located?
- Same repository as the main codebase? (provide path, e.g., tests/e2e/)
- Separate repository? (provide full path)
```

Once you have the path, explore it to understand:
1. **Directory structure** — where test files go
2. **Existing patterns** — read 2-3 test files to understand conventions
3. **Available helpers** — POMs, fixtures, utilities
4. **Test ID convention** — how tests are named (e.g., `TC_FEATURE_XXX`)

### Discovery Commands

```bash
# Explore test structure
ls -la [playwright-repo]/src/tests/

# Find existing test files for reference
find [playwright-repo] -name "*.spec.ts" | head -10

# Check for POMs
ls -la [playwright-repo]/src/common/pom/

# Read a sample test to understand patterns
cat [playwright-repo]/src/tests/[feature]/[feature].spec.ts | head -100
```

## Repository Structure (Example)

This is a typical Playwright test repository structure. Your project may differ — adapt accordingly.

```
playwrighttesting/
├── src/
│   ├── common/
│   │   ├── fixtures.ts          # Test fixtures and custom matchers
│   │   ├── page.ts              # Extended Page class with helpers
│   │   ├── canvas.ts            # Canvas interaction helpers
│   │   ├── project.ts           # Project initialization helpers
│   │   ├── camera.ts            # Camera position helpers
│   │   ├── geometry.ts          # Geometry helpers
│   │   ├── locators.ts          # Common locator helpers
│   │   └── pom/                 # Page Object Models
│   │       ├── opp.ts           # Object Properties Panel
│   │       ├── tags.ts          # Tags functionality
│   │       ├── views.ts         # Views panel
│   │       └── ...
│   └── tests/
│       ├── helper/
│       │   └── bimObject.ts     # BIM object drawing helpers
│       └── [feature]/
│           └── [feature].spec.ts
└── testData/
    └── [feature]/
        └── snapshots/           # Visual regression snapshots
```

## Test File Structure

```typescript
import { expect, test } from '../../common/fixtures';
import { ensurePropertiesPanelIsOpen, ensureDrawMode } from '../../common/project';
import { setCameraPosition } from '../../common/camera';
import { space, floor, wall } from '../helper/bimObject';
import { ensureViewsTabDocked } from '../../common/locators';
import * as allure from 'allure-js-commons';
import { clearSelection } from '../../common/geometry';

// Camera position for consistent 2D view
const camPos2D = {
  position: { _x: 80, _y: 904.4335243676011, _z: -40.09044335243676 },
  alpha: -1.5707963267948966,
  beta: 0,
  radius: 904.4335243676011,
  target: { _x: 80, _y: 0, _z: -40 },
  isOrtho: true,
  orthoLeft: -72.40773439350247,
  orthoRight: 72.40773439350247,
  orthoBottom: -40.72935059634514,
  orthoTop: 40.72935059634514
};

// Camera position for consistent 3D view
const camPos3D = {
  position: { _x: 150, _y: 200, _z: 150 },
  alpha: -0.7853981633974483,
  beta: 1.0471975511965976,
  radius: 300,
  target: { _x: 0, _y: 0, _z: 0 },
  isOrtho: false
};

// Helper functions at top of file
function generateRandomString(length: number): string {
  const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
  let result = '';
  for (let i = 0; i < length; i++) {
    result += characters.charAt(Math.floor(Math.random() * characters.length));
  }
  return result;
}

test.beforeEach(async ({ page }) => {
  await setCameraPosition(page, camPos2D);
});

/**
 * @id TC_FEATURE_XXX
 * @description Brief description of what this test verifies
 *
 * @steps
 * 1) Step one
 * 2) Step two
 * 3) Step three
 *
 * @expected Expected outcome
 */
test('TC_FEATURE_XXX', async ({ page }) => {
  await allure.description(`Test Description: ...

Steps to reproduce:
  1) ...
  2) ...

Expected outcome:
  ...
`);

  // Test implementation
});
```

## Test ID Conventions

- Format: `TC_[FEATURE]_[SUBFEATURE]_[NUMBER]`
- Examples:
  - `TC_TAGS_CM_001` — Tags Color Mode test 1
  - `TC_TAGS_FL_003` — Tags Flooring test 3
  - `TC_TAGS_FS_002` — Tags Filter Selection test 2

## Page Object Models (POM)

### Using POMs
```typescript
// Access via page object
await page.opp.spaceDepartment.departmentDropDown.click();
await page.opp.spaceDepartment.addNew.click();
await page.tags.activeTagInput.fill('TagName');
await page.views.viewSettings.click();
```

### Creating/Updating POMs
POMs live in `src/common/pom/`. Each POM class:
- Takes `page: Page` in constructor
- Defines locators as readonly properties
- Uses stable selectors (testids > IDs > text > CSS > XPath)

```typescript
// Example POM structure
export class SpaceDepartment {
  readonly departmentDropDown: Locator;
  readonly addNew: Locator;
  readonly applyButton: Locator;

  constructor(page: Page) {
    this.departmentDropDown = page
      .getByTestId('space-tag-category-dropdown-DEPARTMENT')
      .locator('svg');
    this.addNew = page
      .getByTestId('space-tag-category-dropdown-DEPARTMENT-listbox')
      .getByRole('button', { name: 'Add new' });
    this.applyButton = page.getByTestId('create-or-edit-tag-apply-button');
  }
}
```

## Locator Priority (Most to Least Preferred)

1. **data-testid** — `page.getByTestId('element-id')`
2. **ID attribute** — `page.locator('#element-id')`
3. **Role + name** — `page.getByRole('button', { name: 'Submit' })`
4. **Text (exact)** — `page.getByText('Label', { exact: true })`
5. **CSS selector** — `page.locator('.class-name')`
6. **XPath** — `page.locator("//div[text()='Label']")` (last resort)

### Scoping Locators
Always scope locators to avoid ambiguity:
```typescript
// Bad - might match multiple elements
await page.getByRole('button', { name: 'Add new' }).click();

// Good - scoped to specific container
const dropdown = page.getByTestId('category-dropdown-listbox');
await dropdown.getByRole('button', { name: 'Add new' }).click();
```

## Handling Missing Locators

When writing E2E tests and a required `data-testid` is missing from the UI component:

### DO NOT use workarounds like:
```typescript
// Bad - fragile text matching
await page.getByText('Select...').click();  // Matches tooltips too!

// Bad - brittle XPath
await page.locator("//div[contains(@class, 'dropdown')]//button").click();

// Bad - unstable CSS structure
await page.locator('.panel > div:nth-child(3) > button').click();
```

### DO request the locator be added:

**Stop and report the missing locator:**

```
═══ MISSING LOCATOR ════════════════════════════════════

Element: [description of element]
Component: [file path if known]
Current state: No data-testid attribute

Suggested testid: [component]-[element]-[qualifier]
Example: "color-mode-dropdown-trigger"

Action needed: Add data-testid to the component before
continuing E2E test development.

════════════════════════════════════════════════════════
```

### Locator Request Template

When requesting a locator be added to a React component:

```tsx
// Before (not testable)
<button onClick={handleClick}>
  Submit
</button>

// After (testable)
<button
  data-testid="form-submit-button"
  onClick={handleClick}
>
  Submit
</button>
```

### Checklist for UI Components

Before E2E tests can be written, verify these elements have `data-testid`:

- [ ] All buttons and clickable elements
- [ ] Form inputs (text, select, checkbox, radio)
- [ ] Dropdown triggers AND dropdown lists/options
- [ ] Modal/dialog containers and action buttons
- [ ] Panel headers and toggle controls
- [ ] List items that need individual selection
- [ ] Tab controls and tab panels
- [ ] Loading/error/empty state indicators
- [ ] Navigation elements

### When Locators Cannot Be Added

If the component is in a third-party library or cannot be modified:

1. **Use role-based locators** with proper scoping:
   ```typescript
   const modal = page.getByRole('dialog');
   await modal.getByRole('button', { name: 'Confirm' }).click();
   ```

2. **Use aria-label** if available:
   ```typescript
   await page.getByLabel('Close dialog').click();
   ```

3. **Document the workaround** with a comment explaining why:
   ```typescript
   // Third-party modal component - no testid available
   // Using role-based selector as fallback
   await page.getByRole('dialog').getByRole('button', { name: 'OK' }).click();
   ```

## 2D/3D View Handling

### Setting Camera Position
```typescript
test.beforeEach(async ({ page }) => {
  await setCameraPosition(page, camPos2D); // or camPos3D
});
```

### Toggling Views
```typescript
import { toggle2D, toggle3D, is2D } from '../../common/project';

// Switch to 2D
await toggle2D(page);

// Switch to 3D
await toggle3D(page);

// Check current mode
const isTwoD = await is2D(page);
```

### View-Specific Tests
For features that behave differently in 2D vs 3D:
```typescript
test('TC_FEATURE_001_2D', async ({ page }) => {
  await toggle2D(page);
  // Test 2D-specific behavior
});

test('TC_FEATURE_001_3D', async ({ page }) => {
  await toggle3D(page);
  // Test 3D-specific behavior
});
```

## Common Edge Cases to Cover

### Modal/Dialog Handling
```typescript
// Dismiss unsaved changes modal if it appears
const modal = page.getByText('You have unsaved changes');
if (await modal.isVisible({ timeout: 2000 }).catch(() => false)) {
  await page.getByRole('button', { name: 'Cancel' }).click();
  await page.waitForTimeout(300);
}
```

### Dropdown/Overflow Handling
```typescript
// Expand collapsed sections
const showAllButton = page.locator('text=/Show all \\d+ tags/');
if (await showAllButton.isVisible({ timeout: 2000 }).catch(() => false)) {
  await showAllButton.click();
  await page.waitForTimeout(300);
}
```

### Canvas Interactions
```typescript
// Drawing objects
await page.canvas.click(x1, y1);
await page.canvas.click(x2, y2);

// Right-click context menu
await page.canvas.click(x, y, { button: 'right' });

// Selection
await page.canvas.click(centerX, centerY);
await clearSelection(page);
```

### Panel State
```typescript
// Ensure panels are open/closed
await ensurePropertiesPanelIsOpen(page);
await ensureViewsTabDocked(page);
await ensureDrawMode(page);
```

### Wait Patterns
```typescript
// Wait for element
await page.waitForSelector('#element', { state: 'visible', timeout: 20000 });

// Wait for element to disappear
await page.locator('.toast').waitFor({ state: 'detached' });

// Fixed timeout (use sparingly)
await page.waitForTimeout(300);

// Wait for element before interaction
await element.waitFor({ state: 'visible', timeout: 10000 });
await element.click();
```

## Visual Regression (Snapshots)

### Taking Snapshots
```typescript
// Canvas snapshot
await expect(page).toHaveCanvasSnapshot('tc_feature_001.png', { maxDiffPixels: 960 });

// Full page screenshot (for debugging)
await page.screenshot({ path: 'debug.png' });
```

### Updating Snapshots
```bash
# Update all snapshots
npx playwright test [test-file] --update-snapshots

# Run specific test with snapshot update
npx playwright test --grep "TC_FEATURE_001" --update-snapshots
```

## Error Patterns and Fixes

### Tooltip/Canvas Intercepts Pointer
Use ID selectors instead of text:
```typescript
// Bad - might match tooltip
const option = page.getByText('Select...');

// Good - uses element ID
const option = page.locator('#selectMenu');
```

### Strict Mode Violation
Scope the locator:
```typescript
// Bad - matches multiple
await page.getByRole('button', { name: 'Add new' }).click();

// Good - scoped
await container.getByRole('button', { name: 'Add new' }).click();
```

### Element Not Found After Navigation
Wait for element visibility:
```typescript
await element.waitFor({ state: 'visible', timeout: 10000 });
await element.click();
```

### Submenu Navigation
Click (not hover) for multi-level menus:
```typescript
// Open submenu by clicking
await page.locator('#selectByTag').click();
await page.waitForTimeout(300);

// Then select option
await page.getByText('Department', { exact: true }).click();
```

## Test Exhaustiveness Checklist

For each feature, cover:

### Happy Path
- [ ] Basic functionality works as expected
- [ ] UI reflects changes correctly
- [ ] Data persists after actions

### Edge Cases
- [ ] Empty states (no data)
- [ ] Single item
- [ ] Maximum items / overflow
- [ ] Special characters in input
- [ ] Very long strings
- [ ] Unicode/emoji

### View Modes
- [ ] 2D view behavior
- [ ] 3D view behavior
- [ ] Switching between views

### Panel States
- [ ] Panel open vs closed
- [ ] Panel collapsed vs expanded
- [ ] Multiple panels interaction

### Selection States
- [ ] Single selection
- [ ] Multi-selection
- [ ] No selection
- [ ] Mixed selection (varies state)

### Error States
- [ ] Invalid input
- [ ] Network errors (if applicable)
- [ ] Concurrent operations

### Integration
- [ ] Interaction with other features
- [ ] State persistence across operations
- [ ] Undo/redo behavior (if applicable)

## Updating Existing Tests

When implementing a feature, existing tests may break. Follow this process:

### 1. Find Related Tests
```bash
# Search for tests mentioning the feature
grep -r "feature-name\|FeatureName" [playwright-repo]/src/tests/

# List tests by file
npx playwright test --list | grep -i "feature"

# Run related tests to check for failures
npx playwright test src/tests/[feature]/ --timeout=120000
```

### 2. Categorize Failures

| Failure Type | Cause | Fix |
|--------------|-------|-----|
| **Locator not found** | Element ID/testid changed | Update locator in test or POM |
| **Strict mode violation** | New element matches same locator | Scope locator to container |
| **Timeout** | New modal/panel blocking | Add dismissal helper |
| **Assertion failed** | Behavior intentionally changed | Update expected value |
| **Flow broken** | New required step | Add step to test |

### 3. Common Update Patterns

**Locator changed:**
```typescript
// Before
await page.locator('#old-element-id').click();

// After
await page.locator('#new-element-id').click();
```

**New modal interrupts flow:**
```typescript
// Add helper at top of file
async function dismissNewModal(page: any) {
  const modal = page.getByText('New Modal Title');
  if (await modal.isVisible({ timeout: 2000 }).catch(() => false)) {
    await page.getByRole('button', { name: 'Close' }).click();
    await page.waitForTimeout(300);
  }
}

// Call in test where modal might appear
await dismissNewModal(page);
await page.canvas.click(x, y);
```

**Element moved to different container:**
```typescript
// Before - element was in panel A
await page.getByTestId('panel-a').getByRole('button', { name: 'Action' }).click();

// After - element moved to panel B
await page.getByTestId('panel-b').getByRole('button', { name: 'Action' }).click();
```

**Assertion value changed:**
```typescript
// Before - feature showed 2 items
await expect(page.locator('.item')).toHaveCount(2);

// After - feature now shows 3 items (intentional)
await expect(page.locator('.item')).toHaveCount(3);
```

### 4. Update POM If Needed

If multiple tests use the same broken locator, update the POM:

```typescript
// In src/common/pom/feature.ts
// Before
this.actionButton = page.locator('#old-action-btn');

// After
this.actionButton = page.getByTestId('new-action-button');
```

### 5. Present All Updates for Review

Never silently update existing tests. Present:
- Which tests were updated
- What changed in each
- Why the change was necessary

## Running Tests

```bash
# Run all tests in a file
npx playwright test src/tests/tags/tagsColorMode.spec.ts

# Run specific test by name
npx playwright test --grep "TC_TAGS_CM_001"

# Run with debug UI
npx playwright test --debug

# Run with headed browser
npx playwright test --headed

# Update snapshots
npx playwright test --update-snapshots

# Run against specific project
npx playwright test --project=chromium
```

## Debugging

```bash
# Debug mode with Playwright Inspector
PWDEBUG=1 npx playwright test --grep "TC_FEATURE_001"

# Take screenshot on failure (default)
# Screenshots saved to test-results/

# Check test-results folder for:
# - Screenshots at failure point
# - Trace files (if enabled)
# - Video recordings (if enabled)
```
