# shared-docs - Development Guide

**Documentation owned by more than one repository**

---

## Project Overview

This repo holds documents that two or more other repos need. It has no build, no tests and no
code beyond one shell script. Its whole job is to be the single place a shared document lives, so
that the copies in consuming repos never get a chance to drift apart.

**Current Status:** Created September 2026, out of
[GFPriceChecker#47](https://github.com/CuWilliams/GFPriceChecker/issues/47).

---

## Project Structure

```
shared-docs/
├── meta/                        # repo-agnostic: applies to any project
│   ├── CLAUDE_MD_TEMPLATE.md
│   ├── CLAUDE_MD_CHECKLIST.md
│   └── check-claude-md.sh
└── gfpricechecker/              # the GFPriceChecker product: iOS + web + future Android
    ├── CONTENT_CONVENTIONS.md
    └── PRODUCT_DEFINITION.md
```

The split is by **audience**, not by topic. `meta/` is read by anyone working in any of the eleven
repos that carry a `CLAUDE.md`. `gfpricechecker/` is read by anyone working on that one product,
whichever platform they are on.

---

## The rule this repo exists to enforce

**If two repos need the same document, neither of them owns it.**

A document is a candidate for this repo when changing it in one place would leave another repo
saying something untrue. That is a stricter test than "both repos mention it" — plenty of
duplication is harmless. What is not harmless is a *claim*: about what the product does, what it
is compliant with, what the data model is, or what the privacy posture is.

Two things that earned their place here:

- **The CRA-compliance rule.** The website removed that class of claim in its v1.2.0 release. The
  app's README and development guide went on making it for months, because the rule lived in the
  website's `Claude.md` and nothing pointed the app at it.
- **The CLAUDE.md meta-tooling.** Parked in `GFPriceChecker/docs/` where it had nothing to do with
  the app, while that repo's `docs/README.md` told the reader to copy the files into every new
  project.

---

## Development Guidelines

### Changing a shared document

1. Change it here first.
2. Sweep the consuming repos for statements that now contradict it. For the GFPriceChecker content
   rules the sweep is in `gfpricechecker/CONTENT_CONVENTIONS.md`.
3. Fix the contradictions in the same pass — not "later". A shared document that half the repos
   ignore is worse than none, because it reads as authoritative.

### Adding a document

Before adding anything, answer: **which repos read this, and what breaks if they disagree?** If
the answer is "only one repo", it belongs in that repo. This place works only if everything in it
is genuinely shared; padding it with one-repo documents is how it becomes another archive nobody
reads.

### Pointers, not copies

Consuming repos link here. If you are about to `cp` a file out of this repo into a product repo,
stop — either the document is not actually shared, or the consuming repo needs a two-line pointer
instead.

### Accuracy over completeness

`gfpricechecker/PRODUCT_DEFINITION.md` describes **shipped iOS behaviour**, not an aspiration. When
iOS changes, this changes with it. Never write a specification here that no implementation has
met — a document that mixes what is true with what is planned is worse than one that admits it
covers only part of the ground.

---

## Consuming repos

| Repo | Reads |
|---|---|
| `CuWilliams/GFPriceChecker` (iOS) | `meta/`, `gfpricechecker/` |
| `CuWilliams/GFPriceChecker.github.io` | `meta/`, `gfpricechecker/` |
| *future Android app* | `meta/`, `gfpricechecker/` |
| eight other repos with a `CLAUDE.md` | `meta/` |

---

## Known Limitations

- **Nothing enforces the link.** A consuming repo can contradict this one and no build fails. The
  grep sweep in `CONTENT_CONVENTIONS.md` is the only check, and it is run by hand.
- **`check-claude-md.sh` scores structure, not truth.** It counts lines and looks for section
  headings. A `CLAUDE.md` can score 6/6 while confidently stating something false, which is the
  failure that actually costs time.
- **The consuming repos are private or separate.** There is no submodule and no sync. That is
  deliberate — a submodule would be a heavier commitment than the content warrants — but it means
  the pointers are only as good as the person maintaining them.

---

*Maintained by Claude Code alongside the repos that consume it.*
