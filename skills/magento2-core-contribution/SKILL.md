---
name: magento2-core-contribution
description: >-
  Use when contributing a fix or test upstream to magento/magento2 or magento/inventory — building the source checkout, reproducing a defect against trunk, and getting a pull request into a shape maintainers can act on. Covers the environment traps that waste a day, the discipline that separates a real defect from intended behaviour, and when to file an issue instead of a PR.
---

# magento2-core-contribution — upstream work that survives review

For contributing to **magento/magento2** and **magento/inventory** themselves. Not for project work; a fix that only has to run on one install is a different job with different rules.

## The one rule

**Reproduce it against trunk before you fix it, and prove the reproduction fails for the reason you think.** Almost everything expensive in upstream work comes from skipping this: fixing behaviour that was already fixed, fixing behaviour that is intentional, or writing a test that passes for the wrong reason and proves nothing.

A bug report is a claim about someone else's install on some version. Trunk is a different codebase from every release, and both directions bite: a defect can be fixed on trunk and live in releases, or introduced on trunk and absent from releases. Measure the one you are about to change.

## Is it a defect, or is it intended?

The single most useful question, and the one most likely to end the work early.

Before proposing a change, find out **what else depends on the behaviour**. Existing tests are the cheapest oracle: change the behaviour locally, run the module's suite, and read what breaks. A test that asserts the thing you were about to "fix" is a maintainer telling you it is deliberate.

Distinguish three cases, because they need different outputs:

| What you found | What to ship |
|---|---|
| Behaviour nothing depends on, contradicted by its own docs or siblings | A pull request |
| Behaviour that is load-bearing, but its consequences are unwanted | An issue with the trade-off laid out, not a PR |
| Behaviour that is simply intended | A documentation suggestion, or nothing |

The second case is common in inventory and stock code and is worth recognising early. When a value serves two purposes and cannot distinguish them — the same stored flag meaning both "the platform decided this" and "a person decided this" — no local change can be correct, because every fix trades one caller's needs against another's. That is a design decision. Filing it as one, with the measurements that close off each option, is more useful than a PR that will be argued with and closed.

**Symptom to watch for:** you find yourself needing to rewrite an existing test's expectations to make your change pass. That is sometimes right, and it is far more often the codebase telling you the change is wrong. Never do it on your own reasoning alone — check what the test was added for.

## Red before green, and verify the red

Write the failing test first, then read the failure message before writing any production code.

- **A red test that fails on a fixture, bootstrap or import error is broken, not red.** It tells you nothing about the defect. Fix the harness, get a failure that names the real values, then proceed.
- **A test that passes before you change anything does not test the defect.** Sharpen it or delete it. This is also how you find out that trunk already fixed the thing you were sent to fix.
- **Assert the premise.** If the reproduction depends on a starting state, assert that state explicitly with the actual values in the message. A reproduction that quietly sets itself up wrong is the most expensive failure mode here, because it looks like evidence.
- **Write the tests that must not change, in the same pass.** They are expected green from the start and they are what catches you breaking intended behaviour at the moment you do it, rather than in review.

The red run output is not a by-product. It is the reproduction the issue needs and the "Manual testing scenarios" the PR template demands, so capture it.

## Integration tests lie in one specific way

Every "separate request" in an integration test runs in **one PHP process**, so request-scoped caches and in-memory markers survive between the calls you are treating as independent. A defect that only appears across real request boundaries will not reproduce, and a fix that only works because of a warm cache will look correct.

When simulating a sequence of API calls, clear the relevant registry or cache between steps, and say in a comment why. Read persisted rows rather than loaded models when you are asserting on state a previous step wrote — a repository round trip may be served from cache and report what the process last wrote rather than what was stored.

## Fixtures decide what you are testing

Modern attribute fixtures merge their defaults into whatever you pass. `Catalog\Test\Fixture\Product` supplies stock data by default and that propagates into the composite fixtures built on top of it, so "a product created without stock" is not what you get unless you override it explicitly.

Check the defaults of every fixture in the chain before trusting a reproduction. If the premise assertion above fails, this is usually why.

Prefer framework-managed fixtures over hand-rolled ones. They revert themselves, and they avoid the nested-transaction failure that raw SQL setup provokes when database isolation is on.

## Pull request shape

Match the repository you are editing, not your own conventions:

- **Copyright headers follow the file you are touching.** Never introduce your own licence header into an upstream file.
- **Squash to one commit.** The red-then-green sequence is your local workflow; it does not belong in the published history. The evidence goes in the issue and the PR body.
- **Fill the template properly.** "Manual testing scenarios" means numbered steps someone can follow without asking you anything, including how to read the result out of the database.
- **Lead the description with why the change is not new behaviour.** If the codebase already applies your rule somewhere — a sibling class doing the same thing in a narrower scope — cite it. It reframes the PR from "a proposal" to "make this consistent", which is a much shorter argument.
- **Say what you deliberately did not change,** especially where a reviewer would expect you to have changed it. Duplicated logic you left alone, a wider fix you scoped out, a constructor you did not touch for backward compatibility.

**Backward compatibility is cheaper to respect than to argue about.** Do not delete a public class or change a constructor signature to tidy up. Deprecate, unregister from `di.xml`, leave the class. Magento's coding standard also requires a `@see` alongside every `@deprecated`.

Both repositories close a pull request after **two weeks** without a contributor response. Do not open one you cannot watch.

## JavaScript defects

Magento's JS unit layer (`dev/tests/js/jasmine`) needs a full `package.json.sample`
npm + grunt build before it runs a single spec. There is a cheaper harness that
exercises the real module, and a trunk-equality check that licenses using it.
See [`references/js-defects.md`](references/js-defects.md).

## Environment

Building the checkout has its own set of traps, none of them interesting and all of them expensive. They are in [`references/environment.md`](references/environment.md) — read it once before the first build rather than discovering them one at a time.

## Verification

Anything asserted here was measured against `magento/magento2@2.4-develop` and `magento/inventory@develop` in August 2026, in a DDEV source checkout with the Inventory repository wired in as editable path packages.
