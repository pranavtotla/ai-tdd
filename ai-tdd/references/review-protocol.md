# Review Protocol

Presentation format for review gates (Steps 3 and 5) and the collaborative fix loop.

## Principle

Present what was done. Do not guide the review. Prescriptive checklists can bias the review toward what the AI considers important, which may not match what the human needs to evaluate. Present the facts. The human brings their judgment.

## Test Review (Step 3)

### Template

```
═══ TEST REVIEW ════════════════════════════════════════

Tests: [count] ([breakdown by type])
Files: [file paths]

Scenarios:
  • [plain description of what each test verifies]
  • ...

Failing: [all / N of M — explain any that pass]

⏸ Approve, or tell me what to change.
════════════════════════════════════════════════════════
```

### Example

```
═══ TEST REVIEW ════════════════════════════════════════

Tests: 7 (5 unit, 2 integration)
Files: src/__tests__/auth.test.ts, src/__tests__/auth.integration.test.ts

Scenarios:
  • valid email and password returns session token
  • missing email returns validation error
  • wrong password returns authentication error
  • account locked after 5 consecutive failed attempts
  • locked account cannot authenticate even with correct password
  • session token expires after 30 minutes of inactivity
  • concurrent login from second device invalidates first session

Failing: all 7

⏸ Approve, or tell me what to change.
════════════════════════════════════════════════════════
```

### Guidelines

Include:
- Count and type breakdown (unit, integration, E2E)
- File paths
- Every scenario as a plain description of behavior being verified
- Confirmation that new-behavior tests fail, with explanation for any that pass (existing behavior)

Do not include opinions on coverage, quality, or completeness.

## Code Review (Step 5)

### Template

```
═══ CODE REVIEW ════════════════════════════════════════

Summary: [what was built, in one or two sentences]

Files:
  • [path] — [what changed]
  • ...

Tests: [X/X passing]
Regressions: [none / details]

[Diff summary if modifying existing code]

[Concerns, if any — stated plainly]

⏸ Approve, or tell me what to change.
════════════════════════════════════════════════════════
```

### Example

```
═══ CODE REVIEW ════════════════════════════════════════

Summary: Implemented email/password authentication with
session management and account lockout.

Files:
  • src/auth/login.ts — new; handles credential validation and session creation
  • src/auth/session.ts — new; session token generation, expiry, and invalidation
  • src/auth/lockout.ts — new; tracks failed attempts, locks after 5
  • src/middleware/auth.ts — modified; added session validation middleware
  • src/db/migrations/003_sessions.sql — new; sessions and lockout tables

Tests: 7/7 passing
Regressions: none (existing 142 tests still passing)

Concern: lockout counter resets on server restart since it is
stored in-memory. May want to persist to database in a follow-up.

⏸ Approve, or tell me what to change.
════════════════════════════════════════════════════════
```

### Guidelines

Include:
- Implementation summary
- Files with brief change descriptions
- Test results with exact pass count
- Regression check results
- Diff summary in existing codebases
- Honest concerns if they exist

Do not include self-assessment of quality or guided review prompts.

## Collaborative Fix Loop

When the human rejects or requests changes:

1. **Acknowledge** the specific feedback
2. **Ask targeted questions** if anything is ambiguous — do not interpret ambiguity, resolve it
3. **Propose the revision** before making it — state what will change and why
4. **Revise and re-present** using the same format

### Escalation

After 3 rounds on the same gate without resolution:

- Suggest the human edit directly and signal when done
- Suggest pairing on the specific issue
- Suggest revisiting the plan if the disagreement is about scope

Three rounds without resolution means the misalignment is structural. More revision cycles will not fix a problem that exists at the requirements or design level.

## Chunking Proposal

When the plan identifies multiple independent components:

```
═══ CHUNKING PROPOSAL ══════════════════════════════════

Components identified: [N]

Proposed chunks:
  1. [name] — [scope]
  2. [name] — [scope]
  3. [name] — [scope]

Order: [rationale]

Each chunk cycles: Write Tests → Review → Write Code → Review

⏸ Approve chunking, adjust grouping, or batch all.
════════════════════════════════════════════════════════
```
