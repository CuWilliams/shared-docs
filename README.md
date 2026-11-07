# shared-docs

Documentation that belongs to more than one repository.

The rule this repo exists to enforce: **if two repos need the same document, neither of them owns
it.** A copy in each is a fork waiting to happen, and the forks are discovered at the worst
possible moment — when one of them says something that is no longer true.

---

## What's here

### [`meta/`](meta/) — repo-agnostic, any project

Tooling for writing and maintaining a `CLAUDE.md`. Applies to every repository, not just one
product. Eleven repos currently carry a `CLAUDE.md`.

- **[CLAUDE_MD_TEMPLATE.md](meta/CLAUDE_MD_TEMPLATE.md)** — standard structure
- **[CLAUDE_MD_CHECKLIST.md](meta/CLAUDE_MD_CHECKLIST.md)** — maintenance checklist
- **[check-claude-md.sh](meta/check-claude-md.sh)** — health check script

```bash
# From any repo root:
/path/to/shared-docs/meta/check-claude-md.sh
```

It finds the guide file whatever its casing — `CLAUDE.md`, `claude.md` and `Claude.md` are all in
use — or takes an explicit path as its argument.

### [`gfpricechecker/`](gfpricechecker/) — the GFPriceChecker product

Documents shared by the iOS app, the website, and any future Android app.

- **[CONTENT_CONVENTIONS.md](gfpricechecker/CONTENT_CONVENTIONS.md)** — how the product talks
  about what it does. **Read this before writing user-facing copy anywhere.** The first rule is
  legal exposure, not style.
- **[PRODUCT_DEFINITION.md](gfpricechecker/PRODUCT_DEFINITION.md)** — what the product is,
  independent of platform: the data model, the differential arithmetic, the export formats, and
  what a second implementation must match versus what it is free to change.

---

## Consuming repos

| Repo | Uses |
|---|---|
| `CuWilliams/GFPriceChecker` (iOS) | `meta/`, `gfpricechecker/` |
| `CuWilliams/GFPriceChecker.github.io` (website) | `meta/`, `gfpricechecker/` |
| *future Android app* | `meta/`, `gfpricechecker/` |
| the other eight repos with a `CLAUDE.md` | `meta/` |

Consuming repos **link** here. They do not copy. A repo that needs a local reminder should carry
a pointer of a line or two, never a duplicated document.

---

## How to change something

1. Change it here.
2. Check whether any consuming repo now contradicts it — for the GFPriceChecker content rules,
   run the sweep in
   [CONTENT_CONVENTIONS.md](gfpricechecker/CONTENT_CONVENTIONS.md#the-sweep) in each repo.
3. Fix the contradictions in the same pass. A shared document that half the repos ignore is worse
   than no shared document, because it looks authoritative.

If you find yourself wanting to copy a file out of here into a product repo, that is the signal
the document is in the wrong place — either it is not actually shared, or the consuming repo needs
a link instead.

---

## Changing main

`main` is the source of truth for three repos, so changes should arrive as a PR with a
reviewable diff rather than a direct push.

**GitHub enforces this.** A ruleset on `main`, "Protect main"
([settings](https://github.com/CuWilliams/shared-docs/rules/26766634)), requires a pull request
(zero approvals, since a sole owner cannot approve their own PR), blocks force pushes, and
blocks deleting the branch. Nobody is on its bypass list, the owner included. A direct push is
refused with `GH013: Repository rule violations found`. It was turned on in November 2026, once
the repo went public. While it was private on the Free plan, both classic branch protection and
rulesets returned `403 Upgrade to GitHub Pro or make this repository public`.

There is also a `pre-push` hook in [`hooks/`](hooks/), which catches the mistake locally before
anything reaches GitHub. It is opt-in per clone:

```bash
git config core.hooksPath hooks
```

Run that after cloning, or the hook does nothing. It is bypassed by `--no-verify` and exists
only in clones that ran the command above. That is fine now: the ruleset is the lock, and the
hook just gives a faster, clearer refusal than the server's.

---

## History

`meta/` moved out of `GFPriceChecker/docs/` in September 2026 — see
[GFPriceChecker#47](https://github.com/CuWilliams/GFPriceChecker/issues/47). Those three files had
never been about GFPriceChecker; they were parked there because that is where they were written,
and the old `docs/README.md` actively instructed the reader to copy them into every new repo.

`gfpricechecker/` was written at the same time. The content conventions had already forked once —
the website removed its CRA-compliance claims in v1.2.0 while the app's README and development
guide went on making them for months.
