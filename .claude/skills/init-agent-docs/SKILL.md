---
name: init-agent-docs
description: Analyze this project and generate an AGENTS.md-based documentation structure.
---

## Task

1. Create AGENTS.md in the project root as the canonical, self-contained agent
   instruction file — this file must work standalone without any @imports, as
   other agents like GitHub Copilot do not support that syntax
2. Create CLAUDE.md as a separate file (not a symlink) that uses @imports to
   load the focused topic files — this gives Claude Code the benefit of modular
   loading while keeping AGENTS.md universally compatible
3. Create a docs/agent/ directory with focused topic files imported by CLAUDE.md
4. Create a .github/copilot-instructions.md symlink pointing to AGENTS.md

## Files to generate inside docs/agent/

Analyze the codebase and generate whichever of these are relevant — skip any
that don't apply:

- conventions.md — code style, naming conventions, patterns used in this codebase
- architecture.md — system overview, component structure, data flow
- decisions.md — architectural decisions and the reasoning behind them (ADR format)
- bugs.md — known issues, workarounds, and gotchas discovered in this codebase
- workflows.md — how to run, build, test, and deploy the project
- skills.md — tech stack, required knowledge, onboarding context
- context.md — domain-specific terminology, business rules, and glossary of
  terms that have specific meaning in this project

## AGENTS.md structure

AGENTS.md must be fully self-contained — no @imports. It should:

- Give a short project description
- Include a "Quick Start" section with the most essential commands
- Include a "Boundaries" section listing files, modules, or patterns that must
  never be modified without explicit human review — place this prominently
- Contain a concise but complete summary of conventions, architecture, known
  bugs, and workflows — enough to be useful to any agent that reads only this file
- Include a note at the bottom pointing to docs/agent/ for more detailed context:
  "For detailed reference files, see docs/agent/ — these are loaded automatically
  by Claude Code via CLAUDE.md"

## CLAUDE.md structure

CLAUDE.md is Claude Code's entry point and should:

- Give the same short project description as AGENTS.md
- Include the same "Quick Start" and "Boundaries" sections
- NOT duplicate the full content from AGENTS.md — instead use @imports to load
  the detailed topic files:

  ## Reference Files

  Consult these files only when the condition applies — do not load all of them
  by default:
  - @docs/agent/conventions.md — when writing, editing, or reviewing code
  - @docs/agent/architecture.md — when navigating the codebase or proposing
    structural changes
  - @docs/agent/decisions.md — when making or evaluating architectural or
    design choices
  - @docs/agent/bugs.md — when debugging, investigating errors, or working
    around known issues
  - @docs/agent/workflows.md — when running, building, testing, or deploying
  - @docs/agent/skills.md — when onboarding or assessing unfamiliar parts of
    the stack
  - @docs/agent/context.md — when interpreting domain-specific terms or
    business logic

  Use @docs/agent/<filename> syntax — the @ is Claude Code's directive for
  loading files into context, not just a path descriptor. Adapt conditions to
  what actually fits the project.

## Each generated file in docs/agent/ should start with a "When to consult

this file" header
At the top of every docs/agent/\*.md file, add a short section like:

## When to consult this file

Consult this file when writing or reviewing code. Not needed for debugging,
architecture discussions, or deployment tasks.

This serves as a future-proofing signal for agents that support dynamic
loading, and keeps the purpose of each file explicit for human maintainers.

## conventions.md requirements

- Only document conventions not already enforced by linters or formatters
- Where tooling handles a concern, reference the config file instead:
  "Formatting is fully handled by Prettier — see .prettierrc".
  Do not even mention what general settings are inside the config files, as
  this risks making the document stale if the config is updated but conventions.md is not.
- Include an "## Anti-patterns" section listing approaches the team has
  explicitly tried and rejected, with a brief reason for each:
  "We tried X — it caused Y — do not reintroduce it"

## decisions.md requirements

- Use ADR format: Context → Decision → Consequences
- Date-stamp every entry so the agent can weigh how recent each decision is
- Include an "## Anti-patterns" section for architectural approaches that have
  been evaluated and rejected, with reasoning
- Cross-reference related bugs in bugs.md where a decision is the known cause
  of a documented issue

## bugs.md requirements

- Distinguish clearly between:
  - Confirmed bugs with known workarounds
  - Suspected issues still under investigation
- Never present an unconfirmed suspicion with the same confidence as an
  established fact — label each entry accordingly
- Cross-reference related decisions in decisions.md where the root cause is
  a known architectural choice
- Include a "## When to update this file" section so future contributors know
  what belongs here

## context.md requirements

- Document domain-specific terms that have a precise meaning in this project
  that differs from their general usage
- Include business rules that are not obvious from the code itself
- Focus on knowledge that would cause an agent to produce technically correct
  but semantically wrong output if missing

## Instructions

- Base all content on what you actually find in the codebase — do not invent
  or assume
- Keep each file focused and concise — these are agent instructions, not human
  prose documentation
- Do not repeat in documentation what linters, formatters, or type checkers
  already enforce — reference the tooling config instead
- Each file should include a "## How to contribute to this file" section at the
  bottom, describing what kind of additions belong here and when a developer
  should update it — treat these files as a living feedback loop, not a
  one-time setup
- After creating all files, output a short summary of what was generated and
  what you found that informed each file

## Task: Verify and review generated files

After all files have been created, verify each generated file against the
requirements in this document:

- Check that every docs/agent/\*.md file has a "When to consult this file"
  section and a "How to contribute to this file" section
- Check that AGENTS.md is fully self-contained (no @imports)
- Check that CLAUDE.md uses @imports and does not duplicate full content
- Check that conventions.md anti-patterns use the
  "We tried X — it caused Y — do not reintroduce it" format
- Check that decisions.md entries are date-stamped and cross-reference
  bugs.md where a decision is the known cause of a documented issue
- Check that bugs.md clearly distinguishes confirmed bugs from suspected
  issues, and cross-references decisions.md where applicable
- Check that context.md covers terms that would cause semantically wrong
  output if an agent were unaware of them
- Check for any reusable utilities, path aliases, or patterns found in the
  codebase that agents should know about but are not yet documented
- Consider any other additions that would improve prompting accuracy for
  future work on this project
