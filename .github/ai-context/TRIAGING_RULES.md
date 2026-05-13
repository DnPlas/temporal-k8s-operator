# Bug Triaging Rules for Temporal K8s Operator

## Repository Overview

This is a **Python Juju Charmed Operator** (not a Go/kubebuilder operator) that deploys and manages
the Temporal workflow engine on Kubernetes. It uses the `ops` framework, Pebble container management,
and Juju relations for service integration.

---

## Team Guardrails (Hard Rules)

These rules are non-negotiable for any automated or AI-assisted changes:

1. **src/charm.py is restricted.** Automated changes are only allowed for:
   - Typo fixes in comments or strings
   - Minimal signature changes when an upstream API is updated (e.g. ops library upgrade)
   - Any logic change requires human review.

2. **Tests are immutable by agents.** Agents must never:
   - Remove tests to make CI green
   - Change what a test asserts in order to make it pass
   - Skip or xfail tests as a workaround

3. **Every code fix must include a test.** Unless the issue is purely documentation,
   the PR must introduce a new test case covering the fix (unit, scenario, or integration).

4. **PRs must include manual testing steps.** Every PR description must contain
   explicit steps a reviewer can follow to manually verify the fix.

---

## Complexity Guidelines

### Trivial (Auto-resolvable)
- Documentation typos and grammar fixes (docs/, README.md, documentation/)
- Comment clarifications in non-charm.py files
- Formatting fixes (black, isort, flake8)

### Low
- Adding/updating log messages (`log.py`, event handler log calls)
- Updating constants in `src/literals.py`
- Config option description updates in `config.yaml`
- Dependency patch updates (non-breaking, e.g. patch version in requirements.txt)
- Adding or updating Jinja2 templates (`templates/`) with no logic change

### Medium
- New configuration option wired end-to-end (config.yaml → charm.py → template)
- Adding a new relation endpoint handler in `src/relations/`
- Test coverage improvements (unit or scenario tests)
- Bug fixes in relation handlers (`src/relations/*.py`)
- Pebble layer adjustments for container configuration
- State management additions (`src/state.py`)

### High
- Changes to core event handlers in `src/charm.py`
- New relation type (requires metadata.yaml + lib + handler)
- Pebble service lifecycle changes (start/stop/restart ordering)
- TLS certificate handling changes
- Multi-service coordination changes (frontend/history/matching/worker)
- Changes affecting charm upgrade paths

### Critical
- Data loss scenarios (database relation handling)
- Charm status stuck in error/blocked with no recovery path
- Security vulnerabilities (auth bypass, secret exposure)
- Breaking changes to relation interfaces used by other charms
- Schema migration failures

---

## Severity Assessment

Severity must be derived strictly from the four factors below. Do not assess it independently.

| factors                                          | level    |
|--------------------------------------------------|----------|
| (affects_db OR causes_error_state) AND no workaround | critical |
| any factor true AND no workaround                | high     |
| any factor true AND workaround exists            | medium   |
| no factors true                                  | low      |

### Factor definitions
- **affects_db**: issue impacts PostgreSQL relations, visibility DB, or data persistence
- **affects_integrations**: issue breaks a required relation (admin, postgresql) or an optional one (openfga, s3, ui, ingress)
- **causes_error_state**: charm enters Error or Blocked status as a result
- **has_workaround**: a documented manual workaround exists that restores functionality

---

## Steps to Reproduce Assessment

- **present**: issue body includes clear reproduction steps
- **missing**: issue is a bug report but lacks reproduction steps (flag this to the reporter)
- **not_applicable**: use for documentation issues, feature requests, Charmhub display/metadata
  bugs, or any issue where reproducing locally is not relevant to the fix

---

## Effort Estimation (Pulses)

1 pulse = 2 weeks (one sprint).

| estimate | meaning                          |
|----------|----------------------------------|
| 0.25     | a few hours to 1 day             |
| 0.5      | roughly 1 week                   |
| 1        | one full sprint                  |
| 2        | two sprints (multi-sprint work)  |
| 3+       | large feature or architectural change |

---

## Label System

Only two labels are applied automatically. Do not introduce others without team agreement.

| label                | meaning                                                       |
|----------------------|---------------------------------------------------------------|
| `needs-human-review` | default for all issues; a human must assess and act          |
| `needs-copilot`      | AI assessed as auto-eligible; assign Copilot from the sidebar |
| `severity:critical`  | applied in addition to the above when severity level is critical; use to filter urgent issues |

### Copilot assignment
GitHub does not expose a public API for triggering the Copilot coding agent programmatically.
The `needs-copilot` label is the queue signal. A human opens the issue, clicks Assignees in
the sidebar, and selects Copilot. The agent then creates a PR automatically.

### Auto-resolve eligibility criteria (all must be true)
- `auto_resolve_eligible: true` from AI analysis
- `confidence: high`
- `complexity: trivial` or `low`
- `risk_level: low`
- No blocking labels present: `security`, `breaking-change`, `no-ai-resolution`, `needs-discussion`

---

## High-Risk Code Areas

Files requiring extra scrutiny (human review mandatory):

- `src/charm.py` — Main charm class; all Juju event handlers live here
- `src/state.py` — Peer-relation state store; data loss risk if broken
- `src/relations/postgresql.py` — Database relations; blocks charm readiness
- `src/relations/admin.py` — Schema initialization coordination
- `metadata.yaml` — Relation endpoint definitions; breaking changes affect ecosystem
- `templates/config.jinja` — Temporal server config; incorrect output causes service failure
- `templates/dynamic_config.jinja` — Runtime Temporal config
- `lib/charms/temporal_k8s/` — Published charm library; API changes affect consumers

---

## Category-Specific Rules

### Charm Core (charm.py)
- Event handler bugs: High complexity, High priority
- Pebble readiness issues: High complexity, High priority
- Config propagation bugs: Medium complexity, Medium priority
- Status reporting issues: Low-Medium complexity

### State Management (state.py)
- Data serialization bugs: High complexity, High priority
- State migration issues: High complexity, Critical priority

### Relation Handlers (src/relations/)
- PostgreSQL relation failures: High priority (blocks everything)
- Admin relation timing issues: Medium complexity, High priority
- Optional relation bugs (OpenFGA, S3, UI, Ingress): Medium priority
- TLS certificate relation issues: Medium-High complexity

### Configuration (config.yaml + templates/)
- New config option additions: Low complexity
- Template rendering bugs: Medium complexity, High priority
- Validation rule updates: Low-Medium complexity

### Libraries (lib/charms/temporal_k8s/)
- Published library changes: High complexity (consumers are affected)
- Bumping lib revision required for any change

### Documentation (documentation/, README.md)
- Low complexity across the board
- No test required
- No manual testing steps required (docs PRs only)
- steps_to_reproduce: not_applicable

### Testing (tests/)
- Unit test additions: Low complexity
- Scenario test additions: Low-Medium complexity
- Integration test fixes: Medium complexity
- Test infrastructure changes (conftest.py, helpers.py): Medium complexity

---

## Auto-Resolution Eligibility

Safe for auto-resolution (with mandatory PR review):
- Documentation typos (documentation/, README.md)
- Comment-only changes in non-charm.py files
- Linting fixes (black, isort) that don't change logic
- requirements.txt patch version bumps

NOT safe for auto-resolution (human review required):
- Any change to `src/charm.py`
- Any change to `src/state.py`
- Any change to `src/relations/`
- Any change to `metadata.yaml`
- Any change to `templates/`
- Any change to `lib/charms/temporal_k8s/`
- Pebble layer changes
- Changes that touch test assertions

---

## PR Requirements Checklist

All non-docs PRs must include:
- [ ] New or updated test case (unit, scenario, or integration)
- [ ] Manual testing steps in PR description
- [ ] `tox` passes locally (fmt + lint + unit)
- [ ] Integration test run for relation or pebble changes
- [ ] CHANGELOG.md entry for user-facing changes
- [ ] Lib revision bump if `lib/charms/temporal_k8s/` is modified
