# geoph — Philippine administrative boundaries (synthesized, COD-AB format)

> ⚠️ **Unofficial, synthesized dataset — not an official PSA/NAMRIA product.**
> As of June 2026 the Philippine Statistics Authority (PSA) has **not published
> official shapefiles** for the current PSGC. This dataset is *synthesized*: it
> combines the latest openly-available boundary geometry with the official PSGC
> attribute roster, reconciles the two, and reconstructs each year using a
> temporal model of the administrative changes. **Do not treat it as
> authoritative for legal or official purposes.** Boundaries are generalized
> (see *Resolution*).

Philippine administrative boundaries — regions down to barangays — as shapefiles
in the **OCHA COD-AB schema**, with **one folder per year** so you can work with
the boundaries as they stood in that year.

## Layout

```
boundaries/
  2022/   2023/   2024/
    regions.shp                 (adm1)
    provinces.shp               (adm2)
    cities_municipalities.shp   (adm3 — cities + municipalities)
    barangays.shp               (adm4)
```

All layers EPSG:4326 (WGS84), `MultiPolygon`, UTF-8.

## Schema (COD-AB)

Each file follows the OCHA **Common Operational Dataset – Administrative
Boundaries** layout: every feature carries its **full ancestry inline** plus
metadata. For the barangay (adm4) file:

| Group | Fields |
|---|---|
| Unit | `adm4_name`, `adm4_pcode`, `adm4_ref_n`, `adm4_name1..3` (alt names, empty) |
| City/Mun | `adm3_name`, `adm3_pcode`, `adm3_name1..3` |
| Province | `adm2_name`, `adm2_pcode`, … |
| Region | `adm1_name`, `adm1_pcode`, … |
| Country | `adm0_name` (Philippines), `adm0_pcode` (PH) |
| Meta | `valid_on`, `valid_to`, `area_sqkm`, `version`, `lang`, `lang1..3`, `center_lat`, `center_lon` |

Higher-level files (adm1–adm3) carry the same layout truncated to their level.
Field names and order match the official COD-AB Philippines files, so this is a
near drop-in for tooling built around that dataset.

## pcodes

`pcode`s use the COD-AB convention (`PH` + the PSGC code, trimmed and nested by
level), and **each year uses the code that was official at the time.** PSA
renumbered the PSGC in 2021 — the province field grew from 2 to 3 digits — and
OCHA COD-AB adopted the new 10-digit codes around 2023. So:

| Level | 2022 & 2023 (`PH`+legacy, 2-digit province) | 2024 (`PH`+new, 3-digit province) |
|---|---|---|
| region | `PH13` | `PH13` |
| province | `PH1376` | `PH13803` |
| city | `PH137602` | `PH1380300` |
| barangay | `PH137602003` | `PH1381500029` |

(That example is one EMBO barangay: in 2022/2023 it carries its old **Makati**
code, in 2024 its new **Taguig** code — matching reality both years.) Because the
format changes, a unit's pcode differs between the 2023 and 2024 files; bridge
them with the official PSGC old↔new correspondence. The ~48 barangays abolished
in the 2023 Bacoor merger / contested cases have no clean old code and carry a
documented `PH`+as-stored fallback in the 2022 file.

## Differences from official COD-AB

- **Per-year folders** — COD-AB ships a single current snapshot; we reconstruct
  2022, 2023 and 2024 from a temporal model.
- **NCR has no province tier** — PSGC doesn't treat NCR's legislative districts
  as provinces, so `adm2` is **blank** for NCR units (COD-AB fills 4 districts
  there; that's why we have 83 provinces vs COD-AB's 88).
- **Manila sub-municipalities folded** — PSGC's 14 Manila sub-municipalities are
  not a COD-AB tier; their barangays roll up to `adm3` = City of Manila (matching
  COD-AB), and the sub-municipalities are not separate features.
- `valid_on` is the snapshot year date; `valid_to` open. `name1..3`/`lang1..3`
  are empty (they are empty in the source too).

## What changed each year

- **2022** — baseline. Maguindanao already split into del Norte / del Sur; the
  BARMM Special Geographic Area present. Bacoor still has its 44 pre-merger
  barangays; Carmona still a municipality; Sulu in BARMM; EMBO barangays under
  Makati; no Negros Island Region.
- **2023** — Bacoor barangay merger (44 → 18, City Ordinance 275-2023, eff.
  2023-07-29); Carmona becomes a city (RA 11938, eff. 2023-09-30).
- **2024** — EMBO barangays transferred Makati → Taguig (eff. ~2024-06-01);
  **Negros Island Region** established (RA 12000, eff. 2024-06-11); Sulu moved
  from BARMM to Region IX (Supreme Court ruling, eff. 2024-09-18). Current
  configuration — no boundary changes since.

## Counts

| COD-AB level | 2022 | 2023 | 2024 |
|---|--:|--:|--:|
| adm1 — region | 17 | 17 | 18 |
| adm2 — province (incl. Special Geographic Area) | 83 | 83 | 83 |
| adm3 — city / municipality | 1,642 | 1,642 | 1,642 |
| adm4 — barangay | 42,025 | 41,999 | 41,999 |

## How it was made

A reconciliation of two open sources, projected through a temporal model:

1. **Geometry** — PSA/NAMRIA administrative boundaries via the UN OCHA
   Humanitarian Data Exchange (COD-AB Philippines).
2. **Roster / codes / names** — the official PSGC 1Q-2026 publication (PSA), plus
   the legacy correspondence codes.

The barangay layer was reconciled unit-by-unit to the official PSGC roster
(**99.97%** exact code match for the current year); the higher levels match
official counts and codes exactly. Each year is reconstructed from validity
intervals and parent edges. Integrity is clean for every year: 0 invalid
geometries, 0 duplicate pcodes, 0 orphaned ancestry.

## Resolution

Geometry is generalized for web/analytical use (barangays ~11 m; parent layers
coarser). Not survey-grade. The barangay layer intentionally **excludes
non-barangay land parcels** (forest lands, national parks, watersheds, etc.) that
the source carries but PSGC does not classify as barangays; those areas remain
within the enclosing city/municipality polygons.

## Known limitations

See [`DATA_NOTES.md`](DATA_NOTES.md). In brief: pcodes are era-specific (legacy
old-format for 2022/2023, new for 2024), so they aren't a cross-year join key;
2024 has 2 codeless barangays (a Caloocan split we can't subdivide, a contested
Calaca barangay); the ~48 abolished-2023 barangays use a `PH`+as-stored fallback
pcode; only the current (2024) codes are reconciled against an official roster.

## Sources

- **Philippine Statistics Authority (PSA)** — PSGC classification & 1Q-2026
  publication. <https://psa.gov.ph/classification/psgc>
- **NAMRIA / PSA boundaries via UN OCHA Humanitarian Data Exchange** — COD-AB
  Philippines. <https://data.humdata.org/dataset/cod-ab-phl>
- **PSGC quarterly publication mirror** —
  <https://github.com/bendlikeabamboo/barangay-data-repository>
- **Change references:** RA 12000 (Negros Island Region); RA 11938 (Carmona
  cityhood); PSA "Second Quarter 2024 PSGC Updates" (NIR); PSA "Third Quarter
  2023 PSGC Updates" (Bacoor merger, Carmona, EMBO transfer); Bacoor City
  Ordinance 275-2023; Supreme Court ruling excluding Sulu from BARMM (2024).

## License

Data released under **CC-BY-4.0**. Please credit PSA, NAMRIA, OCHA/HDX, and this
project. Provided **as-is, without warranty**; see the disclaimer above.
