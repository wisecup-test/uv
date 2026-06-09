# Adopt Distributed Tracing for HTTP Client Operations: Error Conditions Http

These rules are ALWAYS ACTIVE for all HTTP client implementations and network-bound operations within the uv-client crate and related test suites, including base_client, cached_client, registry_client, and integration tests that exercise HTTP client functionality.

### Rules

- **R-TRACE-001** MUST: Error conditions in HTTP operations MUST be recorded as span events or error attributes to enable failure analysis and alerting.

### Verify

```bash
# Count #[instrument] macro usage in HTTP client code
grep -r '#\[instrument' crates/uv-client/src/ | wc -l

# Count tracing span creation patterns
grep -r 'tracing::' crates/uv-client/src/ | grep -E '(info_span|debug_span|trace_span|warn_span|error_span)' | wc -l

# Verify trace output in test execution
cargo test --package uv-client -- --nocapture 2>&1 | grep -E '(TRACE|DEBUG|INFO)' | head -20
```

**Accept when:**
- All HTTP client public methods in base_client, cached_client, and registry_client have tracing instrumentation with appropriate span names and attributes
- Integration tests demonstrate trace output for representative operations including successful requests, error cases, cache hits/misses, and authentication flows
- Code review confirms no sensitive data (credentials, tokens) is logged in span attributes or events
- Performance benchmarks show tracing overhead is less than 5% for typical HTTP operations
- Error conditions are explicitly recorded as span events or error attributes in HTTP operation handlers

<enforcement>
Claude Code MUST NOT skip or defer verification of tracing instrumentation in HTTP error paths. All HTTP client methods handling error conditions MUST include explicit span event or error attribute recording before merging.
</enforcement>