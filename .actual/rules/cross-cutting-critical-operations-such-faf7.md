# Adopt Structured Logging with Tracing Framework for Observability: Critical Operations Such

These rules are ALWAYS ACTIVE for all Rust crates and modules within the uv project that perform operations requiring observability, debugging, or operational monitoring, including core operational modules (uv-installer, uv-client, uv-cache-info, uv-auth, uv-metadata), command implementations, HTTP client operations, file system operations, and package installation workflows.

### Rules

- **R-TRACING-001** SHOULD: Critical operations such as HTTP requests, cache operations, file I/O, and package installations SHOULD be wrapped in named spans with relevant contextual fields.

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
Claude Code MUST NOT skip or defer verification. Violations are caught by automated CI checks using grep patterns, code review checklists, and periodic architecture audits. Code review blocks merge if logging does not follow tracing patterns. Violations are documented in technical debt backlog for remediation.
</enforcement>