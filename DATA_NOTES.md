# Data notes & known limitations

This dataset reconciles open boundary geometry to the official PSGC roster and
reconstructs each year from a temporal model, in the OCHA COD-AB schema.
Integrity is clean for every year: **0 invalid geometries, 0 duplicate pcodes,
0 orphaned ancestry.** The items below are real-world data quirks, documented so
they aren't mistaken for errors.

## pcodes (read this first)

`adm*_pcode` = **`PH` + the new (2021-revision) PSGC code**, trimmed and nested
by level — the COD-AB convention. A pcode is a **stable identifier**: it stays
constant for a unit across years even when the unit's parent changes. What
changes per year is geometry, names, ancestry, and which units exist.

- The ~48 barangays abolished in the 2023 Bacoor merger (and the contested
  Caloocan/Calaca cases) have **no new PSGC code**, so their `adm4_pcode` falls
  back to `PH` + their legacy code. These won't strictly prefix-nest under their
  parent's pcode, but their ancestry columns are correct.

## Structural mapping vs. official COD-AB

- **NCR `adm2` is blank.** PSGC does not treat NCR's legislative districts as
  provinces, and we did not synthesize them, so NCR units carry `adm1` = NCR and
  an empty `adm2`. Official COD-AB places 4 NCR districts at adm2 — hence 83
  provinces here vs. COD-AB's 88.
- **Manila sub-municipalities folded.** PSGC's 14 Manila sub-municipalities have
  no COD-AB tier; their barangays roll up to `adm3` = City of Manila, and the
  sub-municipalities are not emitted as features (matching COD-AB's adm3 = 1,642).
- `name1..3` and `lang1..3` are empty — they are empty in the source COD-AB too.
- `adm*_ref_n` is set equal to `adm*_name` (the source's reference name differs
  from the name in only ~1% of rows).
- `valid_on` is the snapshot year date (e.g. 20241231); `valid_to` is open.

## Barangay-level edge cases (2024 / current)

- **Barangay 176, City of Caloocan** — officially split into 176-A/B/C/D. The
  source geometry only has the un-split parent polygon, and the four child
  boundaries can't be invented, so this appears as the single parent (pcode falls
  back to `PH`+legacy). Its four official children are absent.
- **San Rafael, City of Calaca** — a contested/transferred barangay (PSA
  footnotes it under G.R. No. 234228). Present in geometry; `PH`+legacy pcode.

## Count vs. the official roster

The 2024 barangay layer has 41,999 vs. the official roster's 42,010 — a gap of 11,
from the cases above (the un-subdividable Caloocan split alone is −3), a couple of
source-duplicate polygons that collapsed to a single coded unit, and a few
barangays the source geometry simply does not carry.

## Excluded: non-barangay land parcels

The source boundary file includes ~21 parcels that are **not** PSGC barangays —
forest lands, national parks (e.g. Mount Apo), watersheds, unclaimed/timber
areas, plus Manila North Cemetery and Tutuban Mall, typically named
"… under Jurisdiction of …". These are **excluded** from the barangay layer.
Their land remains inside the enclosing city/municipality polygon (verified by
containment), so no coverage holes are introduced at the city level.

## Reconciliation scope

Only the **current (2024)** year was reconciled against an official PSGC
publication (1Q-2026), at 99.97% exact barangay code match. The 2022 and 2023
years are reconstructed from the same units via the temporal model — correct for
the reorganizations we model (Bacoor, Carmona, EMBO, NIR, Sulu, Maguindanao), but
not independently reconciled against a year-specific PSGC publication.

## Not included

- Years before 2022, or finer-than-annual snapshots. The model supports any date;
  only these year-end snapshots are published.
- NCR districts and legislative/congressional districts (not synthesized).
