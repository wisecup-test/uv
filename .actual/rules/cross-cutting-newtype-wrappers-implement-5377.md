# Adopt Newtype Pattern for Domain-Specific Type Safety: Newtype Wrappers Implement

These rules are ALWAYS ACTIVE for all data modeling decisions in the codebase. All new domain types MUST follow the newtype pattern unless explicitly exempted.

### Rules

- **R-NEWTYPE-001** SHOULD: Newtype wrappers SHOULD implement standard traits (Display, Debug, Serialize, Deserialize) to maintain ergonomics.
- **R-NEWTYPE-002** MUST: All domain-specific identifiers (names, IDs, keys) MUST use newtype wrappers instead of primitive types.
- **R-NEWTYPE-003** MUST: Validated string types with format requirements MUST be implemented as newtype wrappers.
- **R-NEWTYPE-004** MUST: Numeric types with semantic meaning or constraints (sizes, counts, indices) MUST use newtype wrappers.
- **R-NEWTYPE-005** MUST: Newtype struct fields MUST be private and only accessible through validated constructors.
- **R-NEWTYPE-006** SHOULD: Use #[derive(Debug, Clone, PartialEq, Eq)] as baseline traits for most newtype wrappers.
- **R-NEWTYPE-007** SHOULD: For serialization, apply #[serde(transparent)] to maintain JSON compatibility with the inner type.
- **R-NEWTYPE-008** SHOULD: Implement TryFrom<Primitive> for validated construction and provide an as_str()/as_inner() method for safe access.
- **R-NEWTYPE-009** MAY: Exceptions EX-001 (test-only code) and EX-002 (FFI boundaries with isolated conversion layer) are permitted with documentation.

### Verify

```bash
# Check for domain types using primitives instead of newtype wrappers
grep -r 'pub struct.*Name.*String' crates/ | grep -v 'pub struct.*Name(String)' | wc -l

# Check for type aliases used as domain types
rg 'type\s+\w+Name\s*=\s*String' crates/ --count

# Check for public fields in newtype structs
cargo clippy -- -W clippy::pub_underscore_fields
```

**Accept when:**
- All domain-specific name types are implemented as newtype wrappers, not type aliases
- Newtype struct fields are private and only accessible through validated constructors
- Clippy passes with no warnings about public underscore fields or missing encapsulation
- New domain types include Display, Debug, and appropriate serde attributes
- TryFrom implementations exist for validated construction of domain types

<enforcement>
Claude Code MUST NOT skip or defer verification. All new domain types MUST be checked against these rules before acceptance. Violations require either pattern compliance or documented exception approval.
</enforcement>