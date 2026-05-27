# Adopt Structured Logging with Tracing Framework for Observability: Modules Define Custom

These rules are ALWAYS ACTIVE for all Rust crates and modules within the uv project that perform operations requiring observability, debugging, or operational monitoring.

### Rules

- **R-TRACING-001** MAY: Modules MAY define custom tracing subscribers or layers for specialized observability requirements such as performance profiling or metrics collection.

### Verify

```bash
# Count tracing usage across crates
grep -r 'use tracing::' crates/ --include='*.rs' | wc -l

# Detect non-compliant println!/eprintln! usage in production code
grep -r 'println!\|eprintln!' crates/ --include='*.rs' | grep -v 'test\|example' | wc -l

# Count instrumented functions
grep -r '#\[instrument\]' crates/ --include='*.rs' | wc -l

# Verify tracing dependency presence
cargo tree -p uv --depth 1 | grep tracing
```

**Accept when:**
- All in-scope crates have tracing as a dependency and use tracing macros for logging
- Critical operations (HTTP requests, cache operations, installations) are instrumented with spans
- No use of println!/eprintln! for operational logging in production code paths
- Code review checklist includes verification of appropriate tracing usage and sensitive data handling

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement.
</enforcement>