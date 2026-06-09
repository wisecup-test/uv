# Adopt Serde-Based Public API Serialization with Explicit Type Contracts: Public Types Defined

These rules are ALWAYS ACTIVE for all public API types exposed through CLI interfaces, configuration files, persistent storage, network boundaries, normalization layers, and credential/authentication systems.

### Rules

- **R-SERDE-001** SHOULD: Public API types SHOULD be defined in dedicated modules (e.g., uv-normalize, uv-cli) separate from internal implementation.
- **R-SERDE-002** MUST: All public API types in designated modules MUST have `#[derive(Serialize, Deserialize)]` attributes.
- **R-SERDE-003** SHOULD: Public API types SHOULD use `#[serde(rename_all = "snake_case")]` for consistent JSON field naming.
- **R-SERDE-004** SHOULD: Public API types SHOULD avoid exposing raw primitives without newtype wrappers in modules designated for external contracts.
- **R-SERDE-005** SHOULD: Serialization round-trip tests SHOULD be implemented for all public API types with at least one format (JSON or TOML).
- **R-SERDE-006** SHOULD: Type-level doc comments SHOULD document serialization format with examples showing JSON/TOML representation.
- **R-SERDE-007** MAY: Use `#[serde(deny_unknown_fields)]` during development to catch API mismatches early, but allow unknown fields in production for forward compatibility.

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
- Code review checklist verification confirms Serde derives on new public types
- CI pipeline runs cargo test with serialization test suite successfully

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API types must be validated against these rules before acceptance. Violations must be resolved or documented as exceptions with architecture review board approval.
</enforcement>