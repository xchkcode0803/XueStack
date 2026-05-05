# XueStack

XueStack is my personal Codex skill pack for turning coding agents into sharper project-shipping partners.

The goal is simple: make agents follow durable engineering practices, keep code modular and testable, and avoid shipping slop. Each skill captures a workflow, architecture rule, review habit, or delivery standard that I want Codex to reuse across projects.

## Principles

- **Anti-slop by default**: generated code should be intentional, maintainable, and easy to review.
- **Best practices as reusable skills**: repeatable standards belong in skills, not pasted prompts.
- **Separation of concerns**: apps should keep delivery, domain logic, data access, and infrastructure cleanly separated.
- **Modular and testable code**: every workflow should bias toward small interfaces, explicit contracts, and meaningful tests.
- **Shipping mindset**: skills should help move projects from idea to production without skipping quality gates.

## Skills

### `xue-nextjs-best-practices`

Applies XueStack architecture rules to Next.js App Router projects.

Use it when building, reviewing, or refactoring Next.js apps that need:

- feature-driven project structure
- clean domain boundaries
- thin Server Actions
- service-layer business logic
- public feature APIs through `index.ts`
- Drizzle/Postgres guidance when that stack is present
- modular testing and boundary checks

Location:

```text
skills/xue-nextjs-best-practices/
```

### `xue-quality-gate`

Runs final validation and self-review before committing, pushing, opening a PR, or declaring implementation complete.

Use it when a coding task needs:

- repo-specific verification commands
- CodeRabbit CLI review in agent mode
- triage of review findings before commit
- a concise final xue-quality-gate report

Location:

```text
skills/xue-quality-gate/
```

## Repository Structure

```text
skills/
  xue-quality-gate/
    SKILL.md
    agents/
      openai.yaml
  xue-nextjs-best-practices/
    SKILL.md
    agents/
      openai.yaml
    references/
      architecture.md
      drizzle-postgres.md
      eslint-boundaries.md
      testing.md
```

## Iteration Workflow

1. Add or refine one skill at a time.
2. Keep `SKILL.md` concise and action-oriented.
3. Move deeper examples, rules, and references into `references/`.
4. Add scripts only when a workflow needs deterministic automation.
5. Validate each skill after changes.
6. Use the skill on real projects, then fold useful corrections back into the skill.

## Validation

Validate a skill with:

```bash
python3 /Users/4525150/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/xue-nextjs-best-practices
```

## Vision

XueStack should become a growing library of personal engineering taste: the practices, standards, and workflows I want every coding agent to follow when helping me build and ship software.
