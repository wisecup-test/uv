# Enforce Strict Input Validation with Type-Safe Parsing: Common Validation Patterns

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all code that processes external input, including CLI arguments, HTTP responses, configuration files, and user-provided data.

## Context

- The codebase processes diverse external inputs including CLI arguments, HTTP responses, netrc files, HTML content, and configuration data that require validation before use
- Pattern detected across 77 files with 89.42% confidence, indicating a systematic approach to input validation throughout the security-critical components
- Components like uv-normalize, uv-cli, uv-client, and uv-netrc demonstrate consistent validation patterns for package names, extra names, group names, and network credentials
- Type-safe parsing prevents injection attacks, malformed data propagation, and runtime errors by catching invalid input at system boundaries
- The pattern appears in test files (common/mod.rs, main.rs, ecosystem.rs) suggesting validation is a verified and tested architectural concern

## Problem Statement

Without systematic input validation, external data can introduce security vulnerabilities (injection attacks, path traversal), cause runtime failures from malformed data, or propagate invalid state through the system. The codebase needs a consistent approach to validate and sanitize all external inputs at trust boundaries.

## Decision

1. SHOULD: Common validation patterns SHOULD be extracted into reusable validation modules (e.g., uv-normalize for name validation)

## Policy Block

- SHOULD Common validation patterns SHOULD be extracted into reusable validation modules (e.g., uv-normalize for name validation)

In scope:
- CLI argument parsing and option validation
- HTTP response body parsing and header validation
- Configuration file parsing (TOML, JSON, YAML)
- Package metadata validation (names, versions, extras, groups)
- Credential and authentication data (netrc files, tokens)
- HTML content parsing from package indexes
- File path and URL validation
- User-provided strings in interactive commands

Out of scope:
- Internal data structures already validated at boundaries
- Compile-time constants and hardcoded values
- Data generated programmatically within trusted code
- Test fixtures with known-good data (though tests should verify validation logic)

Exceptions:
- EXC-001: Performance-critical hot paths where input has been pre-validated at an earlier boundary
- EXC-002: Trusted internal APIs where caller contract guarantees valid input

## Rationale

- Pattern detected across 77 files with 89.42% confidence indicates this is an established architectural principle with broad adoption
- Security-critical components (uv-client, uv-netrc, uv-keyring) demonstrate validation is essential for protecting against malicious input
- Type-safe parsing in Rust leverages the language's strong type system to make invalid states unrepresentable
- Consistent validation patterns in normalization modules (extra_name.rs, group_name.rs) show reusable validation reduces duplication and errors

## Consequences

Positive:
- Prevents injection attacks, path traversal, and other input-based security vulnerabilities
- Catches malformed data early at system boundaries, preventing error propagation deep into the application
- Provides clear error messages to users when input is invalid, improving usability
- Leverages Rust's type system to enforce validation at compile time where possible
- Reusable validation modules reduce code duplication and ensure consistent validation logic

Negative:
- Adds parsing overhead at system boundaries, though typically negligible compared to I/O costs
- Requires maintaining validation logic as input formats evolve
- May reject edge cases that could technically be handled, requiring exception processes
- Increases initial development time for new input-handling code

## Alternatives

- Lazy validation: Accept all input and validate only when used (rejected)
  Rejected because: Allows invalid data to propagate through the system, making it harder to track where validation should occur and increasing the attack surface
  When valid: Never appropriate for security-critical systems
- Runtime assertions: Use assert! or unwrap() assuming input is valid (rejected)
  Rejected because: Causes panics on invalid input, leading to denial-of-service vulnerabilities and poor user experience
  When valid: Only for internal invariants, never for external input
- Sanitization-only: Automatically fix invalid input rather than rejecting it (rejected)
  Rejected because: Can mask errors and lead to unexpected behavior; validation with clear errors is more predictable
  When valid: Limited cases like normalizing whitespace where transformation rules are well-defined and documented

## Risks

- Validation logic may have bugs that allow malicious input to bypass checks
  Mitigation: Comprehensive test coverage including fuzzing, security audits of validation code, and use of well-tested parsing libraries
  Owner: Security team and module owners
- Overly strict validation may reject legitimate edge cases, frustrating users
  Mitigation: Document validation rules clearly, provide helpful error messages, and establish exception process for legitimate use cases
  Owner: Engineering team
- Performance overhead from validation could impact high-throughput operations
  Mitigation: Profile validation code, optimize hot paths, and consider caching validation results for repeated inputs
  Owner: Performance engineering team

## Implementation Notes

- Use the uv-normalize crate for package name, extra name, and group name validation to ensure consistency
- Implement validation as TryFrom or FromStr traits to integrate with Rust's type system and enable ? operator usage
- Return descriptive error types (not just String) that can be programmatically handled and provide context for debugging
- Add validation tests including boundary cases, malformed input, and security-relevant attack patterns (injection attempts, path traversal)
- Document validation rules in module documentation so users understand what input is acceptable
- For complex parsing (HTML, netrc), use established libraries (html5ever, netrc parsers) rather than custom implementations

## Continuation Context


Verify commands:
- grep -r 'unwrap()' crates/uv-cli/src/ crates/uv-client/src/ crates/uv-normalize/src/ | grep -v test | wc -l
- rg 'impl.*FromStr' crates/uv-normalize/src/ -A 5 | grep -c 'type Err'
- cargo test --package uv-normalize --package uv-cli -- validation
- rg 'parse.*unchecked|validate.*false' crates/ --type rust

Accept when:
- All external input parsing returns Result types with explicit error handling (no unwrap() in production code paths)
- Validation tests exist for each input type covering valid, invalid, and boundary cases
- Security-sensitive modules (uv-client, uv-netrc, uv-keyring) have no unvalidated external input usage
- Common validation patterns are extracted into reusable modules with documented validation rules

## Enforcement

- Verified by: Automated code review checks for unwrap() and expect() usage on external input
- Verified by: Security-focused code review for all changes to input parsing and validation logic
- Verified by: CI pipeline runs validation test suites and fuzzing on input parsers
- Verified by: Static analysis tools (clippy) configured to warn on unsafe input handling patterns
- Violation handling: CI fails if validation tests are missing for new input-handling code
- Violation handling: Security team review required for any PR that modifies validation logic in security-critical modules
- Violation handling: Post-incident review if invalid input causes production issues, with validation improvements required
- Violation handling: Clippy warnings on unsafe patterns must be addressed or explicitly documented with allow() and justification
- Exception process: Document the exception request with justification for why validation cannot be applied
- Exception process: Security team review and approval for any exception in security-critical paths
- Exception process: Add explicit comments in code explaining why validation is skipped and what guarantees exist
- Exception process: Track exceptions in security review log and revisit during security audits