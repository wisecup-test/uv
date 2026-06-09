# Adopt Serde-Based Public API Serialization with Explicit Type Contracts: Domain Specific Validation

These rules are ALWAYS ACTIVE for all public API types exposed through CLI interfaces, configuration files, persistent storage, network boundaries, normalization layers, and credential/authentication types across uv-cli, uv-normalize, uv-keyring, and uv-auth modules.

### Rules

- **R-SERDE-001** MUST: All public API types in designated modules (uv-cli, uv-normalize, uv-keyring, uv-auth) MUST have `#[derive(Serialize, Deserialize)]` attributes from the Serde crate.
- **R-SERDE-002** SHOULD: Domain-specific validation logic SHOULD be encapsulated within the type definition (e.g., PackageName, ExtraName validation) rather than deferred to deserialization hooks.
- **R-SERDE-003** SHOULD: Public API types SHOULD use `#[serde(rename_all = "snake_case")]` for consistent JSON field naming conventions.
- **R-SERDE-004** SHOULD: Public API types SHOULD avoid exposing raw primitives (String, i32, etc.) without newtype wrappers in modules designated for external contracts.
- **R-SERDE-005** MUST: All public API types MUST have comprehensive serialization round-trip tests that serialize to format, deserialize back, and assert equality.
- **R-SERDE-006** MAY: Performance-critical internal types where serialization overhead is prohibitive and external exposure is guaranteed impossible MAY be exempted (EX-001) with documented justification.

### Verify

```bash
# Count Serde derives in public API modules
grep -r "#\[derive.*Serialize.*Deserialize" crates/uv-normalize/ crates/uv-cli/ crates/uv-keyring/ crates/uv-auth/ | wc -l

# Run serialization test suite
cargo test --package uv-normalize --package uv-cli --package uv-keyring --package uv-auth -- serde

# Verify all public types have Serde derives
rg "pub (struct|enum)" crates/uv-normalize/ crates/uv-cli/ crates/uv-keyring/ crates/uv-auth/ | grep -v "#\[derive.*Serialize" || echo "All public types have Serde derives"

# Check for raw primitives in public API modules
rg "pub (fn|const|static).*: (String|i32|i64|u32|u64|bool)" crates/uv-normalize/ crates/uv-cli/ | grep -v "newtype\|wrapper" || echo "No raw primitives exposed"
```

**Accept when:**
- All public API types in uv-normalize, uv-cli, uv-keyring, and uv-auth modules have Serde Serialize and Deserialize derives
- Serialization round-trip tests pass for all public API types with at least one format (JSON or TOML)
- No public API types expose raw primitives without newtype wrappers in modules designated for external contracts
- CI pipeline runs cargo test with serialization test suite and all tests pass
- Code review checklist verification confirms Serde derives on new public types

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API types in scope MUST have Serde derives and round-trip tests before acceptance. Violations block PR merge until remediated or formally exempted via exception process.
</enforcement>