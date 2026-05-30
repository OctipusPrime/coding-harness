---
name: write-tests
description: Walk the user through writing tests for code that already exists in the current conversation, one test at a time, ordered by impact. Each approved test is written and run to prove it fails (red state) — no implementation. Use when the user wants to add tests to recently written/discussed code, "write tests for what we just built", or invokes this skill mid-conversation after a feature/fix.
---

# Write Tests (interactive, conversation-aware)

This skill is **not** TDD's full red-green-refactor loop. The code under test already exists (or is being scoped out). Your job is to design tests for it together with the user, one at a time, ranked by bang-for-buck, taking feedback at every step. When the user approves a test, you write it and run it so they can see it **fail** — then move on. **Never write the implementation to make it pass.** The user will drive that separately.

## Step 1 — Pick up where the conversation left off

Before saying anything, reconstruct what was just built or discussed:

- What feature, fix, or module is the user most likely asking you to test? Use the most recent substantive code or topic in the conversation as the target. If genuinely ambiguous between two equally-recent candidates, state your best guess in one sentence and proceed — don't stall.
- Read the relevant files yourself if you don't already have them in context. Don't ask the user to paste code you can fetch.
- Note the test framework, runner command, and test file conventions already in use in the repo. Match them. Don't introduce a new framework.

## Step 2 — Design the test list (silently)

Think through the full set of tests you would write for the target. Then rank them by **impact per unit of effort**:

- Critical paths users/callers actually hit
- Behaviors where a regression would be silent or expensive
- Branches with real logic, not trivial passthroughs
- Edge cases that have historically broken or are easy to get wrong

Push down: trivial getters, framework behavior, things already enforced by the type system, exhaustive permutations.

You don't share this full list with the user. You walk through it one at a time.

## Step 3 — Present ONE test at a time

For each test, in priority order, produce a short pitch with three parts:

1. **What** — the behavior under test, phrased as a capability ("user can checkout with an empty cart"), not an implementation step.
2. **Why this one now** — why it earns its slot in the priority order. What breaks silently if this isn't covered? Why is it higher value than the others you're holding back?
3. **How** — a concrete sketch: the setup, the action through the public interface, the observable assertion. Code-shaped but tight. Name the file it would live in.

Then explicitly check in. Something like: "Does this one make sense, or do you want to adjust it before I write it?"

## Step 4 — Incorporate feedback, then write + run the test

- **Positive signal** ("yes", "looks good", "go", "next"):
  1. Write the test to the appropriate test file (Edit or Write).
  2. Run the test suite (or just this test, if the runner supports it) so the user can see it fail.
  3. Show the failing output. Confirm out loud that **failing is the expected and correct outcome here** — the implementation lives outside this skill's scope.
  4. If the test passes unexpectedly, stop and flag it: either the behavior is already implemented, the test is testing the wrong thing, or it's not actually exercising the path. Diagnose with the user before moving on.
  5. If the test errors instead of fails (import error, syntax error, missing fixture), fix the test itself until it produces a real assertion-level failure — that's the bar for "red."
  6. Then move to the **next** test in your ranked list and repeat Step 3.
- **Feedback** ("change X", "I don't care about that case", "test it through the API instead"): incorporate the change and re-present the **same** test with the adjustment. Don't write or run anything yet. Check in again. Don't advance until the user approves the current one.
- **Reprioritization** ("skip that, do the error path first"): re-rank and present the new top item.
- **Stop signal** ("that's enough", "stop", a new unrelated request, switching to another skill): exit cleanly. Do not keep proposing more tests.

One test per turn is the default rhythm. Don't dump a numbered list of all tests you're planning — the whole point is the walk-through.

## Test design rules

Apply these when designing every test. They're the bar.

### What makes a good test

- **Integration-style**: exercises real code paths through the public interface.
- **Describes WHAT, not HOW**: the test name reads like a spec line ("user can checkout with valid cart"), not like an implementation trace ("checkout calls paymentService.process").
- **Survives internal refactors**: if you rename a private function or restructure internals, the test should still pass when behavior is unchanged.
- **One logical assertion per test**.
- **Verifies through the same interface callers use**. Don't reach into the database to confirm a write — call the read API.

```typescript
// GOOD: tests observable behavior through the public interface
test("user can checkout with valid cart", async () => {
  const cart = createCart();
  cart.add(product);
  const result = await checkout(cart, paymentMethod);
  expect(result.status).toBe("confirmed");
});

// GOOD: verifies through the same interface
test("createUser makes user retrievable", async () => {
  const user = await createUser({ name: "Alice" });
  const retrieved = await getUser(user.id);
  expect(retrieved.name).toBe("Alice");
});
```

### Red flags — rewrite or drop these

- Mocking internal collaborators (mock only true system boundaries: network, clock, filesystem when relevant, third-party SDKs).
- Testing private methods directly.
- Asserting on call counts, call order, or which internal method was invoked.
- Verifying through external means (querying the DB, reading a file) instead of the public interface.
- Tests that would break under an internal rename even though behavior is unchanged.
- Tests that only restate type-system or framework guarantees.

```typescript
// BAD: tests implementation, breaks on refactor
test("checkout calls paymentService.process", async () => {
  const mockPayment = jest.mock(paymentService);
  await checkout(cart, payment);
  expect(mockPayment.process).toHaveBeenCalledWith(cart.total);
});

// BAD: bypasses the public interface to verify
test("createUser saves to database", async () => {
  await createUser({ name: "Alice" });
  const row = await db.query("SELECT * FROM users WHERE name = ?", ["Alice"]);
  expect(row).toBeDefined();
});
```

### Mocking, in one paragraph

Mock at the edge of *your* system, not inside it. The HTTP client to a third-party API, the system clock, the random source, the filesystem when the test would otherwise be slow or non-deterministic — those are fair game. Your own services, repositories, and helpers are not. If a test needs internal mocks to pass, that's a signal the module under test is too coupled to its collaborators, not a signal to add more mocks.

## What "done" looks like

There is no fixed end. The user decides when to stop. When they do, briefly state which tests you wrote (and are now failing as expected) and which you were planning next, so they can resume later. Then hand control back without proposing more.
