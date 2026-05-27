# Adopt Serde-Based Public API Serialization with Explicit Type Contracts: Public Types Use

These rules are ALWAYS ACTIVE for all public API types exposed through CLI interfaces, configuration files, persistent storage, network boundaries, normalization layers, and credential/authentication types.

### Rules

- **R-SERDE-001** MUST: Public API types MUST use newtype patterns or dedicated structs rather than exposing raw primitives to enable future evolution.

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

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API types must be checked for Serde derives and serialization round-trip compatibility before acceptance.
</enforcement>