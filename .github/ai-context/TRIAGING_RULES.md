# Bug Triaging Rules for Temporal K8s Operator

## Repository Overview

This is a **Python Juju Charmed Operator** (not a Go/kubebuilder operator) that deploys and manages
the Temporal workflow engine on Kubernetes. It uses the `ops` framework, Pebble container management,
and Juju relations for service integration.

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

## Priority Rules

### Highest (Work in next sprint)
- Charm stuck in error/blocked state with no workaround
- Security vulnerabilities
- Data loss or corruption risk
- Blocking issues for track releases (e.g. 1.23/stable)

### High
- Integration breakage with related charms (temporal-admin-k8s, temporal-ui-k8s)
- PostgreSQL relation failures (blocks all functionality)
- Pebble service crashes or restart loops
- Issues affecting multiple users in production deployments

### Medium
- Non-critical relation failures with workarounds
- Feature enhancements requested by users
- Test flakiness in integration tests
- Documentation gaps that block adoption

### Low
- Nice-to-have configuration options
- Cosmetic log message improvements
- Low-impact bugs with easy workarounds

### Lowest
- Future architectural considerations
- Research items
- Long-term improvements

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

### Testing (tests/)
- Unit test additions: Low complexity
- Scenario test additions: Low-Medium complexity
- Integration test fixes: Medium complexity
- Test infrastructure changes (conftest.py, helpers.py): Medium complexity

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
- RBAC/permission changes
- Pebble layer changes
- Changes that touch test assertions

## PR Requirements Checklist

All non-docs PRs must include:
- [ ] New or updated test case (unit, scenario, or integration)
- [ ] Manual testing steps in PR description
- [ ] `tox` passes locally (fmt + lint + unit)
- [ ] Integration test run for relation or pebble changes
- [ ] CHANGELOG.md entry for user-facing changes
- [ ] Lib revision bump if `lib/charms/temporal_k8s/` is modified
