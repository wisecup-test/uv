# Standardize Public API Testing with Keyring and HTTP Client Integration Patterns: Integration Tests External

These rules are ALWAYS ACTIVE for all HTTP clients making requests to external package registries, authentication and credential management for publish operations, tool installation and management commands requiring registry access, project dependency resolution requiring authenticated API access, and HTTP cache implementations for API responses.

### Rules

- **R-EX-001** MUST: Integration tests for external APIs MUST cover authentication scenarios, threading behavior, and cache interactions.

### Verify

```bash
# Verify keyring integration across public API implementations
grep -r 'use.*keyring' crates/*/src/**/*.rs | grep -v test | wc -l

# Verify HTTP cache integration in uv-client
grep -r 'httpcache' crates/uv-client/src/**/*.rs | wc -l

# Verify threading tests exist for keyring operations
find crates/uv-keyring/tests -name '*.rs' -exec grep -l 'thread' {} \; | wc -l

# Run integration tests with single-threaded execution to catch race conditions
cargo test --package uv-keyring --package uv-client -- --test-threads=1
```

**Accept when:**
- All public API client implementations import and utilize the keyring subsystem for credential management
- HTTP cache integration is present in uv-client and utilized by command implementations
- Threading tests exist for keyring operations and pass consistently
- Common test utilities are available and used across integration tests for authentication scenarios
- Integration tests cover authentication scenarios, threading behavior, and cache interactions

<enforcement>
Claude Code MUST NOT skip or defer verification. All integration tests for external APIs must pass with authentication, threading, and cache coverage before code review approval.
</enforcement>