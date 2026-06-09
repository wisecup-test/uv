# Standardize Public API Testing with Keyring and HTTP Client Integration Patterns: Common Test Utilities

These rules are ALWAYS ACTIVE for all public API implementations, external integrations, authentication-related components, HTTP clients making requests to external package registries, and their corresponding test suites.

### Rules

- **R-KEYRING-001** SHOULD: Common test utilities SHOULD be provided for keyring and HTTP client testing to ensure consistent test patterns across the codebase.

### Verify

```bash
# Verify keyring usage across public API implementations
grep -r 'use.*keyring' crates/*/src/**/*.rs | grep -v test | wc -l

# Verify HTTP cache integration in uv-client
grep -r 'httpcache' crates/uv-client/src/**/*.rs | wc -l

# Verify threading tests exist for keyring operations
find crates/uv-keyring/tests -name '*.rs' -exec grep -l 'thread' {} \; | wc -l

# Run integration tests with single thread to catch race conditions
cargo test --package uv-keyring --package uv-client -- --test-threads=1
```

**Accept when:**
- All public API client implementations import and utilize the keyring subsystem for credential management
- HTTP cache integration is present in uv-client and utilized by command implementations
- Threading tests exist for keyring operations and pass consistently
- Common test utilities are available and used across integration tests for authentication scenarios

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API implementations MUST demonstrate keyring integration and HTTP cache usage. Threading tests MUST pass before accepting changes to authentication or HTTP client code.
</enforcement>