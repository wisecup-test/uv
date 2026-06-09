# Adopt In-Memory Caching Layer for Package Metadata and HTTP Responses: Cache Keys Derived

These rules are ALWAYS ACTIVE for all package management operations, HTTP client interactions, metadata resolution workflows, and data access patterns within the uv package manager codebase.

### Rules

- **R-CACHE-001** SHOULD: Cache keys SHOULD be derived from cryptographic digests of the cached content to ensure uniqueness and integrity.

### Verify

```bash
# Verify cache-related files and modules are present
grep -r 'cache' crates/uv-*/src/ | grep -E '(Cache|cache_|cached)' | wc -l

# Find all cache-related source files
find crates/ -name '*cache*.rs' -type f

# Verify once_map pattern usage for concurrent-safe caching
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
Clause Code MUST NOT skip or defer verification. Cache key derivation from cryptographic digests is mandatory for all in-memory caching implementations. Code review must verify appropriate cache usage in new data access patterns. Performance regression tests must measure cache hit rates. Static analysis must verify thread-safety of cache implementations.
</enforcement>