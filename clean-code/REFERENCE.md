# Clean-code field guide

Read only sections relevant to the current task. These are decision aids, not a requirement to audit or rewrite every layer.

## Names and readable operations

- Use the vocabulary of the problem and the surrounding repository. Make important units and distinctions visible: duration versus timestamp, identifier versus display name, count versus capacity.
- Choose names that remain accurate at their abstraction level. An interface should describe its contract, not accidentally promise one implementation's storage mechanism.
- Distinguish operations that inspect from operations that mutate or perform I/O. If an effect is essential, make it discoverable through the name, type, or documented contract.
- Prefer a descriptive local variable over a dense expression when it exposes an important intermediate concept. Do not introduce temporary variables that merely restate trivial syntax.
- Inspect vague suffixes such as `Manager`, `Data`, and `Helper`, but do not flag them without identifying the obscured responsibility.
- Replace unexplained domain constants with names where useful. Do not turn every zero, one, loop index, or familiar mathematical constant into global configuration.
- When renaming, check public exports, wire formats, reflection, configuration, framework hooks, keyword arguments, and documentation. A text search alone does not prove that a symbol is private.

## Function boundaries and control flow

Before extracting a helper, answer these questions:

1. What coherent operation would its name communicate to a caller?
2. Are its inputs, result, effects, and failure cases understandable without reading its implementation?
3. Does extraction preserve evaluation order, short-circuiting, mutation, exceptions, and the number of I/O operations?
4. Does the caller become easier to follow, or must the reader now jump through several trivial wrappers?

Keep related steps at a comparable level of detail. A top-level operation can explain the sequence while a meaningful helper owns one technical detail; a cohesive algorithm need not be fragmented.

Investigate boolean selectors and output parameters when call sites are unclear. A better verb, enum, return value, or domain object may help, but changing an external signature is a migration rather than a local cleanup.

Guard clauses can flatten exceptional paths, provided they preserve cleanup, transaction boundaries, and effect ordering. Avoid mechanical early-return conversions in code where a common exit performs required work.

Repeated dispatch on the same variants may justify moving behavior behind a type boundary. A simple exhaustive switch or pattern match may be clearer than polymorphism, especially for closed data variants. Do not replace branching merely to satisfy a design slogan.

## Data, objects, and invariants

- Choose representations for their real change pressures. Plain records work well for transfer data and transformations; encapsulated operations help when state has invariants that callers must not violate.
- Getters and setters do not automatically provide useful encapsulation. Ask whether callers still need to understand or coordinate the object's internal representation.
- Investigate navigation through several collaborators when it couples the caller to hidden structure. Prefer asking the appropriate owner for a meaningful operation instead of exposing more internals.
- Do not count dots as a rule: record access, fluent APIs, query builders, and immutable pipelines can legitimately chain operations.
- Keep invariants near the state they govern. Prefer language-supported types and established validation mechanisms over scattered comments asserting that callers must remember a special order.
- Avoid hybrid designs that expose mutable internals while pretending that methods enforce all invariants. Correcting such a contract may require a separately agreed behavioral change.
- Treat absence explicitly using the language's optional/nullability conventions. Do not manufacture an empty object or zero merely to avoid representing an absent value.

## Modules, composition, and external boundaries

- Group code that shares a reason to change. If a module mixes unrelated policy, rendering, persistence, and environment setup, locate an actual boundary before proposing a split.
- Keep construction and configuration separate from ordinary domain work where it improves clarity or testability. Supplying an existing collaborator is often enough; a dependency injection framework is not required.
- Make dependencies explicit. Avoid duplicated assumptions about another module's limits, formats, units, initialization order, or retry policy.
- Localize third-party conventions when they would otherwise spread through domain code. A narrow adapter is useful when it translates a real contract, not when it forwards every call without adding meaning.
- Use small learning or contract tests to establish unfamiliar dependency behavior. Tests against only a hand-written mock cannot prove the external API has the assumed semantics.
- Keep infrastructure concerns such as transactions, authorization, retries, and telemetry observable and testable. Moving them behind a decorator or middleware does not make their ordering irrelevant.
- Preserve lazy initialization, resource lifetime, disposal, and startup behavior when reorganizing construction.
- Introduce variation points for demonstrated needs. Defer speculative plugin systems, generic frameworks, and extra extension interfaces.

## Errors and resource ownership

- Identify where a failure is detected, where it gains domain context, and which layer can decide recovery.
- Use the repository's error idiom: exceptions, tagged results, explicit error values, or another established mechanism. Do not impose a Java-specific exception policy on other languages.
- Keep expected absence, invalid input, cancellation, and operational failure distinguishable when the contract requires it.
- Catch only what the current layer can handle, translate, or report meaningfully. Preserve causes and actionable context without exposing credentials or sensitive payloads.
- A default value is appropriate only when the contract defines that fallback. Returning an empty collection after a failed read is not automatically equivalent to a successful read of no records.
- Keep cleanup correct on success, failure, cancellation, and partial initialization. Prefer established ownership constructs such as context managers, deferred cleanup, or scoped disposal.
- Preserve externally observable error types and timing during refactoring. Changing which input is validated first can change the error a caller sees.
- Do not add retries without checking idempotency, cancellation, attempt limits, and ownership of the retry policy.

## Tests as change protection

- For new behavior or a defect fix, prefer an executable example that fails for the relevant reason before implementation when practical. Follow the repository's testing approach rather than introducing a new framework.
- For legacy cleanup, establish what the system currently does before deciding what it ought to do. Characterization tests preserve the observed contract; desired behavioral changes need separate agreement.
- Choose tests around one coherent scenario, with readable setup and a clear outcome. Several related assertions can belong in one test.
- Keep tests fast enough for frequent use, independent of order, repeatable in their intended environment, and self-checking. Control clocks, randomness, and external state when those are not the subject of the test.
- Keep fixtures and test helpers expressive without hiding the inputs or outcomes that explain the scenario. Do not create a miniature framework for a handful of tests.
- Cover relevant boundaries: empty and single-item inputs, missing versus invalid values, inclusive and exclusive endpoints, dates and time zones, precision, and error paths.
- When a defect reveals a faulty assumption, test adjacent cases rather than only the one reproducer.
- Use coverage to find unexamined behavior, not to assert correctness or meet an invented percentage. A passing test that executes a branch without checking its result may provide little protection.
- Keep integration tests for serialization, transactions, protocol semantics, and other boundaries that unit doubles cannot establish.
- Investigate skipped or flaky tests. Do not normalize unexplained failures, add sleeps to conceal races, or weaken assertions to make a cleanup pass.
- Use existing build and test entry points. Improve a directly related reproducibility problem only within the authorized scope; do not redesign the build system during a readability change.

## Concurrency and asynchronous work

Apply this section when the changed code shares mutable state, schedules parallel work, owns background workers, or controls cancellation and shutdown.

1. Identify the shared state and the invariant that must remain true across a complete operation.
2. Identify the actual coordination mechanism: lock, transaction, atomic API, ownership transfer, message queue, or another established primitive.
3. Check compound operations. Individually thread-safe calls do not make a check-then-act sequence or a multi-step update atomic.
4. Separate scheduling and coordination from ordinary domain logic where useful, while keeping their ordering and failure contracts explicit.
5. Verify cleanup and shutdown as deliberately as startup and normal operation.

Prefer avoiding unnecessary shared state to adding more locks. Copies or immutable values can help, but account for ownership, size, consistency, and memory costs.

Keep a critical section no broader than necessary, but broad enough to protect the entire invariant. Moving a read or a write across the lock boundary is a semantic change even if the new function looks cleaner.

Check lock ordering and waits involving several resources. A consistent acquisition order can prevent some deadlocks; it is not proof that all waiting, callbacks, external I/O, or asynchronous re-entry are safe.

For worker or queue changes, inspect capacity, backpressure, task ownership, exception propagation, cancellation, producer/consumer shutdown, and whether waiting work can be released. Do not introduce an unbounded queue as a convenience.

Use the platform's established concurrent collections and scheduling primitives rather than hand-rolled synchronization unless there is a concrete requirement.

Test relevant interleavings with controlled coordination when possible. Use stress or platform-specific checks where warranted, but do not claim that repeated successful runs prove the absence of races.

Treat rare failures as evidence to investigate. Do not dismiss a failure merely because the same test passed on a retry.

## Incremental refactoring procedure

Use the worked-refactoring chapters as a process lesson, not a requirement to recreate their designs.

1. Understand the algorithm and establish a focused baseline. Record the observable behavior that the requested cleanup must preserve.
2. Start with a low-risk clarity improvement that makes the next change easier to reason about.
3. Perform one coherent transformation. Do not combine a broad structural rewrite with dependency upgrades, formatting churn, and behavioral fixes.
4. Inspect the result and run the relevant checks. If a regression appears, isolate the latest change rather than rationalizing a new expected output.
5. Reassess the design. An extraction may turn out to be less readable; inline or revise your own change when that produces a better result.
6. Stop when the scoped improvement is achieved. A module does not have to reach an imagined perfect architecture.

Before deleting apparently dead code, check exports, reflective calls, plugin discovery, configuration references, and index coverage. No known callers is not always proof of no callers.

When tests expose a pre-existing defect, distinguish the reproducible defect from a subjective expectation. Do not silently redefine case sensitivity, rounding, date arithmetic, or error behavior under the name of refactoring.

## Smell triage

A smell is a reason to inspect, not a verdict. Tie every proposed change to a real maintenance cost or correctness risk.

| Signal | Evidence to gather | Smallest useful response |
| --- | --- | --- |
| Unclear names | What the value means, its units, and who consumes it. | Rename within the authorized contract and update actual consumers. |
| Long operation | Its semantic phases, shared state, and levels of detail. | Extract a cohesive phase only when the resulting calls explain the work better. |
| Many parameters | Which arguments represent one stable domain concept. | Improve the contract; do not hide unrelated values in a generic context bag. |
| Repeated fragments | Whether they encode the same policy and change together. | Share stable knowledge; leave independent policies separate. |
| Large module | Which responsibilities have genuinely independent consumers or change pressures. | Split at an evidenced boundary, not a line-count threshold. |
| Mutable globals | Who owns them and whether order or concurrent access matters. | Make the dependency or ownership explicit without changing lifetime accidentally. |
| Dense conditionals | The domain rule, precedence, side effects, and edge cases. | Name the rule or intermediate value; preserve evaluation order. |
| Unexpected I/O | Call counts, laziness, transactions, and performance expectations. | Expose the effect and preserve its contract before reorganizing it. |
| Stale comments | Whether the comment or the implementation reflects the intended contract. | Update useful rationale; do not assume deleting the comment fixes the discrepancy. |
| Weak tests | Which outcome can change without failing the test. | Add a focused assertion or case that detects that change. |
| Missing boundary cases | The actual accepted domain and endpoint semantics. | Test the relevant edge and decide any behavior correction explicitly. |
| Cross-module assumptions | Duplicated constants, hidden initialization order, or knowledge of internals. | Move the decision to its owner or expose the required dependency directly. |

## Adaptation notes

The source book's discussions of names, functions, comments, formatting, objects, errors, boundaries, tests, classes, systems, emergent design, concurrency, and incremental cleanup inform this guide. Its worked examples and heuristic catalog supply additional questions to ask, not code to copy.

Particularly relevant source locations are the boundary discussion in chapter 8; test design in chapter 9; construction and dependencies in chapter 11; avoiding excessive classes in chapter 12; synchronization and shutdown in chapter 13; incrementalism in chapter 14; revising earlier refactoring decisions in chapter 15; establishing behavior before cleanup in chapter 16; the heuristic catalog in chapter 17; and the deadlock and concurrency-testing discussion in appendix A.

Modern adaptations are deliberate: typed optional/result values, exhaustive pattern matching, immutable transformations, and structured asynchronous lifetimes can serve the same goals. Preserve language and project conventions instead of translating older Java-specific recommendations mechanically.
