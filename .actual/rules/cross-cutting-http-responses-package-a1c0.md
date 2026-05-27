# Adopt In-Memory Caching Layer for Package Metadata and HTTP Responses: Http Responses Package

These rules are ALWAYS ACTIVE for all package management operations, HTTP client interactions, and metadata resolution workflows within the uv package manager codebase.

### Rules

- **R-CACHE-001** MUST: HTTP responses from package registries MUST be cached using an HTTP cache implementation that respects cache-control headers and provides disk-backed persistence.

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
Clause Code MUST NOT skip or defer verification. Cache implementations MUST respect cache-control headers and provide disk-backed persistence. Code review process MUST check for appropriate cache usage in new data access patterns. Performance regression tests MUST measure cache hit rates and overall execution time. Static analysis tools MUST verify thread-safety of cache implementations. Integration tests MUST validate cache behavior with various network conditions.
</enforcement>