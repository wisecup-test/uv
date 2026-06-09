# Adopt Serde-Based Public API Serialization with Explicit Type Contracts: Types Use Custom

These rules are ALWAYS ACTIVE for all public API types exposed through CLI interfaces, configuration files, persistent storage, network boundaries, normalization layers, and credential/authentication types across the codebase.

### Rules

- **R-SERDE-001** MAY: API types MAY use custom Serde attributes (rename, skip, flatten) to optimize wire format while maintaining type safety.
- **R-SERDE-002** MUST: All public API types in designated modules (uv-normalize, uv-cli, uv-keyring, uv-auth) MUST have #[derive(Serialize, Deserialize)] derives.
- **R-SERDE-003** SHOULD: Public API types SHOULD use #[serde(rename_all = "snake_case")] for consistent JSON field naming.
- **R-SERDE-004** SHOULD: Public API types SHOULD be organized in dedicated modules (api/, types/, contracts/) to clearly separate public contracts from internal implementation.
- **R-SERDE-005** MUST: Comprehensive serialization round-trip tests MUST be implemented for all public API types: serialize to format, deserialize back, assert equality.
- **R-SERDE-006** SHOULD: During development, types SHOULD use #[serde(deny_unknown_fields)] to catch API mismatches early; production code MAY allow unknown fields for forward compatibility.
- **R-SERDE-007** SHOULD: Serialization format SHOULD be documented in type-level doc comments with examples showing JSON/TOML representation.
- **R-SERDE-008** MUST NOT: Internal implementation types not exposed beyond module boundaries MUST NOT require Serde derives.
- **R-SERDE-009** MUST NOT: Temporary data structures used only within function scope MUST NOT require Serde derives.
- **R-SERDE-010** MUST NOT: Types used exclusively for in-memory computation without serialization MUST NOT require Serde derives.

### Verify

```bash
# Count Serde derives in public API modules
grep -r "#\[derive.*Serialize.*Deserialize" crates/uv-normalize/ crates/uv-cli/ | wc -l

# Run serialization tests
cargo test --package uv-normalize --package uv-cli -- serde

# Verify all public types have Serde derives
rg "pub (struct|enum)" crates/uv-normalize/ | grep -v "#\[derive.*Serialize" || echo "All public types have Serde derives"
```

**Accept when:**
- All public API types in uv-normalize, uv-cli, uv-keyring, and uv-auth modules have Serde Serialize and Deserialize derives
- Serialization round-trip tests pass for all public API types with at least one format (JSON or TOML)
- No public API types expose raw primitives without newtype wrappers in modules designated for external contracts
- CI pipeline runs cargo test with serialization test suite successfully
- Code review checklist verification of Serde derives on new public types passes
- No violations detected by custom linting or cargo-semver-checks for API compatibility

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API types in scope MUST have Serde derives and round-trip tests before acceptance. Violations block PR merge until resolved or exception is granted through architecture review board.
</enforcement>