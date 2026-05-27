# Adopt In-Memory Caching Layer for Package Metadata and HTTP Responses: Parsed Html Index

These rules are ALWAYS ACTIVE for all package management operations, HTTP client interactions, and metadata resolution workflows within the uv package manager codebase.

### Rules

- **R-CACHE-001** MUST: Parsed HTML index data and flat index structures MUST be cached to prevent redundant parsing of the same registry responses.

### Verify

```bash
# Check for cache-related files and patterns
grep -r 'cache' crates/uv-*/src/ | grep -E '(Cache|cache_|cached)' | wc -l

# Find all cache-related modules
find crates/ -name '*cache*.rs' -type f

# Verify once_map pattern usage
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
Clause Code MUST NOT skip or defer verification. Cache implementations MUST be reviewed for thread-safety, proper invalidation logic, and compliance with RFC 7234 cache-control directives. Performance regression tests MUST validate cache hit rates remain above established thresholds.
</enforcement>