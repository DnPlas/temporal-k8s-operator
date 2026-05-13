# Bug Triaging Rules for Temporal K8s Operator

## Complexity Guidelines

### Trivial (Auto-resolvable)
- Documentation typos and grammar fixes
- README updates with no technical changes
- Comment clarifications
- Formatting/linting issues

### Low
- Simple configuration parameter additions
- Basic validation errors
- Log message improvements
- Minor UI/CLI text changes
- Dependency patch updates (non-breaking)

### Medium
- CRD (Custom Resource Definition) field additions
- Reconciliation logic for specific resources
- Test coverage improvements
- Integration with existing Temporal features
- Bug fixes requiring Kubernetes API changes

### High
- Controller architecture changes
- State management logic updates
- Multi-resource reconciliation changes
- Performance optimizations affecting core loops
- Security-related changes
- Breaking API changes

### Critical
- Data loss prevention
- Cluster stability issues
- Security vulnerabilities
- Upgrade/migration paths
- Core reconciliation loop failures

## Priority Rules

### Highest (Work in next sprint)
- Cluster crashes or data loss
- Security vulnerabilities
- Critical bugs affecting production deployments
- Blocking issues for major releases

### High
- Performance degradation >30%
- Features blocking customer adoption
- Bugs affecting multiple users
- Upgrade path issues

### Medium
- Feature enhancements
- Non-critical bugs with workarounds
- Documentation gaps
- Test flakiness

### Low
- Nice-to-have features
- Cosmetic improvements
- Low-impact bugs

### Lowest
- Future considerations
- Research items
- Long-term architectural ideas

## High-Risk Code Areas

Files/directories requiring extra scrutiny:
- `controllers/` - Core reconciliation logic
- `api/v1beta1/` - CRD definitions (breaking changes)
- `internal/resource/` - Resource creation/update logic
- `pkg/version/` - Version compatibility
- `cmd/` - CLI entry points

## Category-Specific Rules

### Operator/Controller Issues
- Reconciliation loop bugs: High complexity, High priority
- State drift issues: Medium to High complexity
- Watch/event handling: Medium complexity
- Finalizer logic: High complexity (can cause cluster issues)

### CRD/API Issues
- Field additions: Low to Medium complexity
- Breaking changes: Critical complexity, requires migration
- Validation rules: Low to Medium complexity
- Defaulting logic: Low complexity

### Kubernetes Integration
- RBAC permission issues: Medium complexity, High priority
- Service account problems: Medium complexity
- Namespace scoping: Medium complexity
- Resource quota/limits: Low to Medium complexity

### Temporal-Specific
- Temporal cluster configuration: Medium complexity
- Archive/visibility setup: Medium complexity
- Worker deployment: Medium complexity
- Version compatibility: High complexity

### Documentation
- API documentation: Low complexity, Medium priority
- User guides: Low complexity, Low priority
- Code comments: Trivial complexity

### Testing
- Unit test additions: Low complexity
- Integration test fixes: Medium complexity
- E2E test flakiness: Medium complexity, Medium priority
- Test framework changes: High complexity

## Auto-Resolution Eligibility

Safe for auto-resolution (with PR review):
- Documentation typos
- Comment clarifications
- Linting fixes in test files
- Dependency patch updates (automated tools)
- README formatting

NOT safe for auto-resolution:
- Any changes to controllers/
- Any changes to api/v1beta1/
- RBAC/permission changes
- Resource creation logic
- Finalizer logic
- Version compatibility code
