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
```

1. **Plan** — AI explores the codebase, produces implementation + test strategy
2. **Write Tests** — AI writes tests as a specification of behavior
3. **Review Tests** — Human approves the spec (hard gate)
4. **Write Code** — AI implements code to pass the approved tests
5. **Review Code** — Human approves the implementation (hard gate)

Steps 3 and 5 are hard gates. AI does not proceed without explicit human approval.

## Install

Drop the `ai-tdd/` folder into your tool's skill directory:

| Tool | Path |
|------|------|
| Claude Code | `~/.claude/skills/ai-tdd/` or `.claude/skills/ai-tdd/` |
| OpenAI Codex | `~/.codex/skills/ai-tdd/` or `.codex/skills/ai-tdd/` |
| Cursor | `.cursor/skills/ai-tdd/` |
| Universal | `.agent/skills/ai-tdd/` |

Or clone directly:

```bash
# Claude Code (personal)
git clone https://github.com/pranavtotla/ai-tdd.git ~/.claude/skills/ai-tdd

# Codex (personal)
git clone https://github.com/pranavtotla/ai-tdd.git ~/.codex/skills/ai-tdd

# Project-level (any tool)
git clone https://github.com/pranavtotla/ai-tdd.git .agent/skills/ai-tdd
```

## Usage

The skill triggers automatically when you say things like:
- "build this with TDD"
- "write tests first"
- "implement with tests before code"
- "TDD workflow"

Or invoke directly as `/ai-tdd` where supported.

## Structure

```
ai-tdd/
├── SKILL.md                    # Core workflow (1,224 words)
└── references/
    └── review-protocol.md      # Presentation format for review gates
```

## License

MIT
