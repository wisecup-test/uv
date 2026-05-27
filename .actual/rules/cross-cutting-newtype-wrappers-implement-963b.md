# Adopt Newtype Pattern for Domain-Specific Type Safety: Newtype Wrappers Implement

These rules are ALWAYS ACTIVE for all data modeling decisions in the codebase. All new domain types MUST follow the newtype pattern unless explicitly exempted.

### Rules

- **R-NEWTYPE-001** MUST: Implement domain-specific identifiers (names, IDs, keys) as newtype wrappers, not type aliases or primitives.
- **R-NEWTYPE-002** MUST: Keep newtype struct fields private and only accessible through validated constructors.
- **R-NEWTYPE-003** MUST: Apply `#[derive(Debug, Clone, PartialEq, Eq)]` as baseline traits for newtype wrappers.
- **R-NEWTYPE-004** MUST: Implement `TryFrom<Primitive>` for validated construction of newtype wrappers.
- **R-NEWTYPE-005** MUST: Provide `as_str()` or `as_inner()` methods for safe read-only access to the wrapped value.
- **R-NEWTYPE-006** SHOULD: Apply `#[serde(transparent)]` to newtype wrappers for serialization compatibility.
- **R-NEWTYPE-007** SHOULD: Implement `Display` trait for user-facing types and keep `Debug` for development diagnostics.
- **R-NEWTYPE-008** MAY: Implement `Deref` to the inner type when read-only access patterns are common and safe.

### Verify

```bash
# Check for domain types using type aliases instead of newtype wrappers
rg 'type\s+\w+Name\s*=\s*String' crates/ --count

# Check for newtype structs with public fields (should be zero)
grep -r 'pub struct.*Name.*String' crates/ | grep -v 'pub struct.*Name(String)' | wc -l

# Run clippy to detect public underscore fields and missing encapsulation
cargo clippy -- -W clippy::pub_underscore_fields
```

**Accept when:**
- All domain-specific name types are implemented as newtype wrappers, not type aliases
- Newtype struct fields are private and only accessible through validated constructors
- Clippy passes with no warnings about public underscore fields or missing encapsulation
- All newtype wrappers implement `TryFrom<Primitive>` for validated construction
- Safe accessor methods (`as_str()`, `as_inner()`) are provided for read-only access

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for domain type implementations unless an explicit exception (EX-001 or EX-002) is documented and approved.
</enforcement>