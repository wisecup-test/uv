# Adopt Distributed Tracing for HTTP Client Operations: Authentication Proxy Related

These rules are ALWAYS ACTIVE for all HTTP client implementations and network-bound operations within the uv-client crate and related test suites, including base_client, cached_client, registry_client, and integration tests that exercise HTTP client functionality.

### Rules

- **R-TRACE-001** SHOULD: Authentication and proxy-related operations SHOULD include tracing spans with sanitized metadata (excluding credentials) to support security audit and troubleshooting.

### Verify

```bash
# Count #[instrument] macro usage in HTTP client implementations
grep -r '#\[instrument' crates/uv-client/src/ | wc -l

# Count tracing span creation patterns
grep -r 'tracing::' crates/uv-client/src/ | grep -E '(info_span|debug_span|trace_span|warn_span|error_span)' | wc -l

# Verify trace output in integration tests
cargo test --package uv-client -- --nocapture 2>&1 | grep -E '(TRACE|DEBUG|INFO)' | head -20
```

**Accept when:**
- All HTTP client public methods in base_client, cached_client, and registry_client have tracing instrumentation with appropriate span names and attributes
- Integration tests demonstrate trace output for representative operations including successful requests, error cases, cache hits/misses, and authentication flows
- Code review confirms no sensitive data (credentials, tokens) is logged in span attributes or events
- Performance benchmarks show tracing overhead is less than 5% for typical HTTP operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All HTTP client operations must be instrumented with tracing spans before code review approval. Security scanning for credential leakage in traces is mandatory.
</enforcement>