# Adopt Distributed Tracing for HTTP Client Operations: Tracing Instrumentation Not

These rules are ALWAYS ACTIVE for all HTTP client implementations and network-bound operations within the uv-client crate and related test suites, including base_client, cached_client, registry_client, and integration tests (audit, auth, build, proxy).

### Rules

- **R-48-001** MUST_NOT: Tracing instrumentation MUST NOT log sensitive data including authentication tokens, passwords, or API keys in span attributes or events.

### Verify

```bash
# Count #[instrument] macro usage in HTTP client code
grep -r '#\[instrument' crates/uv-client/src/ | wc -l

# Count tracing span creation patterns
grep -r 'tracing::' crates/uv-client/src/ | grep -E '(info_span|debug_span|trace_span|warn_span|error_span)' | wc -l

# Verify trace output in integration tests
cargo test --package uv-client -- --nocapture 2>&1 | grep -E '(TRACE|DEBUG|INFO)' | head -20

# Scan for potential credential patterns in tracing statements
grep -r 'tracing::' crates/uv-client/src/ | grep -E '(token|password|api_key|credential|secret)' | grep -v 'sanitize\|strip\|redact'
```

**Accept when:**
- All HTTP client public methods in base_client, cached_client, and registry_client have tracing instrumentation with appropriate span names and attributes
- Integration tests demonstrate trace output for representative operations including successful requests, error cases, cache hits/misses, and authentication flows
- Code review confirms no sensitive data (credentials, tokens) is logged in span attributes or events
- Performance benchmarks show tracing overhead is less than 5% for typical HTTP operations
- Automated scanning detects no unredacted credential patterns in tracing statements

<enforcement>
Claude Code MUST NOT skip or defer verification of tracing instrumentation presence and sensitive data protection in HTTP client operations. All new HTTP client methods MUST include tracing instrumentation and MUST NOT expose credentials, tokens, or API keys in span attributes or events.
</enforcement>