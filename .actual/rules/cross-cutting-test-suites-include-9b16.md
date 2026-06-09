# Adopt Blocking Wrapper Pattern for Async-to-Sync Bridge Operations: Test Suites Include

These rules are ALWAYS ACTIVE for all Rust modules that expose synchronous APIs wrapping async operations, including library code (crates) that must support both async and sync consumers, keyring operations, credential management, system integration layers, and command-line interface implementations.

### Rules

- **R-17-001** SHOULD: Test suites SHOULD include threading tests to verify blocking wrapper behavior under concurrent access patterns.

### Verify

```bash
# Detect blocking wrapper modules
grep -r "pub mod blocking" crates/ --include="*.rs" | wc -l

# Count blocking wrapper implementations
rg "Runtime::block_on|Handle::block_on" crates/ --type rust -c

# Run threading tests
cargo test --all-features -- threading --nocapture
```

**Accept when:**
- All modules exposing sync wrappers for async operations have dedicated blocking.rs files or clearly marked blocking functions
- Grep commands show consistent usage of Runtime::block_on or Handle::block_on in blocking wrapper implementations
- Threading tests pass demonstrating safe concurrent access to blocking wrappers without deadlocks or panics

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated CI checks using grep/ripgrep MUST detect blocking wrapper patterns in new modules. Code review MUST require blocking wrapper implementation for any new async-to-sync boundaries. Clippy custom lints MUST detect block_on calls within async functions. CI pipeline MUST fail if new async modules lack corresponding blocking wrappers when exposing public APIs.
</enforcement>