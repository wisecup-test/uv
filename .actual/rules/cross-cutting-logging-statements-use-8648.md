# Adopt Structured Logging with Tracing Framework for Observability: Logging Statements Use

These rules are ALWAYS ACTIVE for all Rust crates and modules within the uv project that perform operations requiring observability, debugging, or operational monitoring.

### Rules

- **R-LOG-001** MUST: Logging statements MUST use appropriate tracing levels: `error!` for errors, `warn!` for warnings, `info!` for significant events, `debug!` for detailed diagnostics, and `trace!` for verbose execution details.

### Verify

```bash
# Count tracing imports across crates
grep -r 'use tracing::' crates/ --include='*.rs' | wc -l

# Detect non-compliant println!/eprintln! usage in production code
grep -r 'println!\|eprintln!' crates/ --include='*.rs' | grep -v 'test\|example' | wc -l

# Count instrumented functions
grep -r '#\[instrument\]' crates/ --include='*.rs' | wc -l

# Verify tracing dependency in workspace
cargo tree -p uv --depth 1 | grep tracing
```

**Accept when:**
- All in-scope crates have tracing as a dependency and use tracing macros for logging
- Critical operations (HTTP requests, cache operations, installations) are instrumented with spans
- No use of `println!`/`eprintln!` for operational logging in production code paths
- Code review checklist includes verification of appropriate tracing usage and sensitive data handling

<enforcement>
Claude Code MUST NOT skip or defer verification. All logging statements in scope MUST comply with R-LOG-001. Violations detected by grep patterns or code review MUST block merge.
</enforcement>