---
name: ai-tdd
version: 1.0.0
description: This skill should be used when the user asks to "build with TDD", "write tests first", "ai-tdd", "test-driven development", "implement with tests", "tests before code", "TDD workflow", "write failing tests first", or wants AI to write tests before implementation code. Enforces a 5-step workflow where AI writes tests, human approves them, then AI writes code constrained by those tests.
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
```

Strict ordering. Steps 3 and 5 are hard gates — do not proceed without explicit human approval.

### Step 1: Plan

Identify affected files, existing test conventions, dependencies, and relevant data flow. Understand the requirement. Produce a plan covering:

- **What will be built** — components affected, approach, data flow
- **What will be tested** — test types (unit, integration, E2E), key scenarios, where test files go
- **What will not be built** — explicit scope boundary

The plan makes the AI's interpretation visible before any code is written. Present it to the human. If the interpretation is wrong, this is the cheapest place to correct it.

**Adaptive chunking:** If the plan identifies 3 or more independent components, propose splitting into chunks. Each chunk cycles through Steps 2–5 independently. Present the proposed chunks and let the human decide. Default: chunk when components are independent, batch when tightly coupled.

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

### Step 5: Review Code — HARD GATE

Present the implementation. Show test results, what was built, the diff in existing codebases, and any concerns. Then stop.

**Do not proceed until the human explicitly approves.**

On rejection: same collaborative loop as Step 3.

## Edge Cases

**Bug fixes** — Same workflow. Plan describes the bug. Tests reproduce it (they fail against current code). Code fixes it. Tests go green.

**Refactoring** — Write tests that lock current behavior. Approve them through Step 3. Refactor in Step 4. Tests stay green. If behavior needs to change, that is a feature — start a new cycle.

**Existing code with no tests** — Plan identifies what existing behavior needs coverage. Step 2 writes tests for existing behavior alongside new behavior. Separate the two groups clearly when presenting for review.

**Test infrastructure** — If tests need databases, mocks, or fixtures, identify this in the plan and set up infrastructure before writing test cases.

**Tests that cannot be passed** — Do not silently modify approved tests. Stop. Flag. The human decides.

## Anti-Patterns

**Code before tests.** The core violation. Discard AI-authored implementation code created in the current cycle and restart from Step 2. Do not delete or rewrite pre-existing user code or repository history. Code-first implementation cannot be kept as "reference" — its existence biases test writing toward confirming what was already built.

**Tests that pass immediately.** Either testing existing behavior or testing nothing. Investigate before proceeding.

**Tests that describe implementation.** A test asserting that a specific internal method is called, or that data flows through a specific path, is not a specification — it is a constraint on implementation. Tests should describe observable behavior: given this input, expect this output or side effect.

**Exceeding test scope.** The approved tests define what to build. Additional features, optimizations, or "improvements" go into a new cycle. The human approved a specific scope — respect it.

**Modifying approved tests without review.** Tests are the contract the human signed off on. Changing them without going back through the gate breaks the alignment mechanism.

**Testing trivial operations.** Tests for getters, identity mappings, or framework defaults add maintenance cost without adding confidence. Focus on behavior that could plausibly be wrong.

## The Rule

```
Tests are the specification. Human-approved.
Code fulfills the specification. Nothing more.
Gates are not optional.
```

## References

- **`references/review-protocol.md`** — Presentation format for review gates and the collaborative fix loop
