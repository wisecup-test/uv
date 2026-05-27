# Adopt Blocking Wrapper Pattern for Async-to-Sync Bridge Operations: Blocking Wrappers Preserve

These rules are ALWAYS ACTIVE for all Rust modules that expose synchronous APIs wrapping async operations, including library code, keyring operations, credential management, system integration layers, and CLI implementations.

### Rules

- **R-17-001** SHOULD: Blocking wrappers SHOULD preserve error types and semantics from the underlying async operations.

### Verify

```bash
# Detect blocking wrapper modules
grep -r "pub mod blocking" crates/ --include="*.rs" | wc -l

# Count block_on usage patterns
rg "Runtime::block_on|Handle::block_on" crates/ --type rust -c

# Run threading tests
cargo test --all-features -- threading --nocapture
```

**Accept when:**
- All modules exposing sync wrappers for async operations have dedicated blocking.rs files or clearly marked blocking functions
- Grep commands show consistent usage of Runtime::block_on or Handle::block_on in blocking wrapper implementations
- Threading tests pass demonstrating safe concurrent access to blocking wrappers without deadlocks or panics

<enforcement>
Claude Code MUST NOT skip or defer verification of blocking wrapper patterns in async-to-sync bridge operations.
</enforcement>