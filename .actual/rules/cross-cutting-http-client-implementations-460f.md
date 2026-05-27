# Standardize Public API Testing with Keyring and HTTP Client Integration Patterns: Http Client Implementations

These rules are ALWAYS ACTIVE for all HTTP client implementations, external integrations, and authentication-related components within the codebase.

### Rules

- **R-HTTP-001** MUST: HTTP client implementations MUST implement caching mechanisms for external API responses to minimize network overhead.

### Verify

```bash
# Verify keyring integration across codebase
grep -r 'use.*keyring' crates/*/src/**/*.rs | grep -v test | wc -l

# Verify httpcache usage in uv-client
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
Claude Code MUST NOT skip or defer verification. All HTTP client implementations must demonstrate caching mechanisms and keyring integration before acceptance.
</enforcement>