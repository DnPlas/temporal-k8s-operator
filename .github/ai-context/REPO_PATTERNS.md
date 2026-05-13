# Historical Patterns for Temporal K8s Operator

## Repository Context

This is a **Python Juju Charmed Operator** using the `ops` framework. It is not a Go/kubebuilder
project. Kubernetes interaction is mediated by Juju and Pebble, not direct controller-runtime loops.

- Language: Python 3.10+
- Framework: ops (Juju operator framework)
- Container management: Pebble
- Testing: tox → unit (ops[testing]), scenario, integration (pytest-operator on real k8s)
- Build: charmcraft

---

## Common Issue Patterns

### Charm Stuck in Blocked/Waiting Status
- **Keywords:** "blocked", "waiting", "status", "not ready", "inactive"
- **Common Cause:** Missing required relation (PostgreSQL, admin), or unmet config precondition
- **Complexity:** Medium
- **Priority:** High
- **First Check:** `_on_update_status` logic in `src/charm.py`, `is_ready()` in `src/state.py`
- **Pattern:** Add/fix status message to clearly indicate missing dependency; verify relation data is
  populated before setting Active status

### Pebble Service Failure / Container Not Ready
- **Keywords:** "pebble", "container", "service not running", "waiting for pebble"
- **Common Cause:** Pebble layer misconfiguration, incorrect command or environment variable
- **Complexity:** High
- **Priority:** High
- **First Check:** `_on_temporal_pebble_ready` in `src/charm.py`, Pebble layer dict definition
- **Pattern:** Validate layer config; check environment variables passed to Temporal process;
  review `templates/config.jinja` rendering

### PostgreSQL Relation Failures
- **Keywords:** "database", "db relation", "visibility", "postgresql", "connection refused"
- **Common Cause:** Dual DB setup (db + visibility) — both are required; one missing blocks charm
- **Complexity:** Medium
- **Priority:** High
- **First Check:** `src/relations/postgresql.py`, `is_ready()` checks for both relations
- **Pattern:** Ensure both `db` and `visibility` relation data is present before proceeding;
  check `DATABASE_NAME` and `VISIBILITY_DATABASE_NAME` literals

### Admin Relation Timing Issues
- **Keywords:** "admin", "schema", "not initialized", "temporal-admin"
- **Common Cause:** Schema initialization by `temporal-admin-k8s` charm not yet complete when
  Temporal server starts
- **Complexity:** Medium
- **Priority:** Medium
- **First Check:** `src/relations/admin.py`, relation event order in `src/charm.py`
- **Pattern:** Defer readiness until admin relation data confirms schema is initialized

### Config Template Rendering Errors
- **Keywords:** "config", "jinja", "template", "render", "KeyError", "UndefinedError"
- **Common Cause:** New config option added to `config.yaml` but not wired into template context,
  or vice versa
- **Complexity:** Medium
- **Priority:** High
- **First Check:** `templates/config.jinja`, `templates/dynamic_config.jinja`, and the dict passed
  to `jinja2.Template.render()` in `src/charm.py`
- **Pattern:** Align config.yaml options, charm.py context dict, and template variables as a trio

### State Deserialization Errors
- **Keywords:** "state", "peer relation", "KeyError", "json", "AttributeError on state"
- **Common Cause:** State key added/removed between charm versions without migration
- **Complexity:** High
- **Priority:** High
- **First Check:** `src/state.py` `__getattr__` / `__setattr__`, peer relation data
- **Pattern:** Add default values for new state keys; test upgrade scenarios

### TLS Certificate Issues
- **Keywords:** "certificate", "tls", "CSR", "frontend-certificates", "SSL"
- **Common Cause:** Certificate not yet provided by relation, or CSR format mismatch
- **Complexity:** Medium-High
- **Priority:** High
- **First Check:** TLS certificate relation handler in `src/charm.py`,
  `lib/charms/tls_certificates_interface/`
- **Pattern:** Check CSR generation logic, verify relation provides cert before enabling TLS

### OpenFGA Authorization Failures
- **Keywords:** "openfga", "authorization", "auth-enabled", "forbidden", "store"
- **Common Cause:** OpenFGA store/model not yet created, or `auth-enabled` config mismatch
- **Complexity:** Medium
- **Priority:** Medium
- **First Check:** `src/relations/openfga.py`, `tests/scenario/test_openfga_actions.py`
- **Pattern:** Verify OpenFGA store ID and authorization model ID in relation data

### S3 Archival Misconfiguration
- **Keywords:** "s3", "archival", "bucket", "aws", "boto3"
- **Common Cause:** Missing required S3 parameters (bucket, region, credentials)
- **Complexity:** Low-Medium
- **Priority:** Medium
- **First Check:** `src/relations/s3_archival.py`, `src/literals.py` required S3 params list
- **Pattern:** Validate all required S3 parameters are present before enabling archival

### Flaky Integration Tests
- **Keywords:** "intermittent", "flaky", "timeout", "race", "async"
- **Common Cause:** Timing assumptions in async pytest-operator tests; charm takes longer than
  expected to reach Active status
- **Complexity:** Medium
- **Priority:** Medium
- **First Check:** `tests/integration/helpers.py` wait utilities, `conftest.py` timeouts
- **Pattern:** Use proper `wait_for_idle()` calls with generous timeouts; avoid fixed `sleep()`

### Charm Library API Mismatch
- **Keywords:** "lib", "library", "charmcraft fetch-lib", "LIBPATCH", "LIBAPI"
- **Common Cause:** Library updated upstream but local copy is stale, or API version mismatch
- **Complexity:** Medium
- **Priority:** Medium
- **First Check:** `lib/charms/temporal_k8s/` LIBAPI/LIBPATCH versions, `charmcraft.yaml` libs
- **Pattern:** Bump LIBPATCH for backwards-compatible changes, LIBAPI for breaking changes;
  update consumers

### Ingress / Nginx Route Issues
- **Keywords:** "ingress", "nginx-route", "nginx_ingress_integrator", "external url"
- **Common Cause:** Ingress relation not configured or hostname not set
- **Complexity:** Low-Medium
- **Priority:** Medium
- **First Check:** nginx-route relation handling in `src/charm.py`,
  `lib/charms/nginx_ingress_integrator/`
- **Pattern:** Check service-hostname and service-port are correctly relayed

---

## Resolution Patterns

### Issues Eligible for Auto-Resolution
- Documentation typos (documentation/, README.md, docstrings outside charm.py)
- Comment-only changes in non-critical files
- Linting/formatting fixes (black, isort, flake8) with no logic change
- requirements.txt patch version bumps

### Issues Requiring Careful Human Review
- Any change to `src/charm.py` (even small ones)
- Any change to `src/state.py`
- Relation handler changes in `src/relations/`
- Jinja2 template changes in `templates/`
- Published library changes in `lib/charms/temporal_k8s/`

### Issues Needing Architecture Discussion
- New relation endpoint (requires metadata.yaml, library, handler, tests)
- New Temporal service type
- Multi-unit/HA architecture changes
- Breaking changes to the temporal-host-info relation interface

---

## Team Conventions

### Code Style
- Python 3.10+, formatted with `black` and `isort`
- Linting via `flake8` and `codespell`
- Type annotations encouraged; `pydantic` for config models
- Error handling: raise meaningful exceptions with context; log at appropriate level via `log.py`

### Testing Requirements (Mandatory)
- Every bug fix must include a test case (unit, scenario, or integration)
- Agents must not remove or weaken existing test assertions
- Unit tests: `tests/unit/` using `ops[testing]` harness
- Scenario tests: `tests/scenario/` for relation-level event testing
- Integration tests: `tests/integration/` using `pytest-operator` on real k8s
- Run locally: `tox -e unit`, `tox -e scenario`, `tox -e integration`

### PR Requirements
- `tox` must pass locally before opening PR (fmt + lint + unit)
- PR description must include manual testing steps (what commands to run, what to observe)
- CHANGELOG.md entry for any user-facing change
- Lib revision bump (`LIBPATCH` or `LIBAPI`) if `lib/charms/temporal_k8s/` is changed
- No `src/charm.py` logic changes without human review, even if automated tools propose them

### Juju/Charm Conventions
- Charm status should always reflect root cause ("waiting for PostgreSQL relation")
- Use `self.unit.status` to communicate state to operators
- Defer events when dependencies are not yet ready (don't silently succeed)
- Use `self.app.status` for app-level status (only leader unit can set it)
- Relations are the primary integration mechanism; no direct API calls between charms

### Pebble Conventions
- Always check `container.can_connect()` before pushing files or managing services
- Use `pebble.Layer` dicts; avoid string concatenation for layer config
- Log pebble plan on changes for debugging

---

## Operator-Specific Patterns

### Safe Changes to config.yaml
- Adding new optional config option (with a default value)
- Improving description strings
- Adjusting valid range for numeric options

### Risky Changes to config.yaml
- Removing an existing config option (breaks existing deployments)
- Renaming a config option
- Changing the type of an existing option

### Safe Changes to metadata.yaml
- Adding an optional relation endpoint
- Adding a new extra-binding

### Risky Changes to metadata.yaml
- Removing or renaming an existing relation endpoint (breaks dependent charms)
- Changing relation interface name
- Changing relation role (requires/provides)

### Safe Changes to lib/charms/temporal_k8s/
- Adding new functions/methods (bump LIBPATCH)
- Bug fixes to existing functions (bump LIBPATCH)

### Breaking Changes to lib/charms/temporal_k8s/
- Removing or renaming public functions (bump LIBAPI, notify consumers)
- Changing function signatures in backwards-incompatible ways

---

## External Dependencies

### Temporal Server Versions
- Charm track follows Temporal server major version (track/1.23 → Temporal 1.23.x)
- Check compatibility when updating OCI image tag

### Kubernetes Versions
- Tested via integration tests on Canonical K8s 1.33-classic/stable
- Juju >= 3.1 required

### Juju / ops Library
- Pinned to specific `ops` version in requirements.txt
- Upgrading ops may require signature changes in charm.py event handlers (auto-resolution eligible)

### Related Charms
- `temporal-admin-k8s`: Required for schema initialization
- `temporal-ui-k8s`: Optional UI integration
- `postgresql-k8s` or `pgbouncer-k8s`: Required database
- `openfga-k8s`: Optional authorization
