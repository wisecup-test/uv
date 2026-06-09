# Adopt Newtype Pattern for Domain-Specific Type Safety: Newtype Constructors Validate

These rules are ALWAYS ACTIVE for all data modeling decisions in the codebase. All new domain types MUST follow the newtype pattern unless explicitly exempted.

### Rules

- **R-NEWTYPE-001** MUST: Newtype constructors MUST validate input and return Result types when validation can fail, preventing invalid state construction.
- **R-NEWTYPE-002** MUST: Newtype struct fields MUST be private and only accessible through validated constructors.
- **R-NEWTYPE-003** MUST: Domain-specific identifiers (names, IDs, keys), validated string types with format requirements, and numeric types with semantic meaning or constraints MUST use the newtype pattern.
- **R-NEWTYPE-004** SHOULD: Use #[derive(Debug, Clone, PartialEq, Eq)] as baseline traits for most newtype wrappers.
- **R-NEWTYPE-005** SHOULD: For serialization, apply #[serde(transparent)] to maintain JSON compatibility with the inner type.
- **R-NEWTYPE-006** SHOULD: Implement TryFrom<Primitive> for validated construction and provide an as_str()/as_inner() method for safe access.
- **R-NEWTYPE-007** MAY: Implement Display trait for user-facing types and keep Debug for development diagnostics.

### Verify

```bash
# Check for non-newtype domain name types
grep -r 'pub struct.*Name.*String' crates/ | grep -v 'pub struct.*Name(String)' | wc -l

# Check for type aliases used as domain types
rg 'type\s+\w+Name\s*=\s*String' crates/ --count

# Check for public fields in structs (clippy lint)
cargo clippy -- -W clippy::pub_underscore_fields
```

**Accept when:**
- All domain-specific name types are implemented as newtype wrappers, not type aliases
- Newtype struct fields are private and only accessible through validated constructors
- Clippy passes with no warnings about public underscore fields or missing encapsulation
- Newtype constructors return Result types when validation can fail
- Related newtype definitions are grouped in dedicated modules to maintain organization

<enforcement>
Claude Code MUST NOT skip or defer verification. All verification commands MUST pass before accepting code that introduces or modifies domain types.
</enforcement>