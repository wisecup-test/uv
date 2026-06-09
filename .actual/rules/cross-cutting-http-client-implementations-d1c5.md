# Adopt Distributed Tracing for HTTP Client Operations: Http Client Implementations

These rules are ALWAYS ACTIVE for all HTTP client implementations and network-bound operations within the uv-client crate and related test suites.

### Rules

- **R-HTTP-001** MUST: All HTTP client implementations (base_client, cached_client, registry_client) MUST instrument public methods with tracing spans that capture request context including URL, method, and relevant metadata.

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
Claude Code MUST NOT skip or defer verification of tracing instrumentation in HTTP client implementations. All public methods must be instrumented before code review approval.
</enforcement>