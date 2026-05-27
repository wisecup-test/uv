# Adopt Modular Crate Architecture for Core Library Components: Crates Published Independently

These rules are ALWAYS ACTIVE for all core library functionality in the uv project, new feature development requiring shared components, refactoring of existing monolithic modules, and test infrastructure and common test utilities.

### Rules

- **R-CRATE-001** MAY: Crates MAY be published independently to crates.io if they provide reusable functionality beyond the project.
- **R-CRATE-002** MUST: All crates MUST follow the `uv-{domain}` naming convention consistently.
- **R-CRATE-003** MUST: Each crate MUST have a valid Cargo.toml with clear description and appropriate dependencies.
- **R-CRATE-004** MUST: Crate boundaries MUST maintain a clear dependency hierarchy with core crates at the bottom and application crates at the top.
- **R-CRATE-005** MUST: No circular dependencies between internal crates are permitted.
- **R-CRATE-006** SHOULD: Each crate SHOULD expose a clear public API through lib.rs or mod.rs with documented public interfaces.
- **R-CRATE-007** SHOULD: New crate creation SHOULD require architecture review and maintain a minimum size threshold (e.g., 500+ LOC or 3+ modules).
- **R-CRATE-008** SHOULD: API design patterns SHOULD be consistent across crates using shared traits and types from core crates.

### Verify

```bash
# Count total crates
find crates/ -name 'Cargo.toml' | wc -l | grep -E '^[0-9]+$'

# Verify uv-{domain} naming convention
grep -r 'name = "uv-' crates/*/Cargo.toml | wc -l

# Check dependency tree for circular dependencies
cargo tree --workspace --depth 1 | grep -v 'build-dependencies'

# Verify public API exposure
find crates/*/src -name 'mod.rs' -o -name 'lib.rs' | xargs grep -l 'pub mod\|pub use'
```

**Accept when:**
- At least 8 separate crates exist under the crates/ directory following the uv-{domain} naming pattern
- Each crate has a valid Cargo.toml with clear description and appropriate dependencies
- Dependency tree shows no circular dependencies between internal crates
- Each crate exposes a clear public API through lib.rs or mod.rs with documented public interfaces

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated CI checks using cargo-deny MUST prevent circular dependencies. Code review checklist MUST require justification for new crate creation. PRs introducing circular dependencies are automatically rejected by CI. New crates not following naming conventions require architecture review approval.
</enforcement>