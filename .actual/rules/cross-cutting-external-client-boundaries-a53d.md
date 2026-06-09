# Standardize External Client Boundary Pattern for Internal APIs: External Client Boundaries

These rules are ALWAYS ACTIVE for all internal API implementations that interact with external clients or services, including HTTP clients, cache layers, registry clients, and authentication boundaries.

### Rules

- **R-EX-001** MUST: External client boundaries MUST be designed to support testing through dependency injection or trait abstraction.

### Verify

```bash
# Verify trait-based client implementations exist
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
Claude Code MUST NOT skip or defer verification. All external client boundary implementations MUST satisfy R-EX-001 before merge. Code review by architecture team is mandatory for new client boundary code. Unit test coverage minimum 80% is required.
</enforcement>