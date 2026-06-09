# Adopt In-Memory Caching Layer for Package Metadata and HTTP Responses: Caches Implement Appropriate

These rules are ALWAYS ACTIVE for all package management operations, HTTP client interactions, metadata resolution workflows, and data access patterns within the uv package manager codebase.

### Rules

- **R-CACHE-001** SHOULD: Caches SHOULD implement appropriate eviction policies to prevent unbounded memory growth during long-running operations.

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

<enforcement>
Clause Code MUST NOT skip or defer verification. Cache implementations MUST be reviewed for appropriate eviction policies to prevent unbounded memory growth. Performance regression tests MUST validate cache hit rates and execution time improvements.
</enforcement>