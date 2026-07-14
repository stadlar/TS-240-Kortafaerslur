# TS 240 receipt schematron — adaptations from Peppol BIS Billing 3.0

This document records every way the TS 240 ("Auknar kortafærslur" / augmented card
transactions, Staðlaráð Íslands) validation artifacts deviate from the base Peppol
BIS Billing 3.0 / EN 16931 schematron they were derived from. It is the companion
to the rule work delivered under **AMZ-8659**.

## Provenance

The two TS 240 validation artifacts were copied from the Peppol BIS Billing 3.0
schematron and then adapted:

| TS 240 artifact | Copied from (Peppol BIS Billing 3.0) | Layer |
|---|---|---|
| `EN-validate-receipt.xsl` | `CEN-EN16931-UBL.xsl` | EN 16931 core rules |
| `STADLAR-validate-receipt.xsl` | `PEPPOL-EN16931-UBL.xsl` | Peppol + Icelandic (IS-R-*) rules |

Both are compiled ISO-Schematron XSLT 2.0 stylesheets; validation runs in two
stages (EN layer, then STADLAR layer), selected for documents whose
`CustomizationID = urn:fdc:stadlar.is:kort:1.0`. The Peppol-layer file was
**renamed** `PEPPOL-…` → `STADLAR-…` because it now carries Staðlaráð rules.

A TS 240 document is otherwise a **fully conformant EN 16931 invoice / credit
note** — every EN 16931 and Peppol BIS 3.0 rule not listed below still applies
unchanged.

## Identifier changes

| Business term | Value |
|---|---|
| Specification identifier (BT-24, `cbc:CustomizationID`) | `urn:fdc:stadlar.is:kort:1.0` |
| Business process (BT-23, `cbc:ProfileID`) | `urn:fdc:stadlar.is:ts240:1.0` |

Implemented by adapting `PEPPOL-EN16931-R004` (customization), `PEPPOL-EN16931-R007`
(business process) and the `$profile` derivation in the STADLAR layer.

## Rules added (TS 240-specific) — new pattern in the STADLAR layer

| ID | Severity | Business term | Rule |
|---|---|---|---|
| TS240-R-001 | fatal | BT-3 | Document type code must be **461** |
| TS240-R-002 | fatal | BT-81 | Payment means code must be **54** (credit card) |
| TS240-R-003 | fatal | BT-115 | Amount due for payment must be **0** |
| TS240-R-004 | fatal | BT-106/109/110/112/113 | ISK document totals must have **no decimals** (numeric whole-number check) |
| TS240-R-005 | fatal | BT-125 | Attached document must be **≤ 5 MB** of base64 content |
| TS240-R-006 | fatal | BT-125-2 | Attachment file name must be **≤ 255 characters** |
| TS240-R-007 | warning | BT-25 | Foreign seller ⇒ the original amount + currency should be given in **BT-25** |
| TS240-R-008 | fatal | BT-118/151 | Seller with no VAT/tax id ⇒ every VAT category must be **O** (not subject to VAT) |
| TS240-R-009 | fatal | BT-5 | Document currency must be **ISK** (a foreign original amount goes in BT-25) |
| TS240-R-010 | fatal | BT-13 | Fintech sender reference (`cac:OrderReference/cbc:ID`) is **mandatory** (free text) |
| TS240-R-011 | warning | BT-82 | POS entry mode (`cbc:PaymentMeansCode/@name`), if given, must be one of `MANUAL, MAGSTRIPE, CHIP, CONTACTLESS, ECOMMERCE, MOTO, STORED` |

## Rules adapted (existing EN 16931 rules changed)

| Rule | Change | Reason |
|---|---|---|
| `BR-CL-01` | Added **461** to the invoice and credit-note UNTDID 1001 code lists | 461 is the TS 240 document type code; not in the stock list |
| `BR-51` (warning) | Test changed from a raw ≤ 10-character cap to counting **shown digits** (`replace(., '[^0-9]', '')` ≤ 10) | So a symbol-masked PAN (`542418******1234`) passes while a full card number is still flagged |

## Rules removed / neutralized (STADLAR layer, receipt copy only)

These Icelandic `IS-R-*` rules were removed from the TS 240 copy. The mainline BIS 3.0
schematron is untouched, so ordinary Icelandic invoices are unaffected.

| Rule | Was | Removed because |
|---|---|---|
| IS-R-001 (warning) | Icelandic seller ⇒ type 380/381 | Superseded by TS240-R-001 (type 461) |
| IS-R-003 (fatal) | Icelandic seller ⇒ street name + postal zone | TS 240 §3.1 postal address carries only the country code |
| IS-R-005 (fatal) | IS↔IS ⇒ buyer street name + postal zone | Same as IS-R-003, for the buyer |
| IS-R-008/009/010 (fatal) | EINDAGI (due-date) format rules | Due dates are not relevant to card transactions |

**Kept** from the Icelandic set: IS-R-002 / IS-R-004 (Icelandic seller / buyer must
carry a kennitala with `@schemeID = 0196`, consistent with TS 240 BT-30 / BT-47) and
IS-R-006 / IS-R-007 (giro/transfer account rules, inert because TS 240 uses card
payment).

## Rules kept from EN 16931 / Peppol that satisfy TS 240 by inheritance

No new rule was needed for these TS 240 requirements — the base rules already cover them:

- Seller/Buyer electronic address scheme (BT-34-1 / BT-49) — EAS code list, incl. `0196` (`BR-CL-25`)
- Unit of measure (BT-130) — UN/ECE Rec 20 + 21 (`BR-CL-23`)
- Attachment mime code (BT-125-1) — allowed list incl. pdf/jpeg/png (`BR-CL-24`)
- Country codes (BT-40 / BT-55) — ISO 3166 (`BR-CL-14` / `BR-CL-15`)
- Currency validity (BT-5) — ISO 4217 (`BR-CL-04/05`); TS 240 additionally **forces it to ISK** (TS240-R-009)
- Max-2-decimals for non-ISK amounts (`BR-DEC-*`)
- Document total arithmetic incl. BT-115 = BT-112 − BT-113 (`BR-CO-16`)
- VAT breakdown / category calculations (`BR-S-*`, `BR-CO-*`)

The ~35 rules the BIS 3.0 refresh added over the previously-frozen copy (German
`DE-R-*`, Danish/Swedish national rules, IGIC/IPSI VAT categories) are all
country- or feature-gated and cannot fire for an Icelandic card-transaction
receipt, so none were removed.

## Known open points (require Staðlaráð decision)

See `ts240-spec-issues.md` and the
[Staðlaráð issue tracker](https://github.com/stadlar/TS-240-Kortafaerslur/issues). Still open: the final
BT-81 payment code (48 vs 54 — issue #5); the exact card-number masking format (#10); a field for the
cardholder's name and ID (our suggestion is BT-88 + an object-identifier; FJS's review suggests
BT-82/BT-83, so the numbering needs agreeing — #10); and the ARN / acquirer reference number (#12). (VAT
category AA, the BT-34-1 scheme-code wording, the ISK no-decimals interpretation, the "same as document
level" reading, and the §6 examples are resolved in the spec-issues doc.)

## Verification

All eight reference receipts provided alongside this schematron validate through both stages with
**0 fatal / 0 warning** (Saxon-HE, XSLT 2.0). Negative cases fire exactly the expected rule ids, and an
automated regression test runs both stages over every sample to keep this guaranteed.
