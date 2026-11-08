# Check Studio

A single-file, browser-based check generator. Open one HTML file, type into a
form, and get a print-ready 8.5 × 3.67 in business check — front and back,
with a real E-13B MICR line and a stock design that looks like paper.

No build step. No backend. No npm install. No account.

<img width="1917" height="866" alt="Screenshot 2026-11-08 081713" src="https://github.com/user-attachments/assets/c0652ee0-1948-4d36-9c29-00971060526a" />

<img width="1917" height="867" alt="Screenshot 2026-11-08 081751" src="https://github.com/user-attachments/assets/8611e78e-762f-441e-a94a-3ef913f948bd" />



---

## What it is

Check Studio renders bank check stock in the browser and lets you fill it in
live. It exists for the cases where you need a check that *looks* right:
mockups, invoices that ship with a payment stub, training material, a prop for
a video, a sample in a design review, or a printable draft you're going to
sign and deposit by phone.

Every field is editable. Every edit repaints immediately. Nothing is uploaded
anywhere — the whole thing is client-side and the state lives in memory.

## Features

**Both faces, both real.** The front carries the payer block, date, payee,
numeric and written amount, memo, and signature rule over a combed guilloche
frame. The back is reproduced from actual stock artwork: rotated 90° as
printed, `ENDORSE HERE`, three endorsement rules, a mobile-deposit checkbox,
the `DO NOT WRITE/ SIGN/ STAMP BELOW THIS LINE` boundary, and a tinted
security panel with an `ORIGINAL DOCUMENT` watermark, laid lines, and a
security-feature list.

**Five paper stocks.** Mint green, butter yellow, sky blue, lilac, bone white.
Each is a variable block — paper tint, panel tint, frame ink, border, rule
colour, text colour. One block drives both faces, so the back always matches
the front.

**A real MICR line.** E-13B encoding face loaded via `@font-face`, with a
Unicode-symbol fallback. Transit-routing-transit-account-number, formatted and
zero-padded. The check number also prints top-right in MICR digits, where the
stock pre-prints it.

**Auto-written amounts.** Type `1234.56` and the words line fills itself:
`ONE THOUSAND TWO HUNDRED THIRTY-FOUR AND 56/100`. Uncheck the toggle to
override by hand.

**Check numbering that behaves.** Manual entry, a `Next #` button, and an
auto-increment that fires after export so a run of checks doesn't collide.
Press `N` anywhere outside a field to advance.

**Real exports.**
- PNG at 300 DPI (2550 × 1101 px), front only or front and back
- PDF, 8.5 × 3.67 in, one page per face
- Print stylesheet with `@page` at exact size, zero margin, one check per page

**Layered security detail.** Crossing hairline security screen, rosette
guilloche, microprint band, deterministic barcode derived from the routing and
account, sequential number stamp. All toggleable.

## Quick start
```
git clone https://github.com/<you>/check-studio.git
cd check-studio
open index.html
```
Or skip the clone: download `index.html` and double-click it.

That's the whole install.

## Usage

1. Fill the left panel. The preview updates as you type.
2. Choose a stock from **Check stock**.
3. Set routing and account, then the fractional routing for the top-right
   corner. The classic derivation from routing `AB-CDEF/GHI` is `AB-CDEF/GHI`
   — keep the prefix, numerator from digits 3–6, denominator from 7–9.
4. Export with **PNG** or **PDF**, or **Print** for a physical sheet.

### Print settings

Set scale to **100%** or **Actual size**. "Fit to page" will shrink the check
and misalign the MICR line. Use 24 lb bond or heavier if you're printing to
deposit.

## Customization

All defaults live in one object at the top of the `<script>` block. Change
them once and the file becomes your book:

```js
const DEFAULTS = {
  payerName: "NORTHGATE TRADING LLC",
  payerAddr: "1420 Harbor View Dr, Suite 300\nSeattle, WA 98101",
  bankName:  "Cascade National Bank",
  bankAddr:  "P.O. Box 8891 · Seattle, WA 98124",
  routing:   "125008547",
  account:   "000123456789",
  frac:      "12-3456/789",
  checkNo:   "1025",
  // ...
};
```

## Adding a paper stock
Copy any .tpl-* block, rename it, and set all seven variables:
```.tpl-slate{
  --paper:#dfe3e8;      /* sheet tint, both faces */
  --panel:#c3c9d1;      /* darker tint for the back panel */
  --edge:#9aa2ad;       /* front border */
  --frame:#2b3138;      /* front frame ink */
  --back-edge:#4a545f;  /* back border */
  --line:#242a31;       /* rules */
  --ink:#191d22;        /* text */
  --ros:rgba(40,50,60,.10);
  --wm:rgba(40,50,60,.10);
  --wm-strong:rgba(255,255,255,.94);
  --sig-ink:#171b20;
}
```

Then add an <option value="slate"> to #f-template. Nothing else.

## Making it fully offline
Two CDN scripts (html2canvas, jsPDF) and the MICR font load remotely. To cut
the network entirely:

Download both libraries and inline them, or vendor them locally.
Base64 an E-13B TTF and replace the src: list in the @font-face rule
with url("data:font/ttf;base64,...").
Drop the Google Fonts @import.

## Layout notes
The check is a fixed 816 × 352 px canvas — 8.5 × 3.67 in at 96 DPI — so
exports are pixel-stable and nothing reflows. The back is authored upright in
a 352 × 816 portrait box and rotated 90° into the sheet, matching how the
stock is actually printed. Coordinates for back elements are in that rotated
space: larger top moves toward the left edge of the finished check, larger
left moves toward the bottom.

Capture scale is 300/96 = 3.125. Exported rasters land at exactly 300 DPI.

Browser support
Current Chrome, Firefox, and Edge. The back relies on a CSS transform, and
exports rely on html2canvas, which wants a reasonably modern engine. Safari is
untested — it generally works but text metrics on the rotated layer can drift.

## Limitations
It produces a check image, not a negotiable instrument. No bank will
treat a printout as more than a draft.
No MICR check-digit validation — the routing number is printed as entered,
padded to nine digits.
No fractional-routing derivation; you supply the value.
The barcode is decorative, not a real Code 128 or Interleaved 2 of 5.
Signature is typed text in a script face. No signature pad.
Amount words cap out around the trillions and don't handle negative or
non-numeric input beyond falling back to zero.
