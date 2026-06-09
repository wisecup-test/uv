# Standardize Public API Testing with Keyring and HTTP Client Integration Patterns: Public Facing Commands

These rules are ALWAYS ACTIVE for all public API implementations, external integrations, authentication-related components, and public-facing commands (publish, install, tool management) within the codebase.

### Rules

- **R-PUBAPI-001** SHOULD: Public-facing commands (publish, install, tool management) SHOULD use standardized authentication flows through the keyring subsystem.
- **R-PUBAPI-002** MUST: All HTTP clients making requests to external package registries MUST integrate with the keyring subsystem for credential management.
- **R-PUBAPI-003** MUST: Authentication and credential management for publish operations MUST use the uv-keyring crate's blocking module for all synchronous credential operations.
- **R-PUBAPI-004** SHOULD: Tool installation and management commands requiring registry access SHOULD leverage the uv-client httpcache module as the standard HTTP caching layer.
- **R-PUBAPI-005** SHOULD: Project dependency resolution requiring authenticated API access SHOULD implement HTTP cache with appropriate TTLs based on endpoint characteristics.
- **R-PUBAPI-006** MUST: New public API integrations MUST include threading tests similar to uv-keyring/tests/threading.rs to validate concurrent access patterns.
- **R-PUBAPI-007** SHOULD: Integration tests for new API clients SHOULD reference common test utilities in uv-keyring/tests/common/mod.rs.
- **R-PUBAPI-008** MUST: All new public API client implementations MUST NOT bypass keyring integration without documented exception (EXC-001 or EXC-002).

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
- No hardcoded credentials or non-keyring credential storage is detected in public API code
- All new public API integrations include integration tests for authentication flows

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline MUST fail if integration tests for authentication are missing or failing. Code review MUST block merge if keyring integration is bypassed without documented exception. Security scanning MUST flag hardcoded credentials or non-keyring credential storage. Architecture review MUST escalate patterns that deviate from established standards.
</enforcement>