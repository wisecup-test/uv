# Adopt Blocking Wrapper Pattern for Async-to-Sync Bridge Operations: Modules That Bridge

These rules are ALWAYS ACTIVE for all Rust modules that expose synchronous APIs wrapping async operations, including library code, keyring operations, credential management, system integration layers, and CLI implementations.

### Rules

- **R-17-001** MUST: Modules that bridge async operations to synchronous APIs MUST implement dedicated blocking wrapper types or functions (e.g., blocking.rs modules).
- **R-17-006** MUST: Blocking wrappers MUST NOT be called from async contexts. Enforce through code review and clippy lints to prevent thread pool exhaustion and deadlocks.

### Verify

```bash
# Check for blocking wrapper modules
grep -r "pub mod blocking" crates/ --include="*.rs" | wc -l

# Check for Runtime::block_on or Handle::block_on usage
rg "Runtime::block_on|Handle::block_on" crates/ --type rust -c

# Run threading tests to verify safe concurrent access
cargo test --all-features -- threading --nocapture
```

**Accept when:**
- All modules exposing sync wrappers for async operations have dedicated blocking.rs files or clearly marked blocking functions
- Grep commands show consistent usage of Runtime::block_on or Handle::block_on in blocking wrapper implementations
- Threading tests pass demonstrating safe concurrent access to blocking wrappers without deadlocks or panics

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must execute successfully before accepting changes that introduce or modify async-to-sync bridge operations.
</enforcement>