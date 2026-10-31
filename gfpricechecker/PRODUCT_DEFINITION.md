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

**Displayed as:** "Package Price Differential" (`differential`) and "Size-Adjusted Differential"
(`normalizedDifferential`), each with an "Extended" form for the purchase. The size-adjusted
figure is a dollar amount per package, not a rate, so it is **never** labelled a unit price
differential; that name belongs to the rate alone.

**Rates never scale with quantity.** Buying three loaves does not change the price per gram, so
`unitPriceDifferential` has no extended form. Everything that totals a purchase must use an
`extended*` figure; everything that shows a rate must not.

`normalizedDifferential` and the unit prices return **nothing** when size data is incomplete for
either side. Do not substitute zero — a missing comparison is not a zero-dollar comparison.

**Round the final sum, not the rows.** A total — for a receipt, a category, a month, a year or an
export — adds up the unrounded `extended*` figures and rounds to the cent once, when it is shown.
Rounding each row first and adding those up can be off by up to half a cent per row. The
size-adjusted figures divide, so this is the usual case for them, not an edge case. The package
differentials are differences of whole-cent prices, so they come out the same either way.

**Rounding is half-up**, the common rule: a figure exactly on a half cent rounds away from zero,
so $0.465 shows as $0.47 and -$0.125 as -$0.13. This applies to every figure shown and every
figure the CSV writes. It is `BigDecimal`'s `RoundingMode.HALF_UP`. It is not what iOS's currency
and number formatters do by default, which is half-even, so iOS has to ask for it; nor is it
`Math.round`, which rounds -0.125 to -0.12.

Worked example, from the iOS source:

> GF $7.99 / 400 g vs Regular $3.99 / 675 g
> → ($0.01997 − $0.00591) × 400 = **$5.63**

### 4. Unit price convention

`$/100 g` for gram weights; `$/unit` for everything else. Labelled `100g` or the raw unit.
This is a display convention and must be identical across platforms, or the same shopping trip
produces two different numbers.

The CSV writes unit prices at this scale to 4 decimal places — `$/mL` rounds to nothing at 2 —
and states the basis (`100g`, `kg`, `mL`, …) in its own column, since it varies row to row. With
no size unit there is no basis, and the unit price fields are empty.

### 5. Privacy posture

Local storage only. No cloud sync, no account, no analytics, no data collection. On-device text
recognition. This is a **constraint**, not a description — an Android implementation that adds
telemetry is a different product, and the website's privacy claims would become false.

What is *not* true, and must not be claimed: backups are unencrypted plain JSON, and the tax ID is
stored unencrypted. Matching iOS here means matching the honesty, not the weakness — an Android
build is free to do better, and should say so rather than inheriting the caveat silently.

### 6. Export formats

Both exports are what the user actually hands to an accountant, so the CSV column set is part of
the product, not an implementation detail.

CSV columns, in order:

```
Purchase Date, Quantity, GF Product, GF Price, GF Size, GF Size (num),
Non-GF Product, Non-GF Price, Non-GF Size, Non-GF Size (num), Size Unit,
Unit Price Basis, GF Unit Price, Regular Unit Price, Package Price Differential,
Size-Adjusted Differential, Extended Package Differential,
Extended Size-Adjusted Differential, Category, Store, Receipt Verified, Notes
```

The CSV is for spreadsheets, not people, so its numbers and dates are machine formats whatever the
device locale: `.` as the decimal separator, no grouping, Latin digits, and `YYYY-MM-DD` dates
(`en_US_POSIX` on iOS; `Locale.ROOT` is the JVM equivalent). Money has 2 places and unit prices 4.
Each row is rounded on its own, so a size-adjusted column can sum to a cent or so away from the
total the app shows for the same rows. That is expected (§3: the total rounds the final sum), and
the app's figure is the total.
A field holding a comma, a quote or any line break — Windows `\r\n` included — is quoted, with
quotes doubled. The file is UTF-8 with a BOM, for Excel.

The PDF is a branded multi-page report covering a date range, with receipt images included.

### 7. Backup interchange

The JSON backup is the only route a user has between devices, and **it is the migration path if
someone moves from iOS to Android.** If Android reads or writes this format, the two must agree
field for field.

> Note carried over from the iOS source, because it has already caused a data-loss bug once:
> every stored property is hand-duplicated in the backup model and in the mapping code on both
> sides of it. Adding a field without updating all three compiles fine, ships fine, and silently
> drops the field on restore.

### 8. Test vectors

[`vectors/`](vectors/) holds plain inputs and expected outputs for everything above that can be
checked mechanically — the arithmetic and its rounding, reporting totals and date ranges, the CSV
character for character, receipt parsing, price ranking, validation, and reading backups. **The
vectors are normative: a conforming implementation must pass all of them.** Where this document
and a vector disagree, raise it — one of them is wrong — but until it is settled the vector is
what an implementation is tested against. [`vectors/README.md`](vectors/README.md) has the
conventions for reading them.

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
