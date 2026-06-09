# Adopt Modular Crate Architecture for Core Library Components: Core Functionality Organized

These rules are ALWAYS ACTIVE for all core library functionality in the uv project, including new feature development requiring shared components and refactoring of existing monolithic modules.

### Rules

- **R-MODULAR-001** MUST: Core functionality MUST be organized into separate crates based on functional domain (e.g., uv-client, uv-normalize, uv-installer).
- **R-MODULAR-002** MUST: All crates MUST follow the `uv-{domain}` naming convention consistently.
- **R-MODULAR-003** MUST: Each crate MUST have a valid Cargo.toml with clear description and appropriate dependencies.
- **R-MODULAR-004** MUST: The dependency tree MUST show no circular dependencies between internal crates.
- **R-MODULAR-005** MUST: Each crate MUST expose a clear public API through lib.rs or mod.rs with documented public interfaces.
- **R-MODULAR-006** SHOULD: New crate creation SHOULD require architecture review and maintain a minimum size threshold (e.g., 500+ LOC or 3+ modules) before extraction.
- **R-MODULAR-007** SHOULD: Establish and maintain a clear dependency hierarchy with core crates at the bottom and application crates at the top.
- **R-MODULAR-008** SHOULD: Use shared traits and types from core crates to enforce API design consistency across crates.

### Verify

```bash
# Count crates in the workspace
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
- Cargo workspace validation confirms all crates follow naming conventions

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated CI checks using cargo-deny MUST prevent circular dependencies. Code review MUST require justification for new crate creation. Violations of circular dependencies are automatically rejected by CI. New crates not following naming conventions require architecture review approval. Quarterly technical debt reviews MUST identify and prioritize boundary violations.
</enforcement>