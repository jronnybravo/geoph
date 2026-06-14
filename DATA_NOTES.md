# Data notes & known limitations

This dataset reconciles open boundary geometry to the official PSGC roster and
reconstructs each year from a temporal model. Integrity is clean for every year:
**0 invalid geometries, 0 duplicate codes, 0 orphaned parent links.** The items
below are real-world data quirks, documented so they aren't mistaken for errors.

## Codes per year (read this first)

`code` is the **operative** PSGC code for that year:

- **2022, 2023** → the legacy (pre-2021) code, which was the code actually in use
  before the 2021 10-digit revision was fully adopted (~2024).
- **2024** → the new 2021-revision 10-digit code.
- `code_new` is always the new 10-digit code — use it to follow one unit across
  years (its `code` may change when it was renumbered or reassigned).

**Always join the hierarchy with the `parent` field, never by truncating `code`.**
The 46 barangays abolished in the 2023 Bacoor merger were captured with their
*new-format* codes (e.g. Alima `0402103001`) in the 2022 layer, while their parent
city carries an old-format code (`0042103000`). Prefix derivation would fail for
these 46; the explicit `parent` field is correct (verified: 0 orphans).

## Barangay-level edge cases (2024 / current)

- **Barangay 176, City of Caloocan** — officially split into 176-A/B/C/D. The
  source geometry only has the un-split parent polygon, and the four child
  boundaries can't be invented, so this appears as the single parent (codeless,
  `code` blank). Its four official children are therefore absent.
- **San Rafael, City of Calaca** — a contested/transferred barangay (PSA
  footnotes it under G.R. No. 234228). Present in geometry, without an assigned
  current code.

These are the only 2 codeless units in 2024. (2022 and 2023 have 0, because the
legacy codes fill the units that lack a current one.)

## Count vs. the official roster

The 2024 barangay layer has 41,999 vs. the official roster's 42,010 — a gap of 11,
from the cases above (the un-subdividable Caloocan split alone is −3), a couple of
source-duplicate polygons that collapsed to a single coded unit (so one official
sibling is absent rather than duplicated), and a few barangays the source geometry
simply does not carry.

## Excluded: non-barangay land parcels

The source boundary file includes ~21 parcels that are **not** PSGC barangays —
forest lands, national parks (e.g. Mount Apo), watersheds, unclaimed/timber
areas, plus Manila North Cemetery and Tutuban Mall, typically named
"… under Jurisdiction of …". These are **excluded** from the barangay layer.
Their land remains inside the enclosing city/municipality polygon (verified by
containment), so no coverage holes are introduced at the city level — the
barangay layer simply does not tile those natural/special areas, which is
faithful to PSGC.

## Reconciliation scope

Only the **current (2024)** year was reconciled against an official PSGC
publication (1Q-2026), at 99.97% exact barangay code match. The 2022 and 2023
years reuse the same units' legacy codes — correct for the affected
reorganizations we model (Bacoor, Carmona, EMBO, NIR, Sulu, Maguindanao), but
not independently reconciled against a year-specific PSGC publication.

## Not included

- Years before 2022, or finer-than-annual snapshots. The underlying model
  supports any date, but only these year-end snapshots are published.
- Legislative/congressional districts (PSA does not publish their boundaries).
