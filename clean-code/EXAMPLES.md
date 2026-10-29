# Clean-code examples

These examples are original. They demonstrate judgment, not mandatory syntax or architecture.

## Writing: clarify the contract first

Request: "Add a retry-delay parser using clean-code principles."

Inspect the existing configuration format and callers before implementing it. Establish the unit, accepted syntax, valid range, and failure behavior. A name such as `parse_retry_delay_seconds` may communicate the contract; silently returning zero for malformed input would introduce a policy decision. Reuse the repository's parsing and error conventions rather than inventing a generic parsing framework.

Test supported values, the established boundaries, and malformed input. Do not invent requirements such as fractional support or automatic clamping simply to make the implementation appear friendly.

## Refactoring: clearer names without new behavior

Before:

```python
def posted_total_cents(entries):
    n = 0
    for r in entries:
        if r["status"] == "posted":
            n += r["amount_cents"]
    return n
```

After:

```python
def posted_total_cents(entries):
    total_cents = 0
    for entry in entries:
        if entry["status"] == "posted":
            total_cents += entry["amount_cents"]
    return total_cents
```

The function and parameter names remain unchanged, so existing positional and keyword callers keep their contract. Only local names change; there is no new helper, class, dependency, or validation policy.

Relevant preservation checks:

- Empty input still returns zero.
- Posted positive, zero, and negative amounts accumulate in the original order.
- Non-posted entries do not require or read `amount_cents`.
- Input is traversed once, so generators still work without eager materialization.
- Missing required keys still raise `KeyError` at the original point.
- Input records are not modified.

Do not silently replace required dictionary accesses with `.get(..., 0)`: that would convert invalid data into an apparently valid total. Do not add logging or input validation as part of this naming-only change.

## Reviewing: distinguish a suggestion from a defect

For the before example, the supported observation is that `n` and `r` obscure the accumulated unit and the record's role. The minimal improvement is to rename those locals. Nothing in that observation proves the calculation is incorrect.

In a real review, cite the actual file and lines; explain the maintenance cost and suggest the local rename. Do not manufacture a high-severity bug, request a new service class, or edit the file when the user requested review only.

## Boundary design: test an assumption before wrapping it

Request: "Clean up a module that calls a storage client."

Inspect the actual client contract: an empty listing, a missing container, a permission failure, and a timed-out request may represent different outcomes. A small contract test can establish the distinction. An adapter is justified if it prevents external response shapes and error conventions from spreading through domain code.

A helper that only renames the SDK's `list` method is not necessarily a useful boundary. Do not make the adapter return an empty list for every failure or retry writes automatically; either choice changes behavior.

## Concurrent cleanup: preserve the operation, not just individual calls

Request: "Make the queue worker easier to read without changing behavior."

Suppose the worker checks capacity and reserves a slot under the same lock. Extracting `has_capacity()` and `reserve_slot()` into separately locked functions can introduce a race between those calls, even if each function is thread-safe by itself.

Keep the complete check-and-reserve operation atomic. A helper can own that operation while ordinary formatting or parsing stays separate. Check the existing cancellation, worker shutdown, error propagation, and resource-release paths as well as successful work.

The improvement is clearer responsibility with the original coordination intact, not merely shorter functions. Do not claim that a single successful stress run proves the change race-free.

## Legacy cleanup: distinguish observed and desired behavior

Request: "Refactor the date parser without changing accepted inputs."

Characterize the currently accepted formats and boundary dates first. If the parser currently rejects lowercase month names, accepting them is a behavioral change even if it seems convenient. Preserve that behavior during structural cleanup and raise any desired expansion separately.

Use a failing test to capture a confirmed defect when a fix is in scope. Do not rewrite existing expected outputs to match a new implementation and call that preservation.

## Judgment calls

| Situation | Appropriate response |
| --- | --- |
| A long function implements one well-documented parsing algorithm. | Inspect control flow and tests; extract meaningful concepts only if doing so improves understanding. Length alone is not a finding. |
| Two similar transformations enforce different business policies. | Keep them separate when they evolve independently. Do not couple them through an increasingly configurable helper. |
| A public field has an awkward name. | Preserve the wire format; consider an internal adapter or a separately approved migration rather than a breaking rename. |
| A failure path returns an empty collection. | Establish whether empty means success, absence, or failure. Report ambiguity with evidence; do not change established recovery behavior under the label of cleanup. |
| An extraction moves a database read outside a lock. | Reject or revise it unless the original consistency guarantee is preserved and supported by appropriate evidence. Shorter code does not justify a race. |
| Several assertions describe one coherent result. | Keep the test readable; do not split it mechanically into one-assertion tests. |
| A comment explains a protocol constraint or subtle algorithm. | Retain and update the explanation. Clear identifiers cannot replace all rationale. |
