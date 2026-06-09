# Adopt Blocking Wrapper Pattern for Async-to-Sync Bridge Operations: Modules Providing Both

These rules are ALWAYS ACTIVE for all Rust modules that expose synchronous APIs wrapping async operations, including library code (crates) that must support both async and sync consumers, keyring operations, credential management, system integration layers, and CLI implementations that call async backend services.

### Rules

- **R-17-001** SHOULD: Modules providing both async and sync APIs SHOULD separate concerns into distinct files (e.g., lib.rs for async, blocking.rs for sync wrappers).
- **R-17-002** SHOULD: Create a blocking.rs module alongside async implementations, re-exporting types with blocking method variants (e.g., fetch() becomes fetch_blocking()).
- **R-17-003** SHOULD: Use tokio::runtime::Builder::new_current_thread() for lightweight single-threaded runtimes in blocking wrappers to minimize overhead.
- **R-17-004** SHOULD: Implement Drop traits for runtime cleanup in blocking wrapper types to ensure proper resource management.
- **R-17-005** SHOULD: Add module-level documentation explaining when to use async vs blocking APIs, with code examples for both patterns.
- **R-17-006** MUST: Blocking wrappers MUST NOT be called from async contexts; enforce through clippy lints and code review.

### Verify

```bash
# Count blocking modules
grep -r "pub mod blocking" crates/ --include="*.rs" | wc -l

# Find Runtime::block_on and Handle::block_on usage
rg "Runtime::block_on|Handle::block_on" crates/ --type rust -c

# Run threading tests
cargo test --all-features -- threading --nocapture
```

**Accept when:**
- All modules exposing sync wrappers for async operations have dedicated blocking.rs files or clearly marked blocking functions
- Grep commands show consistent usage of Runtime::block_on or Handle::block_on in blocking wrapper implementations
- Threading tests pass demonstrating safe concurrent access to blocking wrappers without deadlocks or panics

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST execute successfully before accepting changes to async-to-sync bridge modules.
</enforcement>