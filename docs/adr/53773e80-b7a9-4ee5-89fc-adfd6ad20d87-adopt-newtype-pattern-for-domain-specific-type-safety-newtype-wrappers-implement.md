# Adopt Newtype Pattern for Domain-Specific Type Safety: Newtype Wrappers Implement

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ALWAYS ACTIVE for all data modeling decisions in the codebase. All new domain types MUST follow the newtype pattern unless explicitly exempted.

## Context

- The codebase contains numerous domain-specific concepts (extra_name, group_name, pointer_size) that require type-level guarantees beyond primitive types
- Evidence from 77 files shows consistent use of single-field wrapper structs to create distinct types from primitives (strings, integers)
- Type confusion bugs and invalid state representation are common risks when using primitive types directly for domain concepts
- Rust's zero-cost abstraction model makes the newtype pattern performant while providing compile-time safety
- Files like uv-normalize/src/extra_name.rs and uv-normalize/src/group_name.rs demonstrate the pattern applied to string-based domain identifiers

## Problem Statement

Without type-level distinctions, domain concepts represented as primitive types (strings, integers) can be accidentally mixed, passed to wrong functions, or constructed in invalid states. This leads to runtime errors that could be prevented at compile time, and makes the codebase harder to reason about as the type system doesn't encode domain semantics.

## Decision

1. SHOULD: Newtype wrappers SHOULD implement standard traits (Display, Debug, Serialize, Deserialize) to maintain ergonomics

## Policy Block

- SHOULD Newtype wrappers SHOULD implement standard traits (Display, Debug, Serialize, Deserialize) to maintain ergonomics

In scope:
- All domain-specific identifiers (names, IDs, keys)
- Validated string types with format requirements
- Numeric types with semantic meaning or constraints (sizes, counts, indices)
- Types that should not be interchangeable despite having the same underlying representation
- Configuration values with validation requirements

Out of scope:
- Pure data transfer objects with no validation logic
- Internal implementation details not exposed in public APIs
- Temporary variables in local scopes
- Performance-critical hot paths where profiling demonstrates measurable overhead

Exceptions:
- EX-001: The type is used exclusively in test code and type confusion is not a realistic risk
- EX-002: FFI boundaries require primitive types and conversion layer is clearly isolated

## Rationale

- Pattern detected across 77 files with 89.42% confidence indicates this is an established architectural standard in the codebase
- The newtype pattern leverages Rust's type system to prevent entire classes of bugs at compile time with zero runtime cost
- Evidence from files like extra_name.rs and group_name.rs shows successful application to string-based domain identifiers with validation
- Type-driven design improves code clarity by making domain concepts explicit in function signatures and data structures

## Consequences

Positive:
- Compile-time prevention of type confusion bugs where different domain concepts share the same underlying representation
- Self-documenting code where function signatures clearly express domain semantics
- Centralized validation logic in newtype constructors ensures consistency across the codebase
- Zero runtime overhead due to Rust's zero-cost abstraction guarantees

Negative:
- Increased boilerplate code for type definitions and trait implementations
- Learning curve for developers unfamiliar with the newtype pattern
- Potential ergonomic friction when interfacing with external libraries expecting primitive types
- More verbose code in some cases due to explicit type conversions

## Alternatives

- Use type aliases (type ExtraName = String) instead of newtype wrappers (rejected)
  Rejected because: Type aliases do not create distinct types in Rust - they are merely synonyms. This provides no compile-time safety and allows mixing different domain concepts freely.
  When valid: Only for internal documentation purposes where no type safety is needed
- Use enums with single variants to wrap domain types (rejected)
  Rejected because: Single-variant enums add unnecessary memory overhead (discriminant tag) and syntactic noise compared to newtype structs
  When valid: When the type may evolve to have multiple variants in the future
- Use phantom types with generic parameters for type-level distinctions (rejected)
  Rejected because: Phantom types add complexity and are harder to understand than simple newtype wrappers for most use cases
  When valid: When modeling complex type-level state machines or when the same underlying type needs multiple orthogonal type-level tags

## Risks

- Developers may bypass newtype constructors by directly accessing inner fields or using unsafe code
  Mitigation: Enforce private fields in newtype structs and use linting rules to detect unsafe patterns. Code review should verify proper constructor usage.
  Owner: Engineering team
- Excessive use of newtypes for trivial cases may lead to boilerplate fatigue and reduced code clarity
  Mitigation: Establish clear guidelines for when newtype pattern is required vs. optional. Focus on domain boundaries and validation requirements.
  Owner: Architecture team
- Serialization/deserialization may require additional configuration to handle newtype wrappers correctly
  Mitigation: Use serde's transparent attribute for simple wrappers. Document serialization patterns in implementation guide.
  Owner: Engineering team

## Implementation Notes

- Use #[derive(Debug, Clone, PartialEq, Eq)] as baseline traits for most newtype wrappers
- For serialization, apply #[serde(transparent)] to maintain JSON compatibility with the inner type
- Implement TryFrom<Primitive> for validated construction and provide an as_str()/as_inner() method for safe access
- Consider implementing Display trait for user-facing types and keep Debug for development diagnostics
- Group related newtype definitions in dedicated modules (e.g., uv-normalize for name types) to maintain organization

## Continuation Context


Verify commands:
- grep -r 'pub struct.*Name.*String' crates/ | grep -v 'pub struct.*Name(String)' | wc -l
- rg 'type\s+\w+Name\s*=\s*String' crates/ --count
- cargo clippy -- -W clippy::pub_underscore_fields

Accept when:
- All domain-specific name types are implemented as newtype wrappers, not type aliases
- Newtype struct fields are private and only accessible through validated constructors
- Clippy passes with no warnings about public underscore fields or missing encapsulation

## Enforcement

- Verified by: Automated CI checks using cargo clippy with custom lint rules
- Verified by: Code review checklist requiring newtype pattern for new domain types
- Verified by: Static analysis scanning for type alias usage in domain model modules
- Violation handling: CI build fails if clippy detects public fields in newtype structs
- Violation handling: Code review blocks merge if domain types use primitives or type aliases without justification
- Violation handling: Architecture review required for any exceptions to the pattern
- Exception process: Developer documents exception rationale in code comments and PR description
- Exception process: Team lead reviews and approves exception based on policy scope and valid use cases
- Exception process: Exception is logged in architecture decision log with justification and time-bound review date