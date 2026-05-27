# Adopt Serde-Based Public API Serialization with Explicit Type Contracts: Public Types Implement

These rules are ALWAYS ACTIVE for all public API types exposed through CLI interfaces, configuration files, persistent storage, network boundaries, normalization layers, and credential/authentication types across the codebase.

### Rules

- **R-SERDE-001** MUST: All public API types MUST implement explicit Serde serialization and deserialization traits (Serialize, Deserialize).
- **R-SERDE-002** MUST: Use `#[derive(Serialize, Deserialize)]` on all public API types; add `#[serde(rename_all = "snake_case")]` for consistent JSON field naming.
- **R-SERDE-003** MUST: Create dedicated modules for API types (e.g., api/, types/, contracts/) to clearly separate public contracts from internal implementation.
- **R-SERDE-004** MUST: Implement comprehensive serialization round-trip tests: serialize to format, deserialize back, assert equality.
- **R-SERDE-005** SHOULD: Consider using `#[serde(deny_unknown_fields)]` during development to catch API mismatches early, but allow unknown fields in production for forward compatibility.
- **R-SERDE-006** SHOULD: Document serialization format in type-level doc comments with examples showing JSON/TOML representation.
- **R-SERDE-007** MAY: Exception EX-001 applies to performance-critical internal types where serialization overhead is prohibitive and external exposure is guaranteed impossible.

### Verify

```bash
# Count Serde derives in public API modules
grep -r "#\[derive.*Serialize.*Deserialize" crates/uv-normalize/ crates/uv-cli/ | wc -l

# Run serialization tests
cargo test --package uv-normalize --package uv-cli -- serde

# Check for public types without Serde derives
rg "pub (struct|enum)" crates/uv-normalize/ | grep -v "#\[derive.*Serialize" || echo "All public types have Serde derives"
```

**Accept when:**
- All public API types in uv-normalize, uv-cli, uv-keyring, and uv-auth modules have Serde Serialize and Deserialize derives
- Serialization round-trip tests pass for all public API types with at least one format (JSON or TOML)
- No public API types expose raw primitives without newtype wrappers in modules designated for external contracts
- CI pipeline runs cargo test with serialization test suite and passes
- Code review checklist verification of Serde derives on new public types is complete

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API types must be checked for Serde derives before acceptance. Serialization round-trip tests must pass. Architecture review is required for any changes to existing public API serialization format.
</enforcement>