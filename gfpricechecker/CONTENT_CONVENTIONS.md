# GFPriceChecker — Content Conventions

**This is the single source for how GFPriceChecker talks about what it does.** It governs the iOS
app, the website, and any future Android app. If a rule here and a rule in a repo's own CLAUDE.md
disagree, this file wins; fix the repo.

These are not style preferences. The first rule is legal exposure.

---

## 1. Don't claim tax compliance

The app carries no Canada Revenue Agency branding, and agency references and specific tax line
numbers were stripped out of the iOS interface in **April 2026**. The website removed the same
class of claim in its **v1.2.0** release. Tracking price differences is what the app does; what a
user does with those records at tax time is between them and their accountant.

**Never:**

- Describe the app's exports as CRA-compliant, CRA-approved, accepted, endorsed, or affiliated.
- Promise anyone a deduction, or imply the app determines eligibility.
- Print a tax line number (`Line 33099` and friends) in the interface, in marketing copy, or in
  a repo's development guide.

**Fine:**

- Explaining the Medical Expense Tax Credit as context for why the app exists.
- Saying the app documents the incremental cost of eating gluten free.
- Noting that the credit only applies to medical expenses above a threshold of roughly **three
  percent of net income**, so a modest year may not reach it. Say this where it is relevant —
  it manages expectations honestly.

### The sweep

Before merging copy or documentation changes, in any GFPriceChecker repo:

```bash
# Quote the --include patterns: zsh expands them as globs otherwise.
grep -rn "CRA\|Tax Compliant\|tax compliance\|Line 33099" \
  --include='*.swift' --include='*.kt' --include='*.html' \
  --include='*.json' --include='*.md' .
```

Every surviving hit should be explanatory context, or a dated historical record that admits its
date. Nothing should be a claim about what the app's output *is*.

### The standing exceptions

Historical records narrate what the app was at the time of writing and are left as written. They
are corrected with a note, never silently rewritten:

- **`data/blog.json` in the website repo.** The blog is a first-person record that narrates its
  own corrections in sequence — the 2026-01-24 post calls the exports "a clean, CRA-ready
  document," and the 2026-04-06 post is the author explaining that he pulled exactly that framing
  out of the app. Rewriting the earlier post would erase the correction.
- **`docs/DEVELOPMENT_HISTORY.md` and `docs/archive/` in the app repo.** Session titles and commit
  messages are facts about what happened. Both carry banners stating where the record stops.

---

## 2. What the app is

One sentence, for when you need one:

> GFPriceChecker documents the price difference between gluten-free products and their regular
> equivalents, so the incremental cost is recorded if you claim it as a medical expense.

**Tagline:** Compare, Save, Claim

Note the verbs. The app *documents*, *tracks*, *records*, and *calculates*. It does not *file*,
*submit*, *certify*, *guarantee*, or *comply*.

---

## 3. Vocabulary

Consistent terms across all three platforms. Pick the left column.

| Use | Not |
|---|---|
| product pair | comparison, item pair, entry |
| gluten-free product / GF product | the expensive one, the special one |
| regular equivalent | normal product, non-GF (in user-facing copy) |
| differential | markup, premium, surcharge |
| unit price | price per unit, normalized price |
| receipt verification | proof, validation, audit |
| records / your records | tax documents, submission, filing |
| export | report generation, filing |
| celiac disease | coeliac (this is a Canadian product) |

**Unit pricing** is `$/100 g` for weights and `$/unit` otherwise. Say "adjusted for package size"
when explaining why it exists — that is the honest reason, and it is the app's actual method.

---

## 4. Privacy posture

True on iOS today, and a constraint on any Android implementation rather than a description of
one:

- All data is stored locally on the device.
- No cloud sync, no account, no registration.
- No analytics and no data collection of any kind.
- Text recognition happens on-device.
- The user can export and delete everything.

**Do not overclaim.** Two things are true and must not be described otherwise:

- **Backup files are not encrypted.** The JSON backup is plain text with Base64 images. Never say
  or imply backups are encrypted.
- **The tax ID is stored unencrypted**, in `@AppStorage` on iOS.

Do not describe a privacy property the platform does not actually have. If Android ships with a
different posture, this file changes first, then the copy.

---

## 5. Accessibility claims

The website targets WCAG 2.1 AA. The iOS app supports VoiceOver and Dynamic Type. A full
contrast and touch-target audit is **deferred and not done** — do not claim the app is audited or
certified until it is.

---

*Governs: the iOS app repo, the website repo, and any future Android repo. Change this file
rather than a copy in one of them.*
