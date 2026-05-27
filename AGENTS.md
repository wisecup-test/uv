<!-- actual-ai:adr-governance:start -->
# Project ADRs

This project's conventions are encoded as ADRs under `.actual/rules/`. **The ADRs ARE the pattern.** Follow them verbatim instead of reading existing implementations to figure out how to do something.

> **Note:** this directive is calibrated for one-shot tasks (a single discrete feature). For multi-task interactive sessions, consult the ADRs for each task transition rather than holding to the per-session caps below.

## Workflow (follow in order)

1. **Identify topic.** Match the files you'll edit against the path-glob table below. Pick the 1-3 topics that match. Do not pre-emptively pick "related" topics; pick only what the file paths actually match.

2. **Read ADRs you'll actually apply.** For each chosen topic, run `ls .actual/rules/<topic>-*.md`. Each filename has three parts: `<topic>-<aspect-slug>-<hash>.md`. The `<aspect-slug>` (middle segment) is the ADR's specific concern — e.g., `database-schema-defined`, `zod-input-validation`, `cache-key-format`.

   **Scan all filenames first**, then identify which aspect-slugs match what you're about to write. Read at most **3 ADR files per topic** whose aspect-slugs directly apply. **Do not read more than 5 ADR files total** across all topics. Broadly-scoped topics like `cross-cutting-` may contain 50+ rules; most won't apply to your task. Filter by aspect-slug *before* reading content.

3. **Locate insertion points (one read per file, max 3 files).** You may read source files ONLY to (a) find where to add code (which directory, which barrel export to update) or (b) look up an exact identifier you must import. **Do not read source files as pattern examples — the ADRs already encode the pattern.** If you find yourself reading a file because "I want to see how X is done elsewhere," stop. The ADR you already read tells you how.

4. **Implement.** Write the code following the rule statements verbatim. If two ADRs seem to conflict, follow the more specific one (longer topic prefix wins).

5. **Verify after implementing.** Only after the code is written, re-read the `verify_commands` or `accept_criteria` sections of the ADRs you applied and check your work against them. Run the verify commands if any.

## Anti-patterns to avoid

- Reading the first N rules alphabetically because they're cheap. Filter by aspect-slug first, then read only the relevant ones.
- Reading >5 ADR files for a single feature. If you're tempted, you're over-scoping the topic match.
- Reading existing similar features to "see the pattern" — the ADRs encode the pattern. Trust them.
- Re-reading the same ADR multiple times. Cache it mentally.
- Continuing to browse the codebase after step 3. By step 4 you should be writing, not reading.

Each rule file at `.actual/rules/<topic>-<aspect>-<hash>.md` contains the full ADR with rule statements, verify commands, and accept criteria.

## Verification Protocol

These rules are ALWAYS ACTIVE. Apply every rule that governs the files you touch to all code generation, modification, and review — no exceptions unless a rule says otherwise.

Every rule follows a **Verify → Fix → Repeat** loop. After generating or modifying code for any rule you MUST:

1. **RUN** the rule's `### Verify` command(s).
2. **CAPTURE** the full output (stdout + stderr).
3. **EVALUATE** the output against the rule's **Accept when** criteria.
4. **IF FAILING:** diagnose the root cause, apply a fix, and re-run from step 1.
5. **IF PASSING:** keep the passing output as evidence before moving on.
6. **MAX ITERATIONS:** 5 attempts per rule. If still failing after 5 attempts, STOP and report the failure with all captured output.

Compliance is not optional. Do not skip verification, assume correctness, or defer it to a later task. Every change to a governed area must be accompanied by a passing verification run.

## Path glob → topic

| You're editing | Topic prefix |
|---|---|
| `**/*` | `cross-cutting-` _(84 ADRs)_ |
<!-- actual-ai:adr-governance:end -->

- Read CONTRIBUTING.md for guidelines on how to run tools
- ALWAYS attempt to add a test case for changed behavior
- PREFER integration tests, e.g., at `it/...` over unit tests
- PREFER `insta` snapshots following patterns in nearby tests over substring assertions
- When making changes for Windows from Unix, use `cargo xwin clippy` to check compilation
- NEVER perform builds with the release profile, unless asked or reproducing performance issues
- PREFER running specific tests over running the entire test suite
- AVOID using `panic!`, `unreachable!`, `.unwrap()`, unsafe code, and clippy rule ignores
- PREFER patterns like `if let` to handle fallibility
- ALWAYS write `SAFETY` comments following our usual style when writing `unsafe` code
- PREFER `#[expect()]` over `[allow()]` if clippy must be disabled
- PREFER let chains (`if let` combined with `&&`) over nested `if let` statements
- NEVER update all dependencies in the lockfile and ALWAYS use `cargo update --precise` to make
  lockfile changes
- NEVER assume clippy warnings are pre-existing, it is very rare that `main` has warnings
- ALWAYS read and copy the style of similar tests when adding new cases
- PREFER top-level imports over local imports or fully qualified names
- AVOID shortening variable names, e.g., use `version` instead of `ver`, and `requires_python`
  instead of `rp`
- PREFER [`TypeName`] references when writing Rust doc comments
