# GFPriceChecker — shared product documentation

Shared by the iOS app, the website, and any future Android app.

- **[CONTENT_CONVENTIONS.md](CONTENT_CONVENTIONS.md)** — how the product talks about what it does.
  Read before writing user-facing copy in any repo. The first rule is legal exposure.
- **[PRODUCT_DEFINITION.md](PRODUCT_DEFINITION.md)** — the platform-independent product: data
  model, differential arithmetic, export formats, privacy posture, and the line between what a
  second implementation must match and what it may change.
- **[vectors/](vectors/)** — the cross-platform test vectors: inputs and expected outputs that
  every implementation must pass. GFPriceChecker vendors a copy and runs it as `VectorTests`.

## Which repo owns what

| Lives here | Lives in the product repo |
|---|---|
| How the product describes itself | How a given platform implements it |
| The data model and the arithmetic | Framework choices, architecture, UI |
| Vocabulary and tone | Repo-specific build and test instructions |
| Privacy posture as a constraint | Repo-specific patterns and gotchas |

A product repo's `CLAUDE.md` should link here for the first column rather than restating it.
Restating is how the website and the app came to disagree about CRA claims for several months.
