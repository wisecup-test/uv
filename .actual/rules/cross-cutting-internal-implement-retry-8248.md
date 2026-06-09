# Standardize External Client Boundary Pattern for Internal APIs: Internal Implement Retry

These rules are ALWAYS ACTIVE for all internal API implementations that interact with external clients or services, including HTTP clients, cache layers, registry clients, and authentication boundaries.

### Rules

- **R-BOUNDARY-001** MAY: Internal APIs MAY implement retry logic and circuit breaker patterns within the client boundary layer for resilience.

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
- Client boundary implementations use Rust traits to define interfaces for easy mocking
- Retry logic and circuit breaker patterns are implemented within the client boundary layer, not scattered across business logic

<enforcement>
Claude Code MUST NOT skip or defer verification. All new client boundary code must pass trait-based pattern checks and include unit tests with mocks before acceptance.
</enforcement>