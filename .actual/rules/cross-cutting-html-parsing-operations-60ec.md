# Adopt Distributed Tracing for HTTP Client Operations: Html Parsing Operations

These rules are ALWAYS ACTIVE for all HTTP client implementations and network-bound operations within the uv-client crate, including base_client, cached_client, registry_client, and related test suites that exercise HTTP functionality.

### Rules

- **R-TRACE-001** SHOULD: HTML parsing operations SHOULD be instrumented with spans that capture parsing duration and document size for performance profiling.

### Verify

```bash
# Count #[instrument] macro usage in HTTP client code
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
- HTML parsing operations specifically include span attributes capturing parsing duration and document size

<enforcement>
Claude Code MUST NOT skip or defer verification of tracing instrumentation in HTTP client operations. All new HTTP client methods and HTML parsing code paths MUST include appropriate tracing spans before merge.
</enforcement>