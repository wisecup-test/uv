# Adopt Structured Logging with Tracing Framework for Observability: Logging Statements Include

These rules are ALWAYS ACTIVE for all Rust crates and modules within the uv project that perform operations requiring observability, debugging, or operational monitoring.

### Rules

- **R-LOG-001** SHOULD: Logging statements SHOULD include structured fields using the field=value syntax to enable filtering, aggregation, and analysis.

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
Claude Code MUST NOT skip or defer verification of structured logging compliance. Violations must be flagged during code review and CI pipeline checks must enforce tracing patterns in production code paths.
</enforcement>