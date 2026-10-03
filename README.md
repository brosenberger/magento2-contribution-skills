# magento2-contribution-skills

Agent skills for contributing upstream to **magento/magento2** and **magento/inventory** — building the source checkout, reproducing a defect against trunk, and getting a pull request into a shape maintainers can act on.

**Not project work.** A fix that only has to run on one install is a different job with different rules; see [magento-integration-skills](https://github.com/brosenberger/magento-integration-skills) for feeding data into Magento, and the ordinary Magento module conventions for building against it.

## The set

| Skill | Covers |
|---|---|
| `magento2-core-contribution` | Whether the behaviour is a defect at all, red-before-green against trunk, the ways integration tests mislead, pull request shape, and the environment traps |

## Why it exists

Upstream work fails in ways project work does not:

- **Trunk is not any release.** A defect can be fixed on trunk and live in every release, or the reverse. Reproducing against the branch you are about to change is the whole job, and it is routinely skipped.
- **Load-bearing behaviour looks like a bug.** Stock and inventory code is full of values serving two purposes at once. When a stored flag means both "the platform decided this" and "a person decided this", no local change is correct — and the right output is an issue documenting the trade-off, not a pull request that gets argued with and closed.
- **Integration tests lie in a specific way.** Every simulated request runs in one PHP process, so request-scoped caches survive between calls you are treating as independent. Defects vanish; fixes appear to work.
- **The environment has a dozen small traps** — fork identity, workflow scopes, path-repository ordering, a test database nobody creates for you — none of which announce themselves clearly.

## Install

```sh
git clone https://github.com/brosenberger/magento2-contribution-skills.git
cd magento2-contribution-skills
bin/install-symlinks.sh              # -> ~/.claude/skills
bin/install-symlinks.sh ~/.cursor/skills
```

Per-folder symlinks; existing local entries are never overwritten.

## Provenance

Everything asserted was measured against `magento/magento2@2.4-develop` and `magento/inventory@develop` in August 2026, in a DDEV source checkout with the Inventory repository wired in as editable path packages — not read off documentation. The work that produced it is visible in [magento/magento2#41175](https://github.com/magento/magento2/pull/41175) and [magento/inventory#3466](https://github.com/magento/inventory/issues/3466), including the case where the correct answer turned out to be an issue rather than a fix.

## Licence

MIT — see [LICENSE](LICENSE).

---

More Magento modules and write-ups: [brocode.at](https://brocode.at/modules/)
