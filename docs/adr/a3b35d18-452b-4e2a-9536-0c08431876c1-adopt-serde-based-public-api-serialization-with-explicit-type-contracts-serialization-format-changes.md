# Adopt Serde-Based Public API Serialization with Explicit Type Contracts: Serialization Format Changes

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase contains 46 files implementing public API types with consistent serialization patterns, indicating a systematic approach to external API contracts
- Files like package_name.rs, extra_name.rs, group_name.rs, and credential.rs suggest domain-specific types that require stable serialization for external consumption
- The pattern appears in normalization modules (uv-normalize), CLI options (uv-cli), and integration points (uv-keyring, uv-auth), indicating cross-cutting API concerns
- High confidence (89.66%) and significance suggest this is a deliberate architectural choice rather than incidental code organization
- The presence of macro support (uv-macros) indicates infrastructure investment in maintaining consistent API serialization patterns

## Problem Statement

Public APIs require stable, versioned serialization contracts that can evolve without breaking external consumers. Without explicit type definitions and serialization rules, API changes risk breaking integrations, causing data corruption, or creating maintenance burden through implicit contracts that are difficult to track and validate.

## Decision

1. MUST: Serialization format changes MUST maintain backward compatibility or increment API version

## Policy Block

- MUST Serialization format changes MUST maintain backward compatibility or increment API version

In scope:
- All types exposed through CLI interfaces (uv-cli)
- All types used in configuration files or persistent storage
- All types transmitted over network boundaries (HTTP APIs, RPC)
- All types in normalization layers that define canonical representations (uv-normalize)
- Credential and authentication types (uv-keyring, uv-auth)

Out of scope:
- Internal implementation types not exposed beyond module boundaries
- Temporary data structures used only within function scope
- Types used exclusively for in-memory computation without serialization
- Test fixtures and mock objects (unless testing serialization behavior)

Exceptions:
- EX-001: Performance-critical internal types where serialization overhead is prohibitive and external exposure is guaranteed impossible

## Rationale

- Pattern detected across 46 files with 89.66% confidence indicates this is a proven, stable approach in the codebase
- Serde provides industry-standard serialization with strong ecosystem support, reducing maintenance burden and enabling multiple format targets (JSON, TOML, etc.)
- Explicit type contracts enable compile-time verification of API compatibility and prevent accidental breaking changes
- Separation of API types into dedicated modules (uv-normalize, uv-cli) creates clear boundaries and enables independent evolution of internal vs. external contracts

## Consequences

Positive:
- Strong type safety prevents entire classes of serialization bugs at compile time
- Clear API boundaries make it easier to reason about backward compatibility and versioning
- Serde ecosystem enables automatic support for multiple formats (JSON, YAML, TOML, MessagePack) without code duplication
- Newtype patterns allow adding validation, normalization, and business logic without exposing implementation details

Negative:
- Additional boilerplate required for each public API type (derive macros, validation logic)
- Serde dependency becomes a hard requirement across the codebase, increasing compile times
- Breaking changes to serialization format require careful migration planning and versioning strategy
- Learning curve for developers unfamiliar with Serde's attribute system and customization options

## Alternatives

- Use raw primitives (String, i32, etc.) for API types without newtype wrappers (rejected)
  Rejected because: Exposes implementation details, prevents adding validation logic, and makes future evolution difficult without breaking changes
  When valid: Only for internal types that will never be exposed externally
- Implement custom serialization logic without Serde using manual trait implementations (rejected)
  Rejected because: Significantly increases maintenance burden, loses ecosystem benefits, and is error-prone compared to derive macros
  When valid: Only when Serde cannot support required format (e.g., binary protocols with complex state machines)
- Use protocol buffers or similar schema-first approach for all API types (rejected)
  Rejected because: Adds complexity of schema compilation step, less idiomatic for Rust, and overkill for configuration/CLI use cases
  When valid: For high-performance RPC systems or when cross-language compatibility is primary requirement

## Risks

- Serde version upgrades or breaking changes could require widespread codebase updates
  Mitigation: Pin Serde version in workspace Cargo.toml, test serialization compatibility in CI, maintain comprehensive serialization test suite
  Owner: Platform team
- Developers may accidentally break API compatibility by modifying Serde attributes or type structure
  Mitigation: Implement serialization snapshot tests, add API compatibility checks to CI, require architecture review for public API changes
  Owner: Engineering team
- Performance overhead of serialization/deserialization in hot paths
  Mitigation: Profile serialization performance, use zero-copy deserialization where possible, consider binary formats for performance-critical paths
  Owner: Performance team

## Implementation Notes

- Use #[derive(Serialize, Deserialize)] on all public API types; add #[serde(rename_all = "snake_case")] for consistent JSON field naming
- Create dedicated modules for API types (e.g., api/, types/, contracts/) to clearly separate public contracts from internal implementation
- Implement comprehensive serialization round-trip tests: serialize to format, deserialize back, assert equality
- Consider using #[serde(deny_unknown_fields)] during development to catch API mismatches early, but allow unknown fields in production for forward compatibility
- Document serialization format in type-level doc comments with examples showing JSON/TOML representation

## Continuation Context


Verify commands:
- grep -r "#\[derive.*Serialize.*Deserialize" crates/uv-normalize/ crates/uv-cli/ | wc -l
- cargo test --package uv-normalize --package uv-cli -- serde
- rg "pub (struct|enum)" crates/uv-normalize/ | grep -v "#\[derive.*Serialize" || echo "All public types have Serde derives"

Accept when:
- All public API types in uv-normalize, uv-cli, uv-keyring, and uv-auth modules have Serde Serialize and Deserialize derives
- Serialization round-trip tests pass for all public API types with at least one format (JSON or TOML)
- No public API types expose raw primitives without newtype wrappers in modules designated for external contracts

## Enforcement

- Verified by: CI pipeline runs cargo test with serialization test suite
- Verified by: Code review checklist includes verification of Serde derives on new public types
- Verified by: Automated linting via custom clippy lint or cargo-semver-checks for API compatibility
- Violation handling: CI fails if public API types lack Serde derives (detected via custom lint)
- Violation handling: PR blocked until serialization tests are added for new public types
- Violation handling: Architecture review required for any changes to existing public API serialization format
- Exception process: Submit exception request with justification to architecture review board
- Exception process: Provide evidence that alternative approach is necessary (performance benchmarks, technical constraints)
- Exception process: Document exception in ADR amendments with expiration date for re-evaluation