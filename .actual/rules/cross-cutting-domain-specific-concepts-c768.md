# Adopt Newtype Pattern for Domain-Specific Type Safety: Domain Specific Concepts

These rules are ALWAYS ACTIVE for all data modeling decisions in the codebase. All new domain types MUST follow the newtype pattern unless explicitly exempted.

### Rules

- **R-NEWTYPE-001** MUST: Domain-specific concepts with validation rules or semantic meaning MUST be represented as newtype wrappers around primitive types rather than using primitives directly.
- **R-NEWTYPE-002** MUST: Newtype struct fields MUST be private and only accessible through validated constructors.
- **R-NEWTYPE-003** SHOULD: Use `#[derive(Debug, Clone, PartialEq, Eq)]` as baseline traits for most newtype wrappers.
- **R-NEWTYPE-004** SHOULD: For serialization, apply `#[serde(transparent)]` to maintain JSON compatibility with the inner type.
- **R-NEWTYPE-005** SHOULD: Implement `TryFrom<Primitive>` for validated construction and provide an `as_str()`/`as_inner()` method for safe access.
- **R-NEWTYPE-006** SHOULD: Implement Display trait for user-facing types and keep Debug for development diagnostics.
- **R-NEWTYPE-007** SHOULD: Group related newtype definitions in dedicated modules to maintain organization.

### Verify

```bash
# Check for domain-specific types not using newtype pattern
grep -r 'pub struct.*Name.*String' crates/ | grep -v 'pub struct.*Name(String)' | wc -l

# Check for type aliases used instead of newtypes
rg 'type\s+\w+Name\s*=\s*String' crates/ --count

# Check for public fields in structs (should be private)
cargo clippy -- -W clippy::pub_underscore_fields
```

**Accept when:**
- All domain-specific name types are implemented as newtype wrappers, not type aliases
- Newtype struct fields are private and only accessible through validated constructors
- Clippy passes with no warnings about public underscore fields or missing encapsulation
- New domain-specific identifiers (names, IDs, keys) follow the newtype pattern
- Validated string types with format requirements use newtype wrappers
- Numeric types with semantic meaning or constraints use newtype wrappers

<enforcement>
Claude Code MUST NOT skip or defer verification. All domain-specific types introduced or modified MUST be checked against these rules before acceptance.
</enforcement>