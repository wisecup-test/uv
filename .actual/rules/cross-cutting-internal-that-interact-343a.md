# Standardize External Client Boundary Pattern for Internal APIs: Internal That Interact

These rules are ALWAYS ACTIVE for all internal API implementations that interact with external clients or services, including HTTP clients, cache layers, registry clients, and authentication boundaries.

### Rules

- **R-ECBP-001** MUST: Internal APIs that interact with external clients MUST define explicit boundary interfaces that separate internal logic from external communication.

### Verify

```bash
# Detect trait-based client implementations
grep -r 'impl.*Client' crates/uv-*/src/ | grep -c 'trait'

# Count client boundary structures
rg 'pub struct.*Client' crates/ --type rust | wc -l

# Verify client tests exist
cargo test --all -- --test-threads=1 | grep -c 'test.*client'
```

**Accept when:**
- All new external client implementations define explicit boundary traits or interfaces
- Client boundary code includes unit tests with mock implementations demonstrating testability
- Code review confirms consistent error handling and authentication patterns across client boundaries
- CI pipeline successfully runs verification commands showing trait-based client patterns

<enforcement>
Claude Code MUST NOT skip or defer verification. All new client boundary implementations require trait-based interfaces, unit tests with mocks, and code review confirmation before merge.
</enforcement>