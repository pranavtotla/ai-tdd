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
│       ├── SKILL.md                # Core workflow
│       └── references/
│           └── review-protocol.md  # Presentation format for review gates
├── README.md
└── LICENSE
```

## License

MIT
