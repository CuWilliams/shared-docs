# GFPriceChecker — Platform-Independent Product Definition

What GFPriceChecker *is*, independent of how any one platform builds it. This exists so a second
implementation — Android, most likely — matches the iOS app where it matters and is free to differ
where it doesn't.

**Status:** derived from the iOS app as it stands (TestFlight beta, 1.0 in preparation). It is a
description of shipped behaviour, not a specification written ahead of one. Where iOS changes, this
changes with it.

---

## Must match

These are the things that make two implementations the same product. A difference here is a bug.

### 1. The data model

A **product pair** is one gluten-free product compared against one regular equivalent.

| Field | Type | Notes |
|---|---|---|
| `gfProductName` / `nonGFProductName` | string | |
| `gfPrice` / `nonGFPrice` | **decimal** | never a float — see below |
| `gfSize` / `nonGFSize` | string | as printed on the tag |
| `gfSizeValue` / `nonGFSizeValue` | decimal, optional | parsed numeric size |
| `sizeUnit` | enum | g, kg, mL, L, oz, lb, fl oz, count |
| `quantity` | integer, default 1 | how many were bought |
| `category`, `notes` | string, optional | |
| `createdDate` | date | |
| `gfImageUUID` / `nonGFImageUUID` | string, optional | |
| `gfOCRConfidence` / `nonGFOCRConfidence` | float 0.0–1.0 | extraction reliability |
| `receipt` | relationship, optional | one receipt verifies many pairs |

A **receipt** has `purchaseDate`, `storeName`, `totalAmount`, `receiptImageUUID`,
`extractedOCRText`, and the inverse relationship to its product pairs.

**Reporting date** is `receipt.purchaseDate` when a receipt is linked, otherwise `createdDate`.

### 2. Currency is decimal, never floating point

Prices are money in a document a user may rely on at tax time. Use the platform's decimal type —
`Decimal` on Swift, `BigDecimal` on Kotlin/JVM. Never `Double`, `Float`, or `Number`.

Display as Canadian currency through the platform's localized formatter. Never build a currency
string by interpolation.

### 3. The arithmetic

Four figures, and the distinction between them is the single easiest thing to get wrong:

```
differential                    = gfPrice - nonGFPrice            # per unit
unitPriceDifferential           = gfUnitPrice - regularUnitPrice  # a RATE, per gram/oz/…
normalizedDifferential          = (gfUnitPrice - regularUnitPrice) × gfSizeValue   # per unit
extendedDifferential            = differential × quantity
extendedNormalizedDifferential  = normalizedDifferential × quantity
```

where `gfUnitPrice = gfPrice / gfSizeValue` and `regularUnitPrice = nonGFPrice / nonGFSizeValue`.

**Rates never scale with quantity.** Buying three loaves does not change the price per gram, so
`unitPriceDifferential` has no extended form. Everything that totals a purchase must use an
`extended*` figure; everything that shows a rate must not.

`normalizedDifferential` and the unit prices return **nothing** when size data is incomplete for
either side. Do not substitute zero — a missing comparison is not a zero-dollar comparison.

Worked example, from the iOS source:

> GF $7.99 / 400 g vs Regular $3.99 / 675 g
> → ($0.01997 − $0.00591) × 400 = **$5.63**

### 4. Unit price convention

`$/100 g` for gram weights; `$/unit` for everything else. Labelled `100g` or the raw unit.
This is a display convention and must be identical across platforms, or the same shopping trip
produces two different numbers.

### 5. Privacy posture

Local storage only. No cloud sync, no account, no analytics, no data collection. This is a
**constraint**, not a description — an Android implementation that adds telemetry is a different
product, and the website's privacy claims would become false.

### 6. Export formats

Both exports are what the user actually hands to an accountant, so the CSV column set is part of
the product, not an implementation detail.

CSV columns, in order:

```
Purchase Date, Quantity, GF Product, GF Price, GF Size, GF Size (num),
Non-GF Product, Non-GF Price, Non-GF Size, Non-GF Size (num), Size Unit,
GF Unit Price, Regular Unit Price, Package Price Differential,
Unit Price Differential, Extended Package Differential,
Extended Unit Price Differential, Category, Store, Receipt Verified, Notes
```

The PDF is a branded multi-page report covering a date range, with receipt images included.

### 7. Backup interchange

The JSON backup is the only route a user has between devices, and **it is the migration path if
someone moves from iOS to Android.** If Android reads or writes this format, the two must agree
field for field.

> Note carried over from the iOS source, because it has already caused a data-loss bug once:
> every stored property is hand-duplicated in the backup model and in the mapping code on both
> sides of it. Adding a field without updating all three compiles fine, ships fine, and silently
> drops the field on restore.

---

## Free to differ

Platform conventions should win over matching iOS for its own sake:

- **UI framework, navigation and layout.** Follow the platform. Do not port SwiftUI's shape to
  Android.
- **OCR engine.** iOS uses Vision. ML Kit is the Android equivalent. What must match is the
  *behaviour*: confidence thresholds around 0.7 for product names and 0.6 for prices, a manual
  override whenever confidence is low, and ranking price candidates by font size and position on
  the tag rather than taking the first match.
- **Document scanning.** VisionKit on iOS; whatever is idiomatic on Android.
- **Persistence.** SwiftData on iOS; Room is the obvious Android choice. The schema above is what
  matters, not the framework.
- **Architecture.** iOS is MVVM. Any well-separated architecture is fine.
- **Image storage details.** iOS stores JPEG at 0.7–0.8 quality with 200px thumbnails, under UUID
  filenames. Sensible, but not binding.

---

## Known behaviour worth copying

Learned on iOS the hard way, and cheaper to inherit than rediscover:

- **Receipt parsing is tuned for Canadian grocery chains** — Loblaws, Sobeys, Metro, Costco —
  including Costco-style layouts where the price is on the line after the product name, and
  two-column receipts where a label and its amount are separated.
- **Fuzzy matching** between a product name and a receipt line uses Levenshtein and Jaccard
  distance. Exact matching is useless against real receipt abbreviations.
- **Guide the user to photograph the shelf price tag**, not the product. The tag holds the price
  and the size; the package usually holds neither in readable form.
- **The app is useless empty.** Whatever seeds a demo or a review build on iOS, Android will need
  an equivalent.

---

*See [CONTENT_CONVENTIONS.md](CONTENT_CONVENTIONS.md) for how to talk about any of this. The tax
compliance rule in particular applies to every platform.*
