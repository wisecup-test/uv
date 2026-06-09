# Adopt Newtype Pattern for Domain-Specific Type Safety: Newtype Wrappers Provide

These rules are ALWAYS ACTIVE for all data modeling decisions in the codebase. All new domain types MUST follow the newtype pattern unless explicitly exempted.

### Rules

- **R-NEWTYPE-001** SHOULD: Newtype wrappers SHOULD provide accessor methods rather than exposing inner values directly to maintain encapsulation.

### Verify

```bash
# Check for non-newtype name types (should return 0)
grep -r 'pub struct.*Name.*String' crates/ | grep -v 'pub struct.*Name(String)' | wc -l

# Check for type aliases used as domain types (should return 0)
rg 'type\s+\w+Name\s*=\s*String' crates/ --count

# Check for public underscore fields in structs (should pass with no warnings)
cargo clippy -- -W clippy::pub_underscore_fields
```

**Accept when:**
- All domain-specific name types are implemented as newtype wrappers, not type aliases
- Newtype struct fields are private and only accessible through validated constructors
- Clippy passes with no warnings about public underscore fields or missing encapsulation
- Newtype wrappers derive at minimum: Debug, Clone, PartialEq, Eq
- Validated construction uses TryFrom<Primitive> pattern
- Safe access provided via as_str()/as_inner() methods

<enforcement>
Claude Code MUST NOT skip or defer verification. All domain-specific identifiers, validated string types, and numeric types with semantic meaning MUST be implemented as newtype wrappers with private fields and validated constructors.
</enforcement>