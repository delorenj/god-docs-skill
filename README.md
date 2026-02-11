# GOD Docs

### Your documentation is lying to you. Let's fix that.

---

You know that architecture doc from six months ago? The one that says the auth service talks to Redis? Yeah, you ripped Redis out in March. The doc didn't get the memo. Nobody updated it because nobody *knew* it needed updating — and even if they did, it was Friday and they had a deploy to ship.

This is the normal state of documentation in software. Permanent, silent decay.

**GOD Docs (Guaranteed Organizational Documents)** are a system for making documentation staleness *observable and enforceable* — not through better habits, but through the same deterministic machinery you already use for tests and linting.

When you change source code but don't update the corresponding GOD Doc, the system knows. Your pre-commit hook tells you. Your CI blocks the merge. There is no "we'll update it next sprint."

---

## How It Works (30-Second Version)

1. **Split your repo into non-overlapping domains.** Every file belongs to exactly one domain. (If you can't do this, fix your architecture first.)

2. **Create GOD Docs at three levels:** System → Domain → Component. Each from a template. Same structure every time.

3. **Declare dependencies between docs.** Component A consumes events from Component B? A's GOD Doc declares B's GOD Doc as a dependency.

4. **Enforce freshness automatically.** When source files change, the system checks if the corresponding GOD Doc was updated too. If not → stale → blocked.

That's it. Documentation as a build artifact. Staleness as a CI failure.

---

## What's In This Skill

This is an [OpenClaw](https://github.com/openclaw/openclaw) / AI agent skill that teaches any compatible agent to implement, maintain, and enforce the GOD Doc system.

```
god-docs-skill/
├── SKILL.md                     # Agent instructions (lean, ~750 words)
├── references/
│   └── spec.md                  # Full system specification (~2,100 words)
└── assets/
    ├── COMPONENT-GOD-TEMPLATE.md   # Level 2 template
    ├── DOMAIN-GOD-TEMPLATE.md      # Level 1 template
    ├── pre-commit                  # Git hook for local enforcement
    └── god-check.yml               # GitHub Actions CI gate
```

**SKILL.md** is what the agent loads — quick start, hierarchy, dirty-check algorithm, template sections, enforcement commands. Everything needed to bootstrap GOD Docs in a repo.

**references/spec.md** is the full manifesto — loaded on demand when deep context is needed. Covers the philosophy, the pre-requisites, the dependency graph system, the complete dirty-check algorithm, and Degenerate (the Rust drift detector).

**assets/** are copy-paste ready — templates, hook, and CI action. Drop them into any repo.

---

## The Three Levels

```
docs/GOD.md                          # System — topology, component registry
docs/domains/{domain}/GOD.md         # Domain — component map, event contracts  
{component}/GOD.md                   # Component — events, APIs, architecture
```

**System** is the 30,000-foot view. **Domain** scopes to one concern. **Component** is the deep dive. Each links down to the next. Progressive disclosure — never dump everything at one level.

---

## The Dirty-Check

```
Source files changed + GOD Doc didn't = STALE
```

Three enforcement layers:

| Layer | Where | Behavior |
|-------|-------|----------|
| **Git hook** | Pre-commit, local | Interactive: update now / skip / abort |
| **CI action** | Pull request | Hard gate: stale docs = red CI = no merge |
| **Degenerate** | On-demand CLI | Deep audit: transitive deps, drift severity, commit history |

Any single layer can be bypassed. All three together make staleness harder than freshness.

---

## Degenerate

> *"Fight the entropy of documentation."*

[Degenerate](https://github.com/delorenj/33GOD/tree/main/degenerate) is a Rust CLI that implements the full dirty-check algorithm with transitive dependency resolution. It tracks per-component sync state, walks the dependency graph, and reports drift severity down to the commit.

```
$ degenerate drift --commits

⚠ Infrastructure (12 commits behind)
    → bloodbank (8 commits)
        a3f8c91 Add retry logic to publisher — Jarad
        b7c2e44 Fix DLQ routing key pattern — Jarad
        ...
    → holyfields (4 commits)

✓ Workspace Management (up to date)

Summary: 1 domain with 12 total commits of drift.
```

Shell scripts handle the common case. Degenerate handles the hard case — transitive staleness, submodule repos, drift-over-time reporting.

---

## Inter-GOD Dependencies

The thing that makes this more than "just templates with a hook."

Each GOD Doc declares what it depends on:

```markdown
<!-- GOD-DEPS:
  - event-bus/GOD.md
  - schema-registry/GOD.md
-->
```

When `schema-registry/GOD.md` changes, every doc that declared it as a dependency is **transitively flagged stale**. Not because someone remembered to check — because the graph did it automatically.

---

## Quick Start

### Using as an Agent Skill

Install via [ClawHub](https://clawhub.com) or copy the skill directory into your agent's skills folder:

```bash
# OpenClaw
clawhub install god-docs

# Or manually
cp -r god-docs-skill/ ~/.openclaw/skills/god-docs/
```

Then ask your agent: *"Set up GOD Docs in this repo"* — it'll know what to do.

### Manual Setup (No Agent)

```bash
# Create structure
mkdir -p docs/templates docs/domains docs/sync

# Copy templates
cp assets/COMPONENT-GOD-TEMPLATE.md docs/templates/
cp assets/DOMAIN-GOD-TEMPLATE.md docs/templates/

# Install git hook
cp assets/pre-commit .githooks/pre-commit
chmod +x .githooks/pre-commit
git config core.hooksPath .githooks

# Copy CI action
mkdir -p .github/workflows
cp assets/god-check.yml .github/workflows/

# Create your first GOD Doc
cp docs/templates/COMPONENT-GOD-TEMPLATE.md my-service/GOD.md
# Fill in the {{PLACEHOLDER}} tokens...
```

---

## Design Principles

1. **Documentation is a build artifact.** If it can't be verified, it can't be trusted.
2. **Non-overlapping domains are the foundation.** Ambiguous ownership = ambiguous staleness.
3. **Templates prevent structural drift.** The tenth GOD Doc you read navigates like the first.
4. **Dependencies are explicit and tracked.** Not tribal knowledge — declared, machine-readable relationships.
5. **Dev-facing, not user-facing.** GOD Docs are for builders. READMEs are for users. Different audiences, different documents.

---

## Origin

GOD Docs were developed for [33GOD](https://github.com/delorenj/33GOD), an event-driven agentic pipeline where 17+ microservices needed to stay architecturally documented without anyone losing their mind. Degenerate is the reference implementation of the drift detector.

The system works because it doesn't trust humans to remember. It trusts `git diff`.

---

## License

MIT

---

*"Everything tends toward disorder. But we can at least document the decline."*
