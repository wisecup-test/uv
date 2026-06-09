# Adopt In-Memory Caching Layer for Package Metadata and HTTP Responses: Authentication Credentials Tokens

These rules are ALWAYS ACTIVE for all package management operations, HTTP client interactions, metadata resolution workflows, and authentication credential handling within the uv package manager codebase.

### Rules

- **R-AUTH-CACHE-001** MUST: Authentication credentials and tokens MUST be cached in memory to avoid repeated keyring lookups within a single execution context.

### Verify

```bash
# Verify cache-related files and modules are present
grep -r 'cache' crates/uv-*/src/ | grep -E '(Cache|cache_|cached)' | wc -l

# Find all cache-related source files
find crates/ -name '*cache*.rs' -type f

# Verify once_map pattern usage for concurrent-safe caching
grep -r 'once_map\|OnceMap' crates/ --include='*.rs'

# Verify cache-related tests pass
cargo test --package uv-client --test '*cache*' 2>&1 | grep -E '(test.*ok|passed)'
```

**Accept when:**
- Cache-related files and modules are present in the codebase (httpcache, auth cache, once_map)
- Grep searches for cache patterns return significant matches across multiple crates
- Cache-related tests exist and pass successfully
- Performance benchmarks show reduced network requests and improved execution time compared to no-cache baseline
- Authentication credential caching is implemented using thread-safe concurrent data structures

<enforcement>
Clause R-AUTH-CACHE-001 MUST be verified during code review for all authentication and credential handling code paths. Violations require addition of in-memory caching for repeated credential lookups. Performance regression tests MUST validate cache hit rates. Security team review is REQUIRED for any caching of sensitive authentication data. Claude Code MUST NOT skip or defer verification of this rule.
</enforcement>