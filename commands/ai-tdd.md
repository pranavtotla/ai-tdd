---
name: ai-tdd
description: Start an AI-TDD workflow — write tests first, then implementation, optionally add E2E tests
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
  - AskUserQuestion
  - Task
---

Start the AI-TDD workflow for the user's request. Follow the 5-step process (optionally 7 steps with E2E):

**Core workflow:** Plan → Write Tests → Review Tests (hard gate) → Write Code → Review Code (hard gate)

**Optional E2E:** Write E2E Tests → Review E2E (hard gate)

Read the skill at ${CLAUDE_PLUGIN_ROOT}/skills/ai-tdd/SKILL.md for the full workflow. Follow it exactly.

For E2E testing patterns, see ${CLAUDE_PLUGIN_ROOT}/skills/ai-tdd/references/playwright-e2e.md.

**Triggers for E2E flow:**
- User mentions "e2e tests", "playwright tests", "end-to-end tests"
- User mentions "2d/3d testing", "visual tests", "snapshot tests"
- After Step 5 approval for UI features — offer E2E testing
