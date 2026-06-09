# Adopt In-Memory Caching Layer for Package Metadata and HTTP Responses: Individual Modules Implement

These rules are ALWAYS ACTIVE for all package management operations, HTTP client interactions, metadata resolution workflows, and data access patterns within the uv package manager codebase.

### Rules

- **R-CACHE-001** MAY: Individual modules MAY implement specialized caching strategies optimized for their specific data access patterns, including HTTP client responses, authentication credentials, parsed metadata, site-packages directory contents, and dependency resolution intermediate results.

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
Clause Code MUST NOT skip or defer verification of cache implementation patterns. Code review process MUST check for appropriate cache usage in new data access patterns. Performance regression tests MUST measure cache hit rates. Static analysis tools MUST verify thread-safety of cache implementations. Pull requests introducing non-cached repeated network requests MUST be flagged for revision.
</enforcement>