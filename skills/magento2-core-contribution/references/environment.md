# Building the checkout

A source checkout of core with the Inventory repository wired in as editable packages. Every trap below cost real time on a first build; none of them announces itself clearly.

## Forks and identity

**A pull request needs a fork, so the fork is the clone origin and the branch base.** Clone the fork, add the upstream repository as a second remote, and cut the branch off the freshly fetched upstream ref.

**Check which account your SSH key authenticates as** before assuming a push will work — `ssh -T git@github.com` names it. A machine that carries a work key will authenticate as that account and be denied on a personal fork, with an error that reads like a permissions problem rather than an identity one. HTTPS with the `gh` credential helper avoids the ambiguity entirely.

**Set the commit identity per repository.** The contributor licence agreement is matched against the commit author's email, so a global work address on a personal contribution fails the check after the pull request is already open.

**An old fork will block the first push.** Pushing a branch based on current trunk onto a fork whose branches are years stale necessarily writes `.github/workflows/` files, and an OAuth token without the `workflow` scope is refused:

```
refusing to allow an OAuth App to create or update workflow ... without `workflow` scope
```

The same error blocks the API-based fork sync, so it cannot be worked around by syncing first, and the fork will not have upstream's commit objects to seed a ref from. Grant the scope (`gh auth refresh -h github.com -s workflow`) or fork fresh. A newly created fork mirrors upstream and never hits this.

## Wiring the Inventory repository

Clone it inside the core checkout and declare it as a **path repository** so its modules symlink into `vendor/` and edits are live.

**Repository order matters.** Adding a repository appends it, so the packagist-style repository can end up ahead of the path repository and shadow it — you then edit files that are not the ones running. Put the path repository first, then verify by checking that the vendor directory entry really is a symlink into your checkout. Do this before writing any code; everything downstream depends on it.

## Local-only files must never reach a pull request

The composer manifest and lock carry the path-repository wiring. Freeze them:

```sh
git update-index --skip-worktree composer.json composer.lock
```

That has teeth. Status and diff go silent on those files, and checkout, rebase and pull **fail** whenever the incoming ref touches them — which trunk does often. Before any rebase:

```sh
git update-index --no-skip-worktree composer.json composer.lock
git stash push -- composer.json composer.lock
#  … rebase …
git stash pop
git update-index --skip-worktree composer.json composer.lock
```

Exclude the rest locally — the container config, the Inventory checkout directory, the test install config, any agent notes. Core's own ignore file does not cover them. Then assert before every push that the diff against upstream contains none of them.

**Never put marketplace credentials inside the checkout.** A public fork is the wrong place for an auth file, whatever the tooling's per-project convention suggests. Use the global slot.

## Running the tests

**The integration test database must exist first.** The framework does not create it, and it says so on the first line of output — above roughly ten kilobytes of command usage dump, which is very easy to scroll past. If the first integration run dies, read line one.

**Do not copy a `disable-modules` list from a project install.** A source checkout has no two-factor, Adobe IMS or sample-data modules — they are separate packages, absent here — and naming one fails the install with `Unknown module in the requested list`.

**Inventory integration tests are not in the default suite.** It globs the test suite directory and `app/code/*/*/Test/Integration`; the Inventory modules live in `vendor/` behind a symlink and match neither. Pass an explicit absolute path. Unit tests *are* found, because that suite globs `vendor/` too — so the two behave differently and it is not obvious which you are hitting.

**Never run two integration suites at once.** They share one database and will corrupt each other's results silently. A run that produces no summary line was probably clobbered.

## Reading a failed run

Filter the noise, or the actual message is invisible:

```sh
… 2>&1 | grep -vE '^\s*#[0-9]+ |setup:install \[--'
```

Framework install failures print the real cause first and the stack trace after, so `head` beats `tail` on those.
