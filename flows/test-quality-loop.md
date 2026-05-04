# The Test-Quality Loop

A methodology for ensuring unit tests would actually catch regressions. The execution is automated by the [`verify-tests`](../skills/verify-tests.md) skill; this document explains the *why*, the *when*, and how to read the output.

Read this once. The skill does the rest.

---

## The problem coverage doesn't solve

Line / branch / statement coverage measures whether a test **executes** code. It says nothing about whether the test **asserts** what that code is supposed to do. Three concrete patterns that pass any coverage gate while letting real regressions through:

1. **Absence-only assertions.** A test titled `"renders Spinner instead of title, disables press"` that only asserts `queryByText('Go')` is null. The label can disappear for any number of reasons unrelated to the spinner — coverage hits 100%, the regression slips.
2. **Vacuous "doesn't throw" checks.** `expect(() => render(<Component />)).not.toThrow()` is fine for a no-state smoke test, but for anything with props that affect output it asserts almost nothing — flip a ternary in the source, the test still passes.
3. **Weak shape matching.** `expect.any(Object)` matches any non-null reference. If the source's contract says "pass the parsed error payload" but a regression sends the raw `Error` instead — both satisfy `Object`, the test passes for the wrong reason.

In each case the source has changed in a way the contract forbids, the test ran the affected line, and the assertion was too loose to notice. The 90%-coverage gate is green; the bug ships.

## What mutation testing does

A mutation runner takes the source code and applies small, mechanical changes — **mutations** — one at a time:

- Conditional flips: `>` → `>=`, `===` → `!==`
- Boolean inversions: `true` → `false`, `if (x)` → `if (!x)`
- Logical operator swaps: `&&` → `||`
- Argument and return-value drops: `f(a, b)` → `f(a)`, `return x` → `return null`
- Block elimination: function body → empty
- String/array/number constant replacement: any → `""` / `[]` / `0`

For each mutation, the runner re-runs the test suite. Outcome per mutant:

- **Killed** — at least one test failed. The test caught the change. Good.
- **Survived** — the suite still passed. The test executed the mutated line but didn't assert the post-condition the mutation broke. **This is the test gap.**
- **NoCoverage** — no test reached the line at all. Coverage gap, distinct from a shallow test.
- **Timeout** — the mutation produced an infinite loop or hang. Usually counts as killed; suspicious only when the mutator was a logical operator.

The mutation score = killed / (killed + survived + nocoverage + timeout). This is a much better fitness function for a test suite than coverage. It directly measures the suite's regression-catching power.

## Where it fits in the dev cycle

```
write source code  →  write tests  →  coverage green  →  mutation testing  →  fix survivors  →  commit
                                       ▲                  ▲                   ▲
                                       gate #1            gate #2             gate #3
                                       (existing)         (this loop)         (manual review)
```

Trigger the loop:

- **After writing tests for a new feature.** Primary case — formalizes the "mentally plant a regression" check.
- **After writing a regression test for a bug fix.** Confirms the new test would have caught the original bug.
- **Before merging a large refactor.** A refactor that doesn't change observable behavior should not change the mutation score; if it drops, you've actually changed behavior somewhere.
- **When code review flags a suspicious test pattern.** Proves the pattern empirically rather than arguing about it.

Do NOT trigger:

- On legacy code without unit tests. Mutation testing on a 0-coverage suite reports 100% NoCoverage — uninformative noise. Write tests first.
- On prototypes / spike branches / throwaway scripts. Cost > value.
- On generated code (codegen, scaffolding). Mutators on machine-generated code produce false-positive survivors.
- On tests that are themselves the spec (i.e. the source is a thin pass-through). The mutation score will be artificially low and there's nothing to fix.

## Reading a mutation report

For each survivor the runner gives you:

1. **Location** — `file:line:column`.
2. **Mutator** — what kind of change was applied (`ConditionalExpression`, `BooleanLiteral`, etc.).
3. **Original / mutated source** — both versions, side by side.
4. **Covering tests** — the tests that ran but didn't fail.

Your job (or the skill's, automated) is to figure out: *why didn't any of those tests fail?* Almost always one of:

- The test asserts presence/absence of something other than what the mutation broke.
- The test asserts a loose matcher (`expect.any(...)`, `toBeTruthy()`, `not.toBeNull()`) where a tight one is needed.
- The test calls the function but never inspects its return value or side effect.
- The test happens to use input data that yields the same result for both the original and mutated code (boundary not exercised).

The fix is almost always **adding one assertion**, not rewriting the test. If you're tempted to delete and rewrite, stop — the existing test asserts *something* useful, and your replacement should preserve that while adding what's missing.

## Equivalent mutants — the unavoidable noise

Some mutations don't actually change observable behavior. Common cases:

- **Logging.** `logger.debug('foo')` → no-op. The mutation can't be killed without testing the logger, which usually isn't worth it.
- **Defensive checks.** `if (x == null) return early` against an internal call site that proves x is non-null at the type level. The mutation is unreachable.
- **Commutative operations.** `a + b` → `b + a` for numeric addition. Mathematically identical.
- **Refactor-equivalent expressions.** `Array.from(set)` → `[...set]` — same output, different bytes.

Equivalent mutants are real but rare. Suppress them via the runner's per-line exclusion mechanism (`// Stryker disable next-line ConditionalExpression: defensive check, x is non-null per caller contract`) — and **always** include a one-line justification. Opaque suppressions are tech debt and hide real survivors over time.

Rule of thumb: if you can't articulate why a mutant is equivalent in one sentence, it isn't equivalent.

## Realistic targets

| Mutation score | What it means |
|---|---|
| < 60% | Tests are mostly executing without asserting. Something is structurally wrong. |
| 60–75% | Typical of a fresh suite. Many survivors, easy wins available. |
| **75–85%** | **Practical target.** Above this, returns diminish sharply. |
| 85–90% | Strong suite. Most remaining survivors are equivalent or genuinely hard. |
| > 90% | Either an exceptionally well-tested codebase or noise tests inflating the score. |
| 100% | Almost always the wrong goal — chasing the last mutants produces tests that assert implementation, not behavior. |

Don't gate on absolute score. Gate on **delta** — the mutation score on the changed lines must not drop. A team can have 70% overall mutation score and still ship safely, as long as every PR's diff is covered well.

## CI integration patterns

Three modes, in order of recommended adoption:

### Mode 1 — Warn-only (start here)

Run mutation testing in CI on the diff against the merge base. Post the report as a PR comment. Don't fail the pipeline. Use this for one to two weeks to:

- Get a sense of how often surviving mutants appear in real PRs.
- Surface equivalent mutants and triage them once.
- Build team familiarity with reading mutation reports.

### Mode 2 — Threshold gate on the diff

Once the team is comfortable, fail the pipeline if any survivor exists on lines added/modified in this PR. Pre-existing survivors don't block — they're a separate followup track. This is the sustainable steady-state mode.

Important: gate on **diff-line survivors**, not on absolute mutation score. A score-based gate punishes whoever happens to touch a poorly-tested area; a diff-based gate keeps the responsibility scoped to whoever introduced the gap.

### Mode 3 — Trend gate

After the codebase stabilizes, also fail if the project-wide mutation score regresses by more than ε between merges. This catches the case where a refactor accidentally weakens existing tests without showing up in the diff-line check.

Don't jump straight to Mode 3. It looks rigorous on paper and produces six-month projects to clean up legacy survivors that nobody asked for.

## Toolchain pointers

Concrete runners per ecosystem. The [`verify-tests`](../skills/verify-tests.md) skill detects which one to use; this table is for human reference.

| Ecosystem | Tool | Docs |
|---|---|---|
| JavaScript / TypeScript / Jest / Vitest | StrykerJS | https://stryker-mutator.io/docs/stryker-js/introduction |
| Python / pytest | mutmut | https://mutmut.readthedocs.io |
| Python (async-heavy / decorator-heavy) | cosmic-ray | https://cosmic-ray.readthedocs.io |
| Java / Maven or Gradle | PIT | https://pitest.org |
| .NET / xUnit / NUnit / MSTest | Stryker.NET | https://stryker-mutator.io/docs/stryker-net/introduction |
| Scala | Stryker4s | https://stryker-mutator.io/docs/stryker4s/getting-started |
| Go | go-mutesting / Gremlins | https://github.com/zimmski/go-mutesting , https://gremlins.dev |
| Rust | mutants | https://mutants.rs |
| Swift / iOS | muter | https://github.com/muter-mutation-testing/muter |

If your stack isn't listed, the methodology still applies — search for "mutation testing &lt;language&gt;" and pick the most maintained option.

## Common objections (and answers)

**"Mutation testing is too slow."**
True for full-repo runs. The skill scopes to the diff — usually under five minutes. Run it locally before commit, not in pre-commit hook.

**"Equivalent mutants are too noisy."**
They exist but are rare in practice. Most "this is equivalent" claims dissolve under one minute of inspection — the test really was missing an assertion. Treat the equivalent claim as a hypothesis to disprove first.

**"We already have 95% coverage and feel safe."**
Coverage 95% with mutation score 60% is normal. The gap between the two numbers is exactly the false-confidence zone — code that runs without being checked.

**"Tests are already a bottleneck. This adds more."**
Adding one assertion to an existing test is cheaper than the bug it would have caught hitting production. The skill suggests the assertion; you don't have to design it from scratch.

**"Won't this lock in implementation details?"**
Only if you let it. Resist suggested fixes that test internal call ordering or private state — those are noise. Keep assertions on observable outputs (return values, rendered DOM, side effects on mocks of external dependencies). Mutants that only die under implementation-level tests are usually equivalent at the behavior level.

## Cross-references

- The skill that runs the loop: [`skills/verify-tests.md`](../skills/verify-tests.md).
- The reviewer that flags suspicious test patterns this skill then proves empirically: [`shared/agent-code-reviewer.md`](../shared/agent-code-reviewer.md).
- For setting up a project's `workflow.md` to invoke this skill at the right time: [`claude-code-project-setup.md`](../claude-code-project-setup.md).
