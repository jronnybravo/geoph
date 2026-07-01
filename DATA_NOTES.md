# Data notes & known limitations

This dataset reconciles open boundary geometry to the official PSGC roster and
reconstructs each year from a temporal model, in the OCHA COD-AB schema.
Integrity is clean for every year: **0 invalid geometries, 0 duplicate pcodes,
0 orphaned ancestry.** The items below are real-world data quirks, documented so
they aren't mistaken for errors.

## pcodes (read this first)

`adm*_pcode` = **`PH` + the PSGC code**, trimmed and nested by level (COD-AB
convention). **Each year uses the code that was official at the time.** PSA
renumbered the PSGC in 2021 (province field 2→3 digits) and OCHA COD-AB adopted
the new 10-digit codes around 2023, so:

- **2022 & 2023** → `PH` + the **legacy old-format** code (2-digit province):
  `PH13` → `PH1376` → `PH137602` → `PH137602003`.
- **2024** → `PH` + the **new 10-digit** code (3-digit province):
  `PH13` → `PH13803` → `PH1380300` → `PH1381500029`.

Consequences:

- A unit's pcode **differs between the 2023 and 2024 files** (format change), so
  pcode is **not** a cross-year join key. Use the official PSGC old↔new
  correspondence to bridge years.
- The ~48 barangays abolished in the 2023 Bacoor merger (and the contested
  Caloocan/Calaca cases) have no clean old code, so their `adm4_pcode` is a
  `PH`+as-stored fallback. It may not strictly prefix-nest under its parent's
  pcode, but the ancestry columns are correct (0 orphaned ancestry in all years).

## Structural mapping vs. official COD-AB

- **NCR `adm2` is blank.** PSGC does not treat NCR's legislative districts as
  provinces, and we did not synthesize them, so NCR units carry `adm1` = NCR and
  an empty `adm2`. Official COD-AB places 4 NCR districts at adm2 — hence 83
  provinces here vs. COD-AB's 88.
- **Manila sub-municipalities folded.** PSGC's 14 Manila sub-municipalities have
  no COD-AB tier; their barangays roll up to `adm3` = City of Manila, and the
  sub-municipalities are not emitted as features (matching COD-AB's adm3 = 1,642).
- **Independent cities carry their mother province at `adm2`.** Bacolod and
  Isabela are province-independent in the PSGC (each has its own province-level
  code: Bacolod `…302…`, Isabela `…97…`/`…901…`), but we fill `adm2` with the
  province they sit in so they are not region-level orphans:
  - **City of Bacolod** → `adm2` = Negros Occidental, region per year (Region VI
    in 2022/2023; **NIR** in 2024). In 2022/2023 its `PH0645…` code nests under
    Negros Occidental; in 2024 its independent `…302…` code does not strictly
    prefix-nest, but the ancestry columns are correct.
  - **City of Isabela** → `adm2` = Basilan, but `adm1` = **Region IX** every year.
    Isabela opted out of ARMM/BARMM, so it is governed under Region IX while its
    province Basilan sits in BARMM. This is the one deliberate case where a unit's
    `adm2` belongs to a different region than its `adm1`; its `PH09…` code does not
    prefix-nest under Basilan. Faithful to the real split — not an error.
- `name1..3` and `lang1..3` are empty — they are empty in the source COD-AB too.
- `adm*_ref_n` is set equal to `adm*_name` (the source's reference name differs
  from the name in only ~1% of rows).
- `valid_on` is the snapshot year date (e.g. 20241231); `valid_to` is open.

## Region naming (`adm1_name`)

Region names are standardized to **`REGION <Roman> (Descriptive Name)`** for the
14 regions that carry an official Roman-numeral designation, e.g.
`REGION I (Ilocos Region)`, `REGION IV-A (CALABARZON)`, `REGION XIII (Caraga)`.
MIMAROPA is written `REGION IV-B (MIMAROPA)` — its designation from 2002 until it
was renamed the "Southwestern Tagalog Region (MIMAROPA)" in 2016.

Four regions have a PSGC **region code** but **no Roman-numeral designation**, so
they are kept name-only (no fabricated number):

- `National Capital Region (NCR)` — code 13
- `Cordillera Administrative Region (CAR)` — code 14
- `Bangsamoro Autonomous Region In Muslim Mindanao (BARMM)` — code 15 (2022/2023),
  19 (2024)
- `Negros Island Region (NIR)` — code 18 (2024 only)

Note the two numbering systems differ: the PSGC region code (first two digits of
`adm1_pcode`) is **not** the Roman-numeral designation — Caraga is code 16 but
"Region XIII", and MIMAROPA is code 17 but "Region IV-B". The code lives in
`adm1_pcode`; the designation lives in `adm1_name`.

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
