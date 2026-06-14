# geoph — Philippine administrative boundaries (synthesized)

> ⚠️ **Unofficial, synthesized dataset — not an official PSA/NAMRIA product.**
> As of June 2026 the Philippine Statistics Authority (PSA) has **not published
> official shapefiles** for the current PSGC. This dataset is *synthesized*: it
> combines the latest openly-available boundary geometry with the official PSGC
> attribute roster, reconciles the two, and reconstructs the state of each year
> using a temporal model of the administrative changes. **Do not treat it as
> authoritative for legal or official purposes.** Boundaries are generalized
> (see *Resolution*).

Philippine administrative boundaries — regions down to barangays — as shapefiles,
keyed by PSGC codes, **with one folder per year** so you can work with the
boundaries as they stood in that year.

## Layout

```
boundaries/
  2022/   2023/   2024/
    regions.shp
    provinces.shp
    cities_municipalities.shp   (cities + municipalities + Manila sub-municipalities)
    barangays.shp
```

All layers EPSG:4326 (WGS84). Each feature carries:

| Field | Meaning |
|---|---|
| `code` | **Operative PSGC code for that year** — see *Codes* below |
| `code_new` | The current 2021-revision 10-digit code (stable crosswalk across years; blank for units abolished before the 2021 codes were adopted) |
| `name` | Official name |
| `level` | region / province / city / municipality / submunicipality / barangay |
| `parent` | `code` of the parent unit (always use this for hierarchy joins) |

## Codes — important

The PSGC was renumbered in 2021 (Board Resolution No. 07): the province field grew
from 2 to 3 digits (10-digit codes), phased in over a ~3-year transition completed
around **2024**. To keep each year faithful:

- **2022 & 2023** (`code`) use the **legacy (pre-2021) code** — the code actually
  operative then. Units created in the new-code era that never had a legacy code
  fall back to their new code.
- **2024** (`code`) uses the **new 2021-revision 10-digit code**.
- `code_new` always carries the new code, so you can join the same unit across
  years (e.g. an EMBO barangay reads its Makati code in 2022/2023 and its Taguig
  code in 2024, but `code_new` is constant).

Join hierarchy with the explicit `parent` field, **not** by truncating `code` —
46 barangays abolished in the 2023 Bacoor merger carry new-format codes in the
2022 layer and won't prefix-match their parent (the `parent` field is still
correct). See [`DATA_NOTES.md`](DATA_NOTES.md).

## What changed each year

- **2022** — baseline. Maguindanao already split into del Norte / del Sur; the
  BARMM Special Geographic Area present. Bacoor still has its 44 pre-merger
  barangays; Carmona is still a municipality; Sulu in BARMM; the EMBO barangays
  under Makati; no Negros Island Region.
- **2023** — Bacoor barangay merger (44 → 18, City Ordinance 275-2023, eff.
  2023-07-29); Carmona becomes a city (RA 11938, eff. 2023-09-30).
- **2024** — EMBO barangays transferred Makati → Taguig (eff. ~2024-06-01);
  **Negros Island Region** established (RA 12000, eff. 2024-06-11); Sulu moved
  from BARMM to Region IX (Supreme Court ruling, eff. 2024-09-18). This is the
  current configuration — no boundary changes since.

## Counts

| Level | 2022 | 2023 | 2024 |
|---|--:|--:|--:|
| region | 17 | 17 | 18 |
| province (incl. Special Geographic Area) | 83 | 83 | 83 |
| city | 148 | 149 | 149 |
| municipality | 1,494 | 1,493 | 1,493 |
| sub-municipality (Manila) | 14 | 14 | 14 |
| barangay | 42,025 | 41,999 | 41,999 |

## How it was made

A reconciliation of two open sources, projected through a temporal model:

1. **Geometry** — PSA/NAMRIA administrative boundaries via the UN OCHA
   Humanitarian Data Exchange (COD-AB Philippines).
2. **Roster / codes / names** — the official PSGC 1Q-2026 publication (PSA), plus
   the legacy correspondence codes.

The barangay layer was reconciled unit-by-unit to the official PSGC roster
(**99.97%** exact code match for the current year); region, province,
city/municipality and sub-municipality layers match official counts and codes
exactly. Each year is then reconstructed from validity intervals (when each unit
existed) and parent edges (who its parent was at that time). Integrity is clean
for every year: 0 invalid geometries, 0 duplicate codes, 0 orphaned parent links.

## Resolution

Geometry is generalized for web/analytical use (barangays ~11 m; parent layers
coarser). Not survey-grade. The barangay layer intentionally **excludes
non-barangay land parcels** (forest lands, national parks, watersheds, etc.) that
the source carries but PSGC does not classify as barangays; those areas remain
within the enclosing city/municipality polygons.

## Known limitations

See [`DATA_NOTES.md`](DATA_NOTES.md). In brief: 2024 has 2 codeless barangays
(a Caloocan split we can't subdivide, a contested Calaca barangay); the 46
abolished-2023 barangays carry new-format codes in the 2022 layer; and only the
current (2024) codes are reconciled against an official roster — earlier years
rely on the legacy codes we hold, not a year-specific PSGC publication.

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
