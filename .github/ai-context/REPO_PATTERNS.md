# Historical Patterns for Temporal K8s Operator

## Common Issue Patterns

### Reconciliation Loop Issues
- **Keywords:** "reconcile error", "failed to update", "controller panic"
- **Common Cause:** Resource conflict or stale cache
- **Complexity:** High
- **Priority:** High
- **First Check:** Controller logs, resource status conditions
- **Pattern:** Usually requires retry logic or cache invalidation

### CRD Validation Errors
- **Keywords:** "validation failed", "invalid field", "unknown field"
- **Common Cause:** Schema mismatch between versions
- **Complexity:** Medium
- **Priority:** Medium (High if blocking upgrades)
- **First Check:** CRD yaml definitions, conversion webhooks
- **Pattern:** Often requires CRD update and migration guide

### RBAC Permission Issues
- **Keywords:** "forbidden", "unauthorized", "cannot get resource"
- **Common Cause:** Missing ClusterRole permissions
- **Complexity:** Medium
- **Priority:** High (blocks functionality)
- **First Check:** `config/rbac/role.yaml`
- **Pattern:** Add permission to ClusterRole, regenerate manifests

### Resource Not Found Errors
- **Keywords:** "not found", "missing resource", "404"
- **Common Cause:** Race condition or deletion ordering
- **Complexity:** Medium to High
- **Priority:** Medium
- **First Check:** Finalizer order, dependent resource creation
- **Pattern:** Add proper wait conditions or finalizer ordering

### Temporal Cluster Connection Issues
- **Keywords:** "connection refused", "unable to connect", "timeout"
- **Common Cause:** Service discovery or networking config
- **Complexity:** Medium
- **Priority:** High
- **First Check:** Service definitions, endpoint configurations
- **Pattern:** Verify service/endpoint status, check network policies

### Version Incompatibility
- **Keywords:** "unsupported version", "version mismatch", "incompatible"
- **Common Cause:** Temporal server/client version skew
- **Complexity:** High
- **Priority:** High
- **First Check:** Version compatibility matrix
- **Pattern:** Document supported versions, add validation

### Helm Chart Issues
- **Keywords:** "helm install failed", "template error", "values"
- **Common Cause:** Invalid values or template bug
- **Complexity:** Low to Medium
- **Priority:** Medium
- **First Check:** `charts/` directory, values.yaml
- **Pattern:** Validate with `helm lint`, test with common values

### Flaky Tests
- **Keywords:** "test failure", "intermittent", "flaky"
- **Common Cause:** Race conditions or timing issues
- **Complexity:** Medium
- **Priority:** Medium
- **First Check:** Test logs, timing assumptions
- **Pattern:** Add proper wait conditions, increase timeouts

## Resolution Patterns

### Issues We Can Auto-Resolve
- Documentation typos in README/docs
- Linting errors (gofmt, golint)
- Simple test fixes with clear errors
- Dependency updates (patch versions)
- Adding log statements

### Issues Requiring Careful Review
- Controller reconciliation changes
- CRD schema changes
- RBAC modifications
- Finalizer logic updates
- Version compatibility code

### Issues Needing Architecture Discussion
- New CRD resources
- Controller redesign
- Breaking API changes
- Multi-cluster support
- Performance optimization strategies

## Team Conventions

### Code Style
- Follow standard Go conventions (gofmt, golint)
- Controller methods: Reconcile, SetupWithManager, etc.
- Use kubebuilder markers for RBAC and CRD generation
- Error wrapping: Use fmt.Errorf with %w

### Testing Requirements
- Unit tests for new controller logic
- Integration tests for CRD changes
- E2E tests for user-facing features
- Table-driven tests preferred

### PR Requirements
- Kubebuilder generate manifests (`make generate manifests`)
- Update CHANGELOG.md for user-facing changes
- Add release notes for breaking changes
- Sign commits (DCO)

### Kubernetes Conventions
- Use recommended labels (app.kubernetes.io/*)
- Follow namespace scoping rules
- Implement proper finalizer cleanup
- Add owner references for child resources

## Operator-Specific Patterns

### Safe CRD Changes
- Adding optional fields (with defaults)
- Adding validation rules (non-breaking)
- Adding new enum values at end

### Risky CRD Changes
- Removing fields
- Changing field types
- Making optional fields required
- Reordering enum values

### Controller Best Practices
- Idempotent reconciliation
- Proper error handling and retries
- Status condition updates
- Event recording for user visibility

## External Dependencies

### Temporal Server Versions
- Check compatibility matrix
- Test against supported versions
- Document version requirements

### Kubernetes Versions
- Support last 3 minor versions
- Test against version-specific APIs
- Use deprecation helpers
