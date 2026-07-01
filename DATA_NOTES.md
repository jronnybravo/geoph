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
- **`adm2` is administrative, not geographic.** A city's `adm2` is the province
  that *administers* it. **Highly Urbanized Cities (HUC)** and **Independent
  Component Cities (ICC)** are independent of any province, so their `adm2` (and
  their barangays' `adm2`) is **blank** — same treatment as NCR. Use `geo_par_id`
  (below) for the province a city physically sits in. See `city_class` for the
  classification driving this.
  - **City of Isabela** is the one exception: it is an ordinary **component city**
    of Basilan (its residents vote for Basilan provincial officials), so `adm2` =
    Basilan. But it opted out of ARMM/BARMM and is serviced by **Region IX**, so
    `adm1` = Region IX every year while Basilan itself sits in BARMM — the one
    deliberate case where a unit's `adm2` belongs to a different region than its
    `adm1`. Its `PH09…` code does not prefix-nest under Basilan; ancestry is still
    correct.
- **`city_class`** (cities layer): `HUC` (33), `ICC` (5: Dagupan, Naga [Camarines
  Sur], Ormoc, Santiago, Cotabato), `CC` (component cities, 111), or blank
  (municipalities) — 149 cities total. Invariant: `adm2` is blank ⟺ `city_class`
  is `HUC` or `ICC`. Naga (Camarines Sur) is treated as an **ICC** for 2022–2024;
  its HUC conversion (Proclamation 1267, May 2026) postdates every published year
  and needs a plebiscite to take effect.
- **`geo_par_id`** (all four layers): the *geographic* parent's pcode — barangay →
  its city, city → the **province it physically sits in** (recovered from the
  old-format PSGC code, which embeds the province even where the new code and the
  administrative `adm2` do not), province → region, region → `PH`. For HUCs/ICCs
  this restores the province link that administrative `adm2` drops (e.g. City of
  Cebu → Cebu); NCR cities point to NCR (no province exists). Named `geo_par_id`,
  not `geo_parent_id`, because DBF field names cap at 10 characters. Both
  `city_class` and `geo_par_id` are deliberate additions **outside** the OCHA
  COD-AB schema.
- `name1..3` and `lang1..3` are empty — they are empty in the source COD-AB too.
- `adm*_ref_n` is set equal to `adm*_name` (the source's reference name differs
  from the name in only ~1% of rows).
- `valid_on` is the snapshot year date (e.g. 20241231); `valid_to` is open.

## Region naming (`adm1_name`) and ordering (`adm1_seq`)

Region names follow the Wikipedia
[Regions of the Philippines](https://en.wikipedia.org/wiki/Regions_of_the_Philippines)
table: **`<Name> (<Designation>)`**, e.g. `Ilocos Region (Region I)`,
`Calabarzon (Region IV-A)`, `Caraga (Region XIII)`. The four regions with no
Roman-numeral designation use their acronym as the designation:
`National Capital Region (NCR)`, `Cordillera Administrative Region (CAR)`,
`Negros Island Region (NIR)`, `Bangsamoro Autonomous Region in Muslim Mindanao
(BARMM)`.

- **Mimaropa** is `Southwestern Tagalog Region (Mimaropa)`. It was formerly
  designated **Region IV-B** (2002) until it was renamed and the Roman numeral
  dropped in 2016; we use the current, un-numbered designation.
- Casing follows the source table: `Calabarzon`, `Soccsksargen`, `Mimaropa` are
  title-case (not all-caps acronyms).

Two numbering systems must not be confused: the **PSGC region code** (first two
digits of `adm1_pcode`) is *not* the Roman-numeral designation — Caraga is code
16 but "Region XIII", Mimaropa is code 17 but was "Region IV-B". The code lives
in `adm1_pcode`; the designation lives in `adm1_name`.

**`adm1_seq`** (regions layer only, integer): a sort key that reproduces the
Wikipedia table's order, which is **geographic** — island group Luzon → Visayas →
Mindanao, roughly north-to-south — and matches *neither* the pcode nor the region
number. `ORDER BY adm1_seq` yields NCR, CAR, Region I–IV-A, Mimaropa, V, VI, NIR,
VII, VIII, IX–XII, Caraga, BARMM. Values are 1–18 for 2024 and 1–17 for
2022/2023 (which predate NIR); the relative order is identical. This field is a
deliberate addition **outside** the OCHA COD-AB schema and is not present on the
province/city/barangay layers.

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
