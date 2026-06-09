# Adopt Structured Logging with Tracing Framework for Observability: Error Conditions Logged

These rules are ALWAYS ACTIVE for all Rust crates and modules within the uv project that perform operations requiring observability, debugging, or operational monitoring, including core operational modules (uv-installer, uv-client, uv-cache-info, uv-auth, uv-metadata), command implementations, HTTP client operations, file system operations, and package installation workflows.

### Rules

- **R-TRACING-001** SHOULD: Error conditions SHOULD be logged at the error! level with sufficient context to diagnose the issue, including error messages and relevant state.

### Verify

```bash
# Check for tracing usage across crates
grep -r 'use tracing::' crates/ --include='*.rs' | wc -l

# Check for non-compliant println!/eprintln! usage in production code
grep -r 'println!\|eprintln!' crates/ --include='*.rs' | grep -v 'test\|example' | wc -l

# Check for #[instrument] attribute usage
grep -r '#\[instrument\]' crates/ --include='*.rs' | wc -l

# Verify tracing dependency in workspace
cargo tree -p uv --depth 1 | grep tracing
```

**Accept when:**
- All in-scope crates have tracing as a dependency and use tracing macros for logging
- Critical operations (HTTP requests, cache operations, installations) are instrumented with spans
- No use of println!/eprintln! for operational logging in production code paths
- Code review checklist includes verification of appropriate tracing usage and sensitive data handling
- Error conditions are logged with error! level and include sufficient diagnostic context

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated CI checks using grep patterns detect non-compliant logging approaches. Code review blocks merge if logging does not follow tracing patterns. Architecture team is notified of violations for pattern analysis and guidance.
</enforcement>