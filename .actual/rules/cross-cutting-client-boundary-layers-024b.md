# Standardize External Client Boundary Pattern for Internal APIs: Client Boundary Layers

These rules are ALWAYS ACTIVE for all internal API implementations that interact with external clients or services, including HTTP clients, cache layers, registry clients, and authentication boundaries.

### Rules

- **R-CBL-001** SHOULD: Client boundary layers SHOULD implement caching strategies where appropriate to reduce external calls and improve performance.

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
- Client boundary code achieves minimum 80% unit test coverage

<enforcement>
Claude Code MUST NOT skip or defer verification of client boundary patterns. All new external client implementations MUST be reviewed against these rules before acceptance.
</enforcement>