# Third Place Index (TPI)
## National Census Tract Dataset

**Version**: 2025 v1.1 (corrected October 2026)
**File**: `third_place_index_1.1.csv`
**Records**: 84,414 U.S. Census Tracts (2020 Census tract geography)
**Author**: Evan O'Neil / Data Studio of Evan O'Neil
**Contact**: evan@evanoneil.studio


---
## Changelog

### v1.1 correction — October 2026
This file replaces the v1.1 file published March 16, 2026. If you downloaded v1.1 before October 2026, download it again.

- **Connecticut GEOIDs now use 2020 tract codes.** The March file gave Connecticut's tracts 2022 planning-region county codes (09110–09190), so they did not join to 2020 tracts. Connecticut now uses its 2020 county codes (09001–09015), like every other state. Tract numbers did not change.
- **Connecticut has 883 tracts, matching the 2020 Census.** Census split one Connecticut water-only tract into two in 2022. The March file carried both halves; this file restores the single 2020 tract. Total records drop from 84,415 to 84,414. Water-only tracts contain no population or places.
- **All GEOIDs are 11 digits and all state FIPS codes are 2 digits.** The March file dropped leading zeros for states with FIPS codes 01–09 (15,662 rows).
- **New `geoid_2022` column.** Use it to join ACS 2022 and later data, which use Connecticut's planning-region codes. Outside Connecticut it equals `geoid`.
- **Connecticut scores corrected.** The March file scored Connecticut with a different formula from every other state. That raised Connecticut scores by 0.046 on average and by up to 0.08. Connecticut tracts are now scored exactly like every other tract. Scores outside Connecticut did not change.
- **Number formatting repaired.** The March file went through a spreadsheet. It wrote 31 incomes in scientific notation (for example, 1.00E+05) and rounded 17 low density values. Incomes are now whole numbers, and the 17 densities are restored to full precision from v1.0.
- **Documentation corrected.** Scores rank each tract within its urban/rural class, not against all U.S. tracts. `area_sq_miles` is not land area; see Limitations.
- **Validated** against the Census 2020 Gazetteer tract list (every GEOID present, none extra) and ACS 2022 5-year tables B01003 and B19013 (population and median income match for every tract, joined on `geoid_2022`).

### v1.1 — March 16, 2026
- Added Connecticut, which v1.0 omitted.

### v1.0 — January 13, 2026
- Initial release. Does not include Connecticut. Available from the `v1.0` tag.

---

## Reading the File

`geoid`, `geoid_2022` and `state_fips` are text codes with leading zeros. Read them as text, or they will lose their leading zeros and fail to join.

- **R:** `read.csv("third_place_index_1.1.csv", colClasses = c(geoid = "character", geoid_2022 = "character", state_fips = "character"))`
- **Python (pandas):** `pd.read_csv("third_place_index_1.1.csv", dtype={"geoid": str, "geoid_2022": str, "state_fips": str})`
- **Excel:** Do not double-click the file. Use Data → From Text/CSV and set these three columns to Text.

## Overview

The Third Place Index measures the availability of "third places" - community gathering spaces beyond home (first place) and work (second place) - at the census tract level across the United States. The concept originates from sociologist Ray Oldenburg's 1989 book *The Great Good Place*.

Third places include cafes, restaurants, libraries, parks, community centers, fitness centers, and other spaces where people gather for social interaction and community building.

### Key Features

- **National coverage**: All 50 states plus Washington, D.C.
- **Tract-level precision**: 84,414 census tracts
- **Percentile-based scoring**: Each tract ranked against U.S. tracts in the same urban/rural class
- **Detailed breakdowns**: Counts by category and individual place type

---

## Methodology

### Place Data Collection

Third place locations were collected from OpenStreetMap (OSM) via the Overpass API. Locations were identified across the following categories:

Cafes, restaurants, bars, pubs, barber shops, salons, bookstores, convenience stores, marketplaces, libraries, community centers, places of worship, parks, gardens, plazas, theaters, cinemas, arts centers, fitness centers, coworking spaces, bowling alleys, arcades, escape rooms, and malls.

### Index Calculation

The Third Place Index uses a **percentile-based methodology** :

1. Census tracts are grouped by urban/rural class: Urbanized Area, Urban Cluster or Rural
2. Within each class, tracts are ranked by total third place count
3. Each tract's score is the share of tracts in its class with fewer third places: (tracts with fewer places) ÷ (tracts in the class − 1). Tied tracts get the same score
4. A score of **0.75** means the tract has more third places than 75% of U.S. tracts in its urban/rural class

Scores use the v1.0 reference distribution: the 83,531 tracts outside Connecticut. Connecticut tracts, added in v1.1, are scored against that same distribution, so adding them changed no other tract's score.

**Interpretation**:
- 0.75 - 1.00: High availability
- 0.50 - 0.75: Above average
- 0.25 - 0.50: Below average
- 0.00 - 0.25: Limited availability ("third place desert")

---

## Data Dictionary

### Identifiers

| Column | Type | Description |
|--------|------|-------------|
| `geoid` | String | 11-digit 2020 Census tract GEOID. Joins to TIGER/Line 2020 tracts and 2020 Census data |
| `state_fips` | String | 2-digit state FIPS code |
| `geoid_2022` | String | 11-digit 2022-vintage tract GEOID (last column). Differs from `geoid` only in Connecticut, where it uses planning-region county codes. Blank for three Connecticut water-only tracts that have no single 2022 equivalent |
| `urban_rural_class` | String | "Urbanized Area" (50,000+ pop), "Urban Cluster" (2,500-50,000), or "Rural" |

### Index Score

| Column | Type | Range | Description |
|--------|------|-------|-------------|
| `third_place_percentile` | Float | 0-1 | Share of U.S. tracts in the same urban/rural class with fewer third places (see Index Calculation) |

### Demographics

| Column | Type | Description |
|--------|------|-------------|
| `total_population` | Integer | Total population (ACS 2022 5-year) |
| `median_income` | Float | Median household income in dollars |
| `pct_bachelors` | Float | Percent with bachelor's degree or higher |
| `pop_under_18` | Integer | Population under 18 years old |
| `pct_under_18` | Float | Percent under 18 years old |
| `area_sq_miles` | Float | Tract area in square miles from the TPI pipeline. **Not Census land area**; see Limitations. For land area, use `ALAND` from the Census 2020 Gazetteer or TIGER/Line files |

### Place Counts - Categories

| Column | Description |
|--------|-------------|
| `total_places` | Total count of all third places |
| `food_beverage_count` | Restaurants, cafes, bars, pubs, coffee shops |
| `personal_service_count` | Hairdressers, beauty salons |
| `traditional_retail_count` | Convenience stores, bookstores, marketplaces |
| `civic_educational_count` | Places of worship, libraries, community centres |
| `public_spaces_count` | Parks, gardens, squares |
| `cultural_arts_count` | Theatres, cinemas, arts centres |
| `commercial_retail_count` | Department stores, malls |
| `fitness_wellness_count` | Fitness centres, sports centres |
| `work_creation_count` | Coworking spaces |
| `entertainment_gaming_count` | Bowling alleys, arcades, escape games |

### Place Counts - Individual Types

**Food & Beverage**: `restaurant_count`, `cafe_count`, `bar_count`, `pub_count`, `ice_cream_count`, `coffee_count`, `food_court_count`, `biergarten_count`

**Personal Services**: `hairdresser_count`, `beauty_count`

**Retail**: `convenience_count`, `books_count`, `marketplace_count`

**Civic & Educational**: `place_of_worship_count`, `library_count`, `social_facility_count`, `community_centre_count`

**Public Spaces**: `park_count`, `garden_count`, `square_count`, `village_green_count`

**Cultural & Arts**: `theatre_count`, `cinema_count`, `arts_centre_count`

**Commercial**: `department_store_count`, `mall_count`

**Fitness**: `fitness_centre_count`, `sports_centre_count`

**Entertainment**: `bowling_alley_count`, `amusement_arcade_count`, `escape_game_count`

**Work**: `coworking_space_count`

### Density

| Column | Type | Description |
|--------|------|-------------|
| `total_places_per_sqmi` | Float | `total_places` ÷ `area_sq_miles`. Inherits the `area_sq_miles` error; see Limitations. NA where area is 0 (water-only tracts) |

---

## Data Sources

- **Place Data**: OpenStreetMap contributors (2025) via Overpass API
- **Demographics**: U.S. Census Bureau, American Community Survey 2022 5-Year Estimates
- **Geography**: U.S. Census Bureau, TIGER/Line Shapefiles 2020
- **Connecticut 2020–2022 tract crosswalk**: [CT Data Collaborative](https://github.com/CT-Data-Collaborative/2022-tract-crosswalk); water-only tracts checked against Census TIGERweb

---

## Limitations

- **OSM Coverage**: OpenStreetMap completeness varies by region; urban areas are generally better mapped than rural areas
- **Relative Measure**: The percentile score is relative, not absolute - it compares tracts to each other, not to an ideal standard
- **Presence vs. Quality**: The index measures presence of places, not their quality, accessibility, affordability, or actual usage
- **Point-in-Time**: Data represents a snapshot; third places open and close over time
- **Area and density**: `area_sq_miles` overstates tract area. Compared with Census 2020 land area, state totals run from 1.15× too high (Hawaii) to 5.5× too high (Alaska), and 2.35× nationally. The error grows with latitude, consistent with Web Mercator projection. `total_places_per_sqmi` inherits the error, so do not compare densities across regions or with other sources. Scores do not use area. v2 will use Census land area.

---

## License

- **Place Data**: OpenStreetMap data is available under the Open Database License (ODbL)
- **Census Data**: U.S. Census Bureau data is in the public domain


---

## Contact

Evan O'Neil
evan@evanoneil.studio
