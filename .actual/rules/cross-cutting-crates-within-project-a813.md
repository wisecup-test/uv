# Adopt Structured Logging with Tracing Framework for Observability: Crates Within Project

These rules are ALWAYS ACTIVE for all Rust crates and modules within the uv project that perform operations requiring observability, debugging, or operational monitoring.

### Rules

- **R-TRACING-001** MUST: All crates within the uv project MUST use the tracing framework (tracing crate) for logging and observability rather than traditional logging macros.

### Verify

```bash
# Count tracing imports across crates
grep -r 'use tracing::' crates/ --include='*.rs' | wc -l

# Count println!/eprintln! usage outside tests
grep -r 'println!\|eprintln!' crates/ --include='*.rs' | grep -v 'test\|example' | wc -l

# Count #[instrument] attribute usage
grep -r '#\[instrument\]' crates/ --include='*.rs' | wc -l

# Verify tracing dependency in workspace
cargo tree -p uv --depth 1 | grep tracing
```

**Accept when:**
- All in-scope crates have tracing as a dependency and use tracing macros for logging
- Critical operations (HTTP requests, cache operations, installations) are instrumented with spans
- No use of println!/eprintln! for operational logging in production code paths
- Code review checklist includes verification of appropriate tracing usage and sensitive data handling

<enforcement>
Claude Code MUST NOT skip or defer verification. All new logging code must use the tracing framework. Violations in production code paths must be caught during code review and CI checks.
</enforcement>