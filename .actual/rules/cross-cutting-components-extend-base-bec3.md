# Standardize Public API Testing with Keyring and HTTP Client Integration Patterns: Components Extend Base

These rules are ALWAYS ACTIVE for all public API implementations, external integrations, authentication-related components, HTTP clients making requests to external package registries, and command implementations requiring credential management or registry access.

### Rules

- **R-KEYRING-001** MAY: Components MAY extend the base HTTP client with additional caching strategies specific to their use case.
- **R-KEYRING-002** MUST: All public API client implementations import and utilize the keyring subsystem for credential management.
- **R-KEYRING-003** MUST: HTTP cache integration be present in uv-client and utilized by command implementations.
- **R-KEYRING-004** MUST: Use the uv-keyring crate's blocking module for all synchronous credential operations and ensure proper thread pool configuration.
- **R-KEYRING-005** MUST: Leverage the uv-client httpcache module as the standard HTTP caching layer, configuring appropriate TTLs based on endpoint characteristics.
- **R-KEYRING-006** MUST: All new public API integrations include threading tests similar to uv-keyring/tests/threading.rs to validate concurrent access patterns.
- **R-KEYRING-007** SHOULD: Reference common test utilities in uv-keyring/tests/common/mod.rs when writing integration tests for new API clients.
- **R-KEYRING-008** SHOULD: Follow the pattern established in commands/publish.rs and commands/tool/mod.rs for integrating authentication into command implementations.

### Verify

```bash
# Count keyring imports in non-test code
grep -r 'use.*keyring' crates/*/src/**/*.rs | grep -v test | wc -l

# Count httpcache usage in uv-client
grep -r 'httpcache' crates/uv-client/src/**/*.rs | wc -l

# Count threading tests in uv-keyring
find crates/uv-keyring/tests -name '*.rs' -exec grep -l 'thread' {} \; | wc -l

# Run integration tests with single thread to detect race conditions
cargo test --package uv-keyring --package uv-client -- --test-threads=1
```

**Accept when:**
- All public API client implementations import and utilize the keyring subsystem for credential management
- HTTP cache integration is present in uv-client and utilized by command implementations
- Threading tests exist for keyring operations and pass consistently
- Common test utilities are available and used across integration tests for authentication scenarios
- All integration tests for authentication and HTTP caching pass in CI
- Code review confirms keyring integration for new API clients
- Static analysis confirms no direct credential handling outside keyring subsystem
- Architecture review approves new public API integrations

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement. Violations block merge unless documented exceptions are approved by the architecture review board.
</enforcement>