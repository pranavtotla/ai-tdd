# AI-TDD

AI-driven Test-Driven Development. A skill for AI coding agents.

## The Problem

When AI writes code first, then tests, the tests confirm what was built — not what was asked for. The AI's reward function becomes "pass the code," and edge cases it missed stay undetected.

When AI writes tests first, then code, the reward function flips. Code optimizes for passing the tests. The human reviews the tests — the specification — before any implementation exists. This is the only point where human intent enters the loop before it compounds into code.

## The Workflow

```
Step 1       Step 2         Step 3          Step 4        Step 5
Plan    →  Write Tests  →  Review Tests  →  Write Code  →  Review Code
 AI          AI             HUMAN (gate)      AI            HUMAN (gate)

                              ↓ (optional, after Step 5 approval)

                    Step 6           Step 7
                  Write E2E     →  Review E2E
                    AI              HUMAN (gate)
```

1. **Plan** — AI explores the codebase, produces implementation + test strategy
2. **Write Tests** — AI writes tests as a specification of behavior
3. **Review Tests** — Human approves the spec (hard gate)
4. **Write Code** — AI implements code to pass the approved tests
5. **Review Code** — Human approves the implementation (hard gate)
6. **Write E2E** — (Optional) AI writes Playwright E2E tests for user workflows
7. **Review E2E** — Human approves E2E tests (hard gate)

Steps 3, 5, and 7 are hard gates. AI does not proceed without explicit human approval.

## E2E Testing (v2.0)

After Step 5 approval, the skill offers to write exhaustive Playwright E2E tests for:
- User-facing UI components
- Multi-step user workflows
- 2D/3D view interactions
- Visual regression (snapshots)

E2E tests cover:
- ✅ Happy path workflows
- ✅ Edge cases (empty states, overflow, special characters)
- ✅ 2D and 3D view modes
- ✅ Panel/modal states
- ✅ Selection states (single, multi, none, mixed)
- ✅ Integration with other features

The skill follows existing test patterns from your Playwright repository and updates POMs as needed.

## Install

### As a Claude Code plugin

```bash
claude plugin add pranavtotla/ai-tdd
```

Then invoke with `/ai-tdd` or let it trigger automatically.

### As a standalone skill

Copy `skills/ai-tdd/` into your tool's skill directory:

| Tool | Path |
|------|------|
| Claude Code | `~/.claude/skills/ai-tdd/` or `.claude/skills/ai-tdd/` |
| OpenAI Codex | `~/.codex/skills/ai-tdd/` or `.codex/skills/ai-tdd/` |
| Cursor | `.cursor/skills/ai-tdd/` |
| Universal | `.agent/skills/ai-tdd/` |

Or clone and copy:

```bash
git clone https://github.com/pranavtotla/ai-tdd.git /tmp/ai-tdd

# Claude Code
cp -r /tmp/ai-tdd/skills/ai-tdd ~/.claude/skills/ai-tdd

# Codex
cp -r /tmp/ai-tdd/skills/ai-tdd ~/.codex/skills/ai-tdd

# Project-level (any tool)
cp -r /tmp/ai-tdd/skills/ai-tdd .agent/skills/ai-tdd
```

## Usage

The skill triggers automatically when you say things like:
- "build this with TDD"
- "write tests first"
- "implement with tests before code"
- "TDD workflow"
- "add e2e tests"
- "playwright tests"
- "2d/3d view tests"

Or invoke directly with `/ai-tdd`.

## Structure

```
.
├── .claude-plugin/
│   └── plugin.json                 # Claude Code plugin manifest
├── commands/
│   └── ai-tdd.md                   # /ai-tdd slash command
├── skills/
│   └── ai-tdd/
│       ├── SKILL.md                # Core workflow (v2.0 with E2E)
│       └── references/
│           ├── review-protocol.md  # Presentation format for review gates
│           └── playwright-e2e.md   # Playwright E2E testing patterns
├── README.md
└── LICENSE
```

## E2E Test Commands

After E2E tests are approved:

```bash
# Run E2E tests
npx playwright test [test-file] --timeout=120000

# Update snapshots
npx playwright test [test-file] --update-snapshots

# Debug mode
PWDEBUG=1 npx playwright test --grep "TC_XXX"
```

## License

MIT
