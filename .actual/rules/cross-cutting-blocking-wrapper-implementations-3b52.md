# Adopt Blocking Wrapper Pattern for Async-to-Sync Bridge Operations: Blocking Wrapper Implementations

These rules are ALWAYS ACTIVE for all Rust modules that expose synchronous APIs wrapping async operations, including library code (crates) that must support both async and sync consumers, keyring operations, credential management, system integration layers, and CLI implementations that call async backend services.

### Rules

- **R-17-001** MUST: Blocking wrapper implementations MUST handle runtime initialization and cleanup to prevent resource leaks.
- **R-17-006** MUST: Blocking wrappers MUST NOT be called from async contexts; enforce through clippy lints and code review to prevent thread pool exhaustion and deadlocks.

### Verify

```bash
# Detect blocking wrapper modules
grep -r "pub mod blocking" crates/ --include="*.rs" | wc -l

# Count runtime block_on usage patterns
rg "Runtime::block_on|Handle::block_on" crates/ --type rust -c

# Run threading tests to verify safe concurrent access
cargo test --all-features -- threading --nocapture
```

**Accept when:**
- All modules exposing sync wrappers for async operations have dedicated blocking.rs files or clearly marked blocking functions
- Grep commands show consistent usage of Runtime::block_on or Handle::block_on in blocking wrapper implementations
- Threading tests pass demonstrating safe concurrent access to blocking wrappers without deadlocks or panics
- No block_on calls are detected within async function contexts (verified by clippy custom lints)

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST execute successfully before accepting changes to blocking wrapper implementations. Code review MUST check for violations of R-17-006 (blocking wrappers called from async contexts).
</enforcement>