# Vibe Coding Workflow

> English | [简体中文](README.md)

> Real vibe coding isn't "give the AI one sentence and wait for it to finish." It's a process that makes AI output **controllable, reviewable, and verifiable.**

This is a [Claude Code Skill](https://docs.claude.com/en/docs/claude-code/skills) that bakes "spec-driven development" into a forced pipeline: **Context Engineering → adversarial Reviewer subagent → PRD → implementation doc → test design → stepwise implementation + incremental commits → acceptance gate**.

## The Iron Law

**Before an reviewed, written plan is approved, no code is written.**

You don't need to read every line of code — but you must put your attention on **process design, context quality, acceptance criteria, and risk control**. Once the design is approved, everything runs autonomously: branching, stepwise implementation, incremental auto-commits, and the acceptance gate are all done by the AI. You step in exactly once — at the "design approval" point.

## Why It Exists

What AI produces looks logically complete, yet often hides pitfalls: misunderstood requirements, over-engineering, missed edge cases, poor data structures, insufficient tests, and disregard for production compatibility. This skill fights those with three things:

1. **Every key deliverable is reviewed by an "adversarial Reviewer subagent"** — the Writer advances, the Reviewer picks holes, looping until convergence.
2. **Tests are designed alongside implementation**, with an approved spec in place before any code.
3. **After the design is approved, the rest is fully automatic.**

> A single AI is self-consistent; having a second AI dedicated to finding problems is the crux of this whole process.

## Installation

Drop the whole directory into Claude Code's skills directory:

```bash
# Into the global skills directory
git clone https://github.com/leoomo/vibe-coding-workflow.git \
  ~/.claude/skills/vibe-coding-workflow
```

You can also place it under a project's `.claude/skills/` to scope it to that project. Once installed, invoke it in Claude Code with `/vibe-coding-workflow`.

## Usage

```bash
# Interactive mode (recommended)
/vibe-coding-workflow <task description>

# With an explicit feature name
/vibe-coding-workflow <feature name> <task description>
```

| Argument | Description |
|----------|-------------|
| `<feature name>` | Optional, the feature directory name (kebab-case) |
| `<task description>` | Required, the feature to implement or the bug to fix |

Examples:

```bash
/vibe-coding-workflow User invitations: inviter sends email, invitee clicks link to complete registration

/vibe-coding-workflow invite-by-email Fix: invitation link gets truncated in some email clients and can't be clicked
```

## Workflow

Each task advances through these seven steps:

| Step | Phase | Manual? |
|------|-------|---------|
| Step 0 | Bootstrap, naming, and **create an isolated branch** | Auto |
| Step 1 | **Context Engineering** (Explore agent reads the project deeply) | Auto |
| Step 2 | **PRD** (Writer + adversarial Reviewer loop) | Auto |
| Step 3 | **Implementation doc + test design** (Writer + Reviewer loop) | Auto |
| Step 4 | **Plan approval gate** | ⚠️ The only manual gate |
| Step 5 | Stepwise implementation + **incremental auto-commits** | Auto |
| Step 6 | **Acceptance gate** (full test suite / bugfix dual-branch verification) | Auto |
| Step 7 | Summary and memory confirmation | Auto |

- **Step 4 is the only point that requires human input.** Once confirmed, Steps 5–7 run automatically with no step-by-step asking.
- A bugfix task adds a **dual-branch verification** at Step 6: confirm the relevant tests genuinely fail before the fix and genuinely pass after it — proving the tests actually cover that bug rather than "passing by accident."

## Artifacts Produced

Each task produces a set of spec docs **inside the target project**, for review and archival:

```
<project>/.vibe-coding/<feature-name>/
├── context.md      # Context engineering output
├── prd.md          # What to do (Writer + Reviewer polish)
├── impl.md         # How to do it (implementation doc + test design)
├── review-log.md   # Reviewer findings and how each was handled
└── todo.md         # Fine-grained tasks (each with tests + a commit point)
```

> These docs are generated in the project you're working on, not in this skill's directory.

## Rules (rigid, non-skippable)

- **Spec before code**: no business code is written before `impl.md` is approved.
- **Auto after approval**: once Step 4 is approved, Steps 5–7 execute autonomously, without step-by-step asking.
- **Isolated branch**: Step 0 creates the branch up front; all commits land on it, keeping them rollback-safe and off the main branch.
- **Reviewer is mandatory**: the PRD and impl phases must pass through the adversarial Reviewer subagent and converge.
- **Every step ships tests + auto-commits**: no "naked code" steps, no single mega-commit.
- **Acceptance gate is enforced**: the full test suite must be green; bugfixes need dual-branch verification.
- **Reuse first**: check `context.md` for existing implementations before adding anything new.
- **Test commands follow the project**: which commands the acceptance gate runs depends on the actual project (see the "Testing" section of `context.md`).

## Repository Structure

```
vibe-coding-workflow/
├── SKILL.md                                  # Main skill entry (process definition)
├── README.md                                 # This file (Chinese)
├── README.en.md                              # English version
└── references/
    ├── context-checklist.md                  # Step 1 context collection checklist
    ├── prd-template.md                       # Step 2 PRD template
    ├── implementation-doc-template.md        # Step 3 implementation doc + test design template
    ├── reviewer-prompt.md                    # Step 2/3 adversarial Reviewer prompt
    └── acceptance-checklist.md               # Step 6 acceptance gate checklist
```

The templates/prompts under `references/` are **read on demand** at their respective steps — they aren't dumped into context all at once.

## When to Use

- ✅ Implementing a **new feature** (feature)
- ✅ Fixing a **complex bug** (bugfix, with dual-branch verification)
- ✅ Any AI coding task where you want **reviewable, verifiable, rollback-safe** output

Not a great fit: a one-line typo fix, or a purely exploratory question that needs no code landed.

## Not Included

This skill **does not cover the merge / PR stage** — the workflow ends once the acceptance gate passes. Merging is left to you or another process.

## License

MIT
