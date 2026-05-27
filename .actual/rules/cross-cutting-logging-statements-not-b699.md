# Adopt Structured Logging with Tracing Framework for Observability: Logging Statements Not

These rules are ALWAYS ACTIVE for all Rust crates and modules within the uv project that perform operations requiring observability, debugging, or operational monitoring.

### Rules

- **R-LOG-001** MUST_NOT: Logging statements MUST NOT include sensitive information such as authentication tokens, passwords, or personally identifiable information in plain text.

### Verify

```bash
# Count tracing usage across crates
grep -r 'use tracing::' crates/ --include='*.rs' | wc -l

# Count non-test println!/eprintln! usage
grep -r 'println!\|eprintln!' crates/ --include='*.rs' | grep -v 'test\|example' | wc -l

# Count instrumented functions
grep -r '#\[instrument\]' crates/ --include='*.rs' | wc -l

# Verify tracing dependency
cargo tree -p uv --depth 1 | grep tracing
```

**Accept when:**
- All in-scope crates have tracing as a dependency and use tracing macros for logging
- Critical operations (HTTP requests, cache operations, installations) are instrumented with spans
- No use of println!/eprintln! for operational logging in production code paths
- Code review checklist includes verification of appropriate tracing usage and sensitive data handling
- No sensitive information (tokens, credentials, PII) appears in log statements

<enforcement>
Claude Code MUST NOT skip or defer verification of logging statements for sensitive data exposure. All logging must be reviewed for compliance with R-LOG-001 before acceptance.
</enforcement>