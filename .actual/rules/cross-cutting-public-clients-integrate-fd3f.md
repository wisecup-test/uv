# Standardize Public API Testing with Keyring and HTTP Client Integration Patterns: Public Clients Integrate

These rules are ALWAYS ACTIVE for all public API client implementations, external integrations, authentication-related components, HTTP clients making requests to external package registries, and command implementations requiring credential management.

### Rules

- **R-PUBAPI-001** MUST: All public API clients MUST integrate with the keyring subsystem for credential storage and retrieval.
- **R-PUBAPI-002** MUST: All HTTP clients making requests to external package registries MUST use the uv-client httpcache module as the standard HTTP caching layer.
- **R-PUBAPI-003** MUST: All new public API integrations MUST include threading tests to validate concurrent access patterns.
- **R-PUBAPI-004** MUST: All authentication-related command implementations MUST follow the pattern established in commands/publish.rs and commands/tool/mod.rs.
- **R-PUBAPI-005** SHOULD: Use the uv-keyring crate's blocking module for all synchronous credential operations with proper thread pool configuration.
- **R-PUBAPI-006** SHOULD: Reference common test utilities in uv-keyring/tests/common/mod.rs when writing integration tests for new API clients.

### Verify

```bash
# Count keyring imports in non-test code
grep -r 'use.*keyring' crates/*/src/**/*.rs | grep -v test | wc -l

# Count httpcache usage in uv-client
grep -r 'httpcache' crates/uv-client/src/**/*.rs | wc -l

# Count threading tests in uv-keyring
find crates/uv-keyring/tests -name '*.rs' -exec grep -l 'thread' {} \; | wc -l

# Run integration tests with single thread to catch race conditions
cargo test --package uv-keyring --package uv-client -- --test-threads=1
```

**Accept when:**
- All public API client implementations import and utilize the keyring subsystem for credential management
- HTTP cache integration is present in uv-client and utilized by command implementations
- Threading tests exist for keyring operations and pass consistently
- Common test utilities are available and used across integration tests for authentication scenarios
- No direct credential handling exists outside the keyring subsystem in public API code
- All new public API integrations include authentication and HTTP caching integration tests

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API client implementations MUST be checked for keyring integration, HTTP cache usage, and threading test coverage before approval. Violations MUST be escalated to architecture review.
</enforcement>