# Adopt In-Memory Caching Layer for Package Metadata and HTTP Responses: Cache Implementations Use

These rules are ALWAYS ACTIVE for all package management operations, HTTP client interactions, metadata resolution workflows, and data access patterns within the uv package manager codebase.

### Rules

- **R-CACHE-001** SHOULD: Cache implementations SHOULD use concurrent-safe data structures (such as once_map patterns) to handle parallel operations without data races.

### Verify

```bash
# Count cache-related patterns across crates
grep -r 'cache' crates/uv-*/src/ | grep -E '(Cache|cache_|cached)' | wc -l

# Find all cache-related files
find crates/ -name '*cache*.rs' -type f

# Verify once_map usage for concurrent-safe caching
grep -r 'once_map\|OnceMap' crates/ --include='*.rs'

# Run cache-related tests
cargo test --package uv-client --test '*cache*' 2>&1 | grep -E '(test.*ok|passed)'
```

**Accept when:**
- Cache-related files and modules are present in the codebase (httpcache, auth cache, once_map)
- Grep searches for cache patterns return significant matches across multiple crates
- Cache-related tests exist and pass successfully
- Performance benchmarks show reduced network requests and improved execution time compared to no-cache baseline

<enforcement>
Clause Code MUST NOT skip or defer verification of concurrent-safe cache implementations. All new data access patterns for package metadata, HTTP responses, authentication credentials, and parsed indices MUST be reviewed for appropriate caching strategy and thread-safety guarantees.
</enforcement>