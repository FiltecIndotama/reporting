# Filtec Sales Dashboard — working notes

## Where this ships

The live dashboard is:

```
https://reporting-rouge-six.vercel.app/filtec-sales-dashboard.html
```

Built by the Vercel project `filtec1/reporting`, which deploys from `main`.

**Every change must end up at that URL.** A change sitting on a branch, or in an
open pull request, is not finished — merging to `main` is what publishes it.
Standing instruction from the repo owner: always land work on `main` so the live
link updates.

The normal route is branch → pull request → merge, which also gives a Vercel
preview deployment (the bot comments the link on the PR) to check before
publishing. Confirm before merging only if the change is risky or the owner
hasn't asked for it yet.

Note: the dashboard footer credits `paulsworld.vercel.app`. That is site
branding, not the deploy target — leave it alone unless asked.

## What's here

| Path | What it is |
|---|---|
| `filtec-sales-dashboard.html` | The entire site — one file, ~2,570 lines. HTML, CSS and JS inline. No build step, no framework, no npm. |
| `docs/adding-a-new-tab.md` | Training manual for adding a tab. Read it before editing the dashboard. |
| `docs/adding-a-new-tab.html` | Rendered copy of the manual. Keep in sync if you change the `.md`. |

Two external scripts (`/view-counter.js`, `/sw-register.js`) load at the end.
They live outside this repo — they 404 when you open the file locally, which is
expected and not a bug.

## House rules

- **Read `docs/adding-a-new-tab.md` first.** It documents the four edit points
  for a new tab and the mistakes that silently break the page — notably the
  hard-coded array in `setPage()`, which leaves a dead tab button if you forget
  it.
- **Trilingual, always.** Every user-visible string needs `.en`, `.zh` and `.id`
  spans, adjacent with no whitespace between them. A missing translation looks
  perfect in English and leaves a blank gap for everyone else.
- **Reuse the components and design tokens** listed in the manual's section 6
  rather than inventing CSS. That is what keeps the tabs looking like one
  product.
- **Charts and tables are numbered globally**, not per tab, so a figure can be
  cited across tabs. Never reuse a number.
- **Test in a browser before merging.** There is no test suite; the manual's
  section 8 checklist is the test suite. Headless Chromium is available at
  `/opt/pw-browsers/` — driving it with Playwright catches dead tab buttons,
  unbalanced `<div>`s and missing translations that reading the diff will not.
