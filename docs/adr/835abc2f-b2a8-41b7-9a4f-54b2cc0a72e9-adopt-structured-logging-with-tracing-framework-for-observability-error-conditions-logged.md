# Adopt Structured Logging with Tracing Framework for Observability: Error Conditions Logged

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all Rust crates and modules within the uv project that perform operations requiring observability, debugging, or operational monitoring.

## Context

- The uv project is a complex Python package manager written in Rust with multiple crates handling installation, caching, client operations, authentication, and metadata management
- Evidence shows consistent logging patterns across 21 files spanning critical subsystems including uv-installer, uv-client, uv-cache-info, uv-auth, and uv-metadata with 88.76% confidence
- Operations such as package installation, cache management, HTTP client interactions, and tool management require detailed observability for debugging production issues and understanding system behavior
- The Rust ecosystem provides the tracing framework which offers structured, contextual logging with span-based hierarchical event tracking superior to traditional log statements
- Distributed across multiple crates, the system requires consistent logging practices to enable effective troubleshooting and operational monitoring across module boundaries

## Problem Statement

Without a standardized logging approach, the uv project risks inconsistent observability across its many crates, making it difficult to diagnose issues, understand system behavior in production, and maintain operational excellence as the codebase scales. Traditional logging approaches lack the structured context and hierarchical relationships needed to trace complex operations spanning multiple modules and async boundaries.

## Decision

1. SHOULD: Error conditions SHOULD be logged at the error! level with sufficient context to diagnose the issue, including error messages and relevant state

## Policy Block

- SHOULD Error conditions SHOULD be logged at the error! level with sufficient context to diagnose the issue, including error messages and relevant state

In scope:
- All Rust crates under the uv project workspace
- Core operational modules: uv-installer, uv-client, uv-cache-info, uv-auth, uv-metadata
- Command implementations in uv/src/commands/*
- HTTP client operations and network interactions
- File system operations and cache management
- Package installation and dependency resolution workflows

Out of scope:
- Third-party dependencies that use their own logging frameworks
- Test utilities and mock implementations where logging is not operationally relevant
- Build scripts and code generation tools
- Documentation examples that demonstrate concepts unrelated to logging

Exceptions:
- EXC-001: A third-party library requires integration with the standard log crate and cannot use tracing directly
- EXC-002: Performance-critical hot paths where tracing overhead is measured and deemed unacceptable

## Rationale

- Pattern detected across 21 files with 88.76% confidence indicates this is an established architectural practice within the uv codebase, not an isolated occurrence
- The tracing framework provides superior structured logging with hierarchical spans, enabling better debugging of complex async operations and cross-module workflows common in package management
- Consistent logging practices across all crates enable effective operational monitoring, troubleshooting, and performance analysis as the system scales
- Evidence spans critical subsystems (installer, client, cache, auth, metadata) demonstrating this is a cross-cutting concern requiring standardization

## Consequences

Positive:
- Consistent observability across all uv crates enables efficient debugging and troubleshooting of production issues
- Structured logging with fields enables powerful filtering, aggregation, and analysis capabilities for operational monitoring
- Hierarchical spans provide clear context for operations spanning multiple modules and async boundaries
- Standardized approach reduces cognitive load for developers working across different crates
- Integration with tracing ecosystem enables advanced features like distributed tracing, metrics collection, and performance profiling

Negative:
- Requires learning curve for developers unfamiliar with the tracing framework and its span-based model
- Adds dependency on the tracing crate and its ecosystem to all modules
- Potential performance overhead from tracing instrumentation in hot paths, though typically negligible
- Requires discipline to maintain consistent logging practices across a large codebase with multiple contributors

## Alternatives

- Use the standard log crate with traditional logging macros (log::info!, log::error!, etc.) (rejected)
  Rejected because: The log crate lacks structured logging capabilities, hierarchical context through spans, and the rich ecosystem needed for complex async operations. It provides insufficient observability for a multi-crate async system like uv.
  When valid: Only appropriate for simple, single-threaded applications without complex operational requirements
- Use println!/eprintln! for direct console output without a logging framework (rejected)
  Rejected because: Direct console output lacks log levels, filtering, structured fields, and cannot be redirected or integrated with observability tools. Unsuitable for production systems requiring operational monitoring.
  When valid: Only acceptable in simple CLI examples or throwaway scripts
- Allow each crate to choose its own logging approach independently (rejected)
  Rejected because: Inconsistent logging across crates creates fragmented observability, making it difficult to trace operations spanning multiple modules and increasing maintenance burden.
  When valid: Never appropriate for a cohesive multi-crate project requiring unified observability

## Risks

- Performance degradation in hot paths due to tracing instrumentation overhead
  Mitigation: Use conditional compilation with tracing levels, benchmark critical paths, and apply EXC-002 exception process for proven performance issues
  Owner: Engineering team with performance monitoring
- Inconsistent adoption across crates leading to fragmented observability
  Mitigation: Enforce through code review, provide clear documentation and examples, implement automated verification in CI pipeline
  Owner: Architecture team and code reviewers
- Accidental logging of sensitive information (tokens, credentials, PII)
  Mitigation: Implement code review checklist for sensitive data, provide sanitization utilities, add automated scanning for common patterns of sensitive data in logs
  Owner: Security team and engineering team

## Implementation Notes

- Add 'tracing' dependency to Cargo.toml for all crates requiring logging capabilities
- Use #[instrument] attribute on functions to automatically create spans with function arguments as fields
- For manual span creation, use tracing::info_span! or similar macros with descriptive names and relevant fields
- Configure tracing subscriber in main application entry points (e.g., uv/src/main.rs) with appropriate filtering and formatting
- Use tracing::field::Empty for fields that will be recorded later within a span using span.record()
- Consider using tracing-subscriber's EnvFilter for runtime log level configuration via environment variables

## Continuation Context


Verify commands:
- grep -r 'use tracing::' crates/ --include='*.rs' | wc -l
- grep -r 'println!\|eprintln!' crates/ --include='*.rs' | grep -v 'test\|example' | wc -l
- grep -r '#\[instrument\]' crates/ --include='*.rs' | wc -l
- cargo tree -p uv --depth 1 | grep tracing

Accept when:
- All in-scope crates have tracing as a dependency and use tracing macros for logging
- Critical operations (HTTP requests, cache operations, installations) are instrumented with spans
- No use of println!/eprintln! for operational logging in production code paths
- Code review checklist includes verification of appropriate tracing usage and sensitive data handling

## Enforcement

- Verified by: Automated CI checks using grep patterns to detect non-compliant logging approaches
- Verified by: Code review checklist requiring verification of tracing usage in new code
- Verified by: Periodic architecture audits scanning for println!/eprintln! usage outside tests
- Verified by: Dependency analysis ensuring tracing crate is present in all operational modules
- Violation handling: CI pipeline fails if println!/eprintln! detected in production code paths
- Violation handling: Code review blocks merge if logging does not follow tracing patterns
- Violation handling: Architecture team notified of violations for pattern analysis and guidance
- Violation handling: Violations documented in technical debt backlog for remediation
- Exception process: Submit exception request to architecture team with detailed justification
- Exception process: For performance exceptions (EXC-002): provide benchmark data showing unacceptable overhead
- Exception process: For third-party integration exceptions (EXC-001): document bridge configuration and compatibility approach
- Exception process: All approved exceptions must be documented in code comments with exception ID reference
- Exception process: Exceptions reviewed quarterly to assess if conditions have changed