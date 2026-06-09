# Adopt Blocking Wrapper Pattern for Async-to-Sync Bridge Operations: Blocking Wrappers Not

These rules are ALWAYS ACTIVE for all Rust modules that expose synchronous APIs wrapping async operations, including library code, keyring operations, credential management, system integration layers, and CLI implementations.

### Rules

- **R-17-006** MUST_NOT: Blocking wrappers MUST NOT be called from within an async context where .await would be appropriate, to avoid unnecessary runtime overhead and prevent thread pool exhaustion and deadlocks.

### Verify

```bash
# Detect blocking wrapper modules
grep -r "pub mod blocking" crates/ --include="*.rs" | wc -l

# Count block_on usage patterns
rg "Runtime::block_on|Handle::block_on" crates/ --type rust -c

# Run threading tests to verify safe concurrent access
cargo test --all-features -- threading --nocapture
```

**Accept when:**
- All modules exposing sync wrappers for async operations have dedicated blocking.rs files or clearly marked blocking functions
- Grep commands show consistent usage of Runtime::block_on or Handle::block_on in blocking wrapper implementations
- Threading tests pass demonstrating safe concurrent access to blocking wrappers without deadlocks or panics
- No block_on calls are detected within async function bodies (enforcing R-17-006)

<enforcement>
Claude Code MUST NOT skip or defer verification. Blocking wrapper violations must be caught before merge and documented with ADR reference if exceptions are required.
</enforcement>