# Adopt Distributed Tracing for HTTP Client Operations: Cache Hit Miss

These rules are ALWAYS ACTIVE for all HTTP client implementations and network-bound operations within the uv-client crate and related test suites, including base_client, cached_client, registry_client, and integration tests that exercise HTTP client functionality.

### Rules

- **R-TRACE-001** SHOULD: Cache hit/miss decisions in cached_client SHOULD be recorded as span attributes to enable cache effectiveness analysis.
- **R-TRACE-002** SHOULD: Use the tracing crate's `#[instrument]` macro for automatic span creation on function entry/exit, configuring skip parameters for large arguments to avoid excessive data capture.
- **R-TRACE-003** SHOULD: Create span attributes using `tracing::field` for structured data rather than string interpolation to enable efficient querying in observability backends.
- **R-TRACE-004** SHOULD: For async operations, ensure spans are properly entered/exited across await points using `tracing::Instrument` trait or async `#[instrument]` macro.
- **R-TRACE-005** SHOULD: Implement URL sanitization utilities that strip query parameters containing tokens or credentials before recording in span attributes.
- **R-TRACE-006** MUST: Do not log sensitive data (credentials, tokens) in trace spans or span attributes.

### Verify

```bash
# Count #[instrument] macros in HTTP client implementations
grep -r '#\[instrument' crates/uv-client/src/ | wc -l

# Count tracing span creation calls
grep -r 'tracing::' crates/uv-client/src/ | grep -E '(info_span|debug_span|trace_span|warn_span|error_span)' | wc -l

# Verify trace output in integration tests
cargo test --package uv-client -- --nocapture 2>&1 | grep -E '(TRACE|DEBUG|INFO)' | head -20

# Check for potential credential patterns in tracing code
grep -r 'tracing::' crates/uv-client/src/ | grep -E '(password|token|credential|secret|auth)' | grep -v 'sanitize\|strip\|redact'
```

**Accept when:**
- All HTTP client public methods in base_client, cached_client, and registry_client have tracing instrumentation with appropriate span names and attributes
- Integration tests demonstrate trace output for representative operations including successful requests, error cases, cache hits/misses, and authentication flows
- Code review confirms no sensitive data (credentials, tokens) is logged in span attributes or events
- Performance benchmarks show tracing overhead is less than 5% for typical HTTP operations
- Cache hit/miss decisions are recorded as span attributes in cached_client operations

<enforcement>
Claude Code MUST NOT skip or defer verification of tracing instrumentation presence, span attribute structure, sensitive data scanning, and performance overhead thresholds. All HTTP client methods must be reviewed for compliance with R-TRACE rules before code acceptance.
</enforcement>