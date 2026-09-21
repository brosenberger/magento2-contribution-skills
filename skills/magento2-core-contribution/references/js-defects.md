# JavaScript defects

## A release sandbox can stand in for trunk — after you prove it

`diff` the exact file you are changing between trunk and the installed release
before using a release install as your harness. Byte-identical means a red/green
run there is evidence about trunk; anything else and you are measuring a
different codebase. This is the trunk-reproduction rule applied per file, and it
is what makes a throwaway sandbox legitimate instead of sloppy.

## Use the storefront as the spec runner

`pub/static/.../<Module>/js/<file>.js` is a **symlink into `vendor/`** on a
developer-mode install. Swapping the vendor file swaps what the browser loads —
no static redeploy, no cache flush, no npm build. So:

1. Run the spec bodies against the real RequireJS module in a live page
   (`require(['jquery','Magento_Ui/js/modal/modal'], ...)`) and capture red.
2. Copy the patched file over the vendor one. Re-run. Capture green.
3. Restore the vendor file and verify with `diff`.

Drive it with WSL2-native Playwright; `*.ddev.site` resolves to 127.0.0.1 via
public DNS, so a DDEV site is reachable with `--ignore-certificate-errors` and
nothing else. Also run the file's pre-existing specs against the patched build —
that is the regression half, and it is what the PR checklist is asking about.

## Broken red, the JS flavour

A widget's methods branch on DOM state, so a spec that calls the internals in the
wrong order fails for the wrong reason. Two of three specs here first failed
because `_getVisibleCount()` still counted the modal, so teardown never ran —
the assertion tripped, not the defect. Reproduce the **real call sequence**
(`openModal()` → `closeModal()` → `_close()`), not a convenient shortcut, and
read the failure text before believing the red.

## Armed but inert

A defect can be reachable by a normal gesture and still never fire on the stock
theme. Here a checkout pencil click armed a `transitionend` handler on a modal
whose overlay was `undefined`, but stock Luma has zero transitions in that
subtree, so it never detonated. Measure both facts separately: what arms it, and
what fires it.

This changes the PR, not just the analysis. The argument becomes "latent hazard
any theme detonates" rather than "users see this", and the manual testing
scenario has to tell the reviewer how to make it fire — otherwise they follow
your steps, see a clean console, and close the PR.
