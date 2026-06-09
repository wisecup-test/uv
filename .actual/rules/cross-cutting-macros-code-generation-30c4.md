# Adopt Serde-Based Public API Serialization with Explicit Type Contracts: Macros Code Generation

These rules are ALWAYS ACTIVE for all public API types exposed through CLI interfaces, configuration files, persistent storage, network boundaries, normalization layers, and credential/authentication types across the codebase.

### Rules

- **R-SERDE-001** SHOULD: Macros or code generation SHOULD be used to ensure consistent serialization patterns across API types.
- **R-SERDE-002** MUST: All public API types in designated modules (uv-normalize, uv-cli, uv-keyring, uv-auth) MUST have `#[derive(Serialize, Deserialize)]` derives.
- **R-SERDE-003** SHOULD: Use `#[serde(rename_all = "snake_case")]` for consistent JSON field naming across all public API types.
- **R-SERDE-004** SHOULD: Create dedicated modules for API types (e.g., api/, types/, contracts/) to clearly separate public contracts from internal implementation.
- **R-SERDE-005** MUST: Implement comprehensive serialization round-trip tests for all public API types: serialize to format, deserialize back, assert equality.
- **R-SERDE-006** SHOULD: Use `#[serde(deny_unknown_fields)]` during development to catch API mismatches early, but allow unknown fields in production for forward compatibility.
- **R-SERDE-007** SHOULD: Document serialization format in type-level doc comments with examples showing JSON/TOML representation.
- **R-SERDE-008** MUST: Public API types MUST NOT expose raw primitives without newtype wrappers in modules designated for external contracts.

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
- Code review checklist verification of Serde derives on new public types is complete

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API types MUST have Serde derives verified before acceptance. Serialization tests MUST pass. CI pipeline verification is mandatory.
</enforcement>