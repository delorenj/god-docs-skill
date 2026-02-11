# GOD Docs Skill

Architecture docs drift. GOD Docs gives you a practical way to keep them in sync with code.

This repo ships the agent skill, templates, and enforcement bits to make that happen.

## TL;DR

- Split the repo into clear, non-overlapping domains.
- Keep docs in a 3-level structure: system -> domain -> component.
- Run freshness checks in pre-commit and CI.
- Declare doc dependencies so upstream changes mark downstream docs stale.

## Repo Layout

```text
god-docs-skill/
├── SKILL.md                        # Agent workflow/instructions
├── references/
│   └── spec.md                     # Full spec
└── assets/
    ├── COMPONENT-GOD-TEMPLATE.md   # Component doc template
    ├── DOMAIN-GOD-TEMPLATE.md      # Domain doc template
    ├── pre-commit                  # Local freshness gate
    └── god-check.yml               # CI freshness gate
```

## Doc Hierarchy

```text
docs/GOD.md                          # System overview
docs/domains/{domain}/GOD.md         # Domain architecture + contracts
{component}/GOD.md                   # Component-level details
```

Rule of thumb: every source file belongs to exactly one domain. Without clear ownership, freshness checks get fuzzy.

## Freshness Check

If relevant source files changed and the mapped GOD doc did not, that doc is stale.

Enforcement layers:
- `assets/pre-commit` for local checks.
- `assets/god-check.yml` for PR/CI checks.
- Degenerate CLI (optional) for deeper transitive drift analysis.

## Dependencies Between Docs

Docs can declare dependencies with `GOD-DEPS`:

```markdown
<!-- GOD-DEPS:
  - event-bus/GOD.md
  - schema-registry/GOD.md
-->
```

When an upstream doc changes, dependents are flagged stale until reviewed/updated.

## Quick Start

### Agent Install

```bash
# OpenClaw
clawhub install god-docs

# Manual install
cp -r god-docs-skill/ ~/.openclaw/skills/god-docs/
```

Then ask your agent to set up GOD Docs in your repo.

### Manual Setup

```bash
# Create docs structure
mkdir -p docs/templates docs/domains docs/sync

# Copy templates
cp assets/COMPONENT-GOD-TEMPLATE.md docs/templates/
cp assets/DOMAIN-GOD-TEMPLATE.md docs/templates/

# Install pre-commit hook
mkdir -p .githooks
cp assets/pre-commit .githooks/pre-commit
chmod +x .githooks/pre-commit
git config core.hooksPath .githooks

# Install CI workflow
mkdir -p .github/workflows
cp assets/god-check.yml .github/workflows/

# Create first component doc
cp docs/templates/COMPONENT-GOD-TEMPLATE.md my-service/GOD.md
```

## Optional: Degenerate CLI

[Degenerate](https://github.com/delorenj/33GOD/tree/main/degenerate) is the Rust tool for deeper drift audits.

Useful for:
- Transitive drift across domains/components.
- Commit-level drift reporting.
- Repo-wide audits beyond simple file-change checks.

## Principles

1. Docs should be verifiable.
2. Ownership should be explicit.
3. Structure should be consistent.
4. Dependencies should be machine-readable.
5. Internal architecture docs and user-facing READMEs have different jobs.

## Origin

GOD Docs came out of the [33GOD](https://github.com/delorenj/33GOD) ecosystem to keep multi-service architecture docs trustworthy at development speed.

## License

MIT
