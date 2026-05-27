# Adopt Modular Crate Architecture for Core Library Components: Crates Minimize Dependencies

These rules are ALWAYS ACTIVE for all core library functionality in the uv project, including new feature development requiring shared components and refactoring of existing monolithic modules.

### Rules

- **R-CRATE-001** SHOULD: Crates SHOULD minimize dependencies on other internal crates to reduce coupling.

### Verify

```bash
# Count total crates in the project
find crates/ -name 'Cargo.toml' | wc -l | grep -E '^[0-9]+$'

# Verify uv-{domain} naming convention is followed
grep -r 'name = "uv-' crates/*/Cargo.toml | wc -l

# Check dependency tree for circular dependencies
cargo tree --workspace --depth 1 | grep -v 'build-dependencies'

# Verify each crate exposes clear public API
find crates/*/src -name 'mod.rs' -o -name 'lib.rs' | xargs grep -l 'pub mod\|pub use'
```

**Accept when:**
- At least 8 separate crates exist under the crates/ directory following the uv-{domain} naming pattern
- Each crate has a valid Cargo.toml with clear description and appropriate dependencies
- Dependency tree shows no circular dependencies between internal crates
- Each crate exposes a clear public API through lib.rs or mod.rs with documented public interfaces

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated CI checks using cargo-deny MUST prevent circular dependencies. Code review checklist MUST require justification for new crate creation. Periodic architecture reviews MUST examine crate boundaries and dependencies. PRs introducing circular dependencies are automatically rejected by CI. New crates not following naming conventions require architecture review approval.
</enforcement>