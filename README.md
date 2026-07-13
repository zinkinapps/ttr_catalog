# TTRacker Catalog

This repository hosts the official game version catalog for **TTRacker**, a score tracking app for Ticket to Ride (Aventureros al Tren).

The catalog is a single JSON file (`catalog.json`) that the app download automatically on launch to stay up to date with new game versions and data corrections — no app update required.

---

## How it works

The app ship with a bundled copy of `catalog.json` as a fallback. On each launch, it attempts to download the latest version from this repository. If the download succeeds, the updated catalog is cached locally. If not, the app uses the last cached version.

The `catalog_version` field is used to detect whether an update is available.

---

## catalog.json structure

```json
{
  "catalog_version": "1.0",
  "catalog_url": "https://raw.githubusercontent.com/zinkinapps/ttr_catalog/refs/heads/main/catalog.json",
  "readme_url": "https://github.com/zinkinapps/ttr_catalog/blob/main/README.md",
  "versions": [ ... ]
}
```

| Field | Type | Description |
|---|---|---|
| `catalog_version` | string | Semantic version of the catalog. Increment on every change (e.g. `"1.0"` → `"1.1"`). |
| `catalog_url` | string | Canonical URL where apps fetch the JSON. Do not change this. |
| `readme_url` | string | URL of this README. Apps use this field to show a link in Settings — do not hardcode any URL in the app. |
| `versions` | array | List of game version objects (see below). |

---

## Version object

Each entry in `versions` represents one edition, expansion or map of the game:

```json
{
  "id": "europe",
  "status": "confirmed",
  "type": "game",
  "year_of_release": 2005,
  "related_to_id": null,
  "map_collection": null,
  "name_keys": {
    "es": "Europa",
    "en": "Europe",
    "fr": "Europe",
    "de": "Europa",
    "pt": "Europa",
    "it": "Europa"
  },
  "max_players": 5,
  "wagons_official": 45,
  "train_colors": ["blue", "green", "red", "yellow", "black"],
  "destination_tickets": {
    "normal": [5, 5, 5, 5, 5, 6, 6, 6, 6, 6, 7, 7, 7, 7, 7,
               8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8,
               9, 9, 10, 10, 10, 10, 10, 11, 11, 12, 12, 13],
    "long": [20, 20, 20, 21, 21, 21],
    "long_ticket_limit": 1,
    "ticket_control": true
  },
  "scoring_table": {
    "1": 1,
    "2": 2,
    "3": 4,
    "4": 7,
    "6": 15,
    "8": 21
  },
  "bonuses": {
    "longest_route": 10,
    "stations": {
      "per_player": 3,
      "points_per_unused": 4
    },
    "globetrotter": null,
    "extra_points": false
  }
}
```

Some city editions use a piece other than a wagon to claim routes. Those entries add a `piece_name_keys` field:

```json
"piece_name_keys": {
  "singular": {"es": "taxi", "en": "taxi", "fr": "taxi", "de": "Taxi", "pt": "táxi", "it": "taxi"},
  "plural":   {"es": "taxis", "en": "taxis", "fr": "taxis", "de": "Taxis", "pt": "táxis", "it": "taxi"}
}
```

Omit `piece_name_keys` entirely for versions that use standard wagons — the apps default to "wagon" in all six languages.

### Fields

| Field | Type | Description |
|---|---|---|
| `id` | string | Unique identifier. Use snake_case (e.g. `"nordic_countries"`). Never change an existing ID — it is stored in users' game history. |
| `name_keys` | object | Display name in each supported language. Include all six keys: `es`, `en`, `fr`, `de`, `pt`, `it`. |
| `max_players` | integer | Maximum number of players for this version. |
| `wagons_official` | integer | Number of route-claiming pieces per player, as stated in the rulebook (wagons, taxis, buses, etc. — whatever `piece_name_keys` names them). |
| `train_colors` | array of strings | Colors available in this version. Use only values from the official palette: `"blue"`, `"green"`, `"red"`, `"yellow"`, `"black"`, `"white"`, `"beige"`, `"purple"`, `"turquoise"`, `"pink"`. Order matters — it determines display order in the app. |
| `destination_tickets.normal` | array of integers | Point value of each normal destination ticket in the deck. Include one entry per physical card (duplicates allowed). |
| `destination_tickets.long` | array of integers | Point value of each long-distance destination ticket. Use `[]` if the version has none. |
| `destination_tickets.ticket_control` | boolean | Controls deck limit enforcement for both normal and long-distance tickets. `true`: when the sum of selections of a given value across all players (completed + uncompleted) reaches the count in the array, that value is disabled in the modal for all remaining players. `false`: same modal UI, but no value is ever disabled. Use `false` for versions where you know the point values but not yet the exact per-value card counts. |
| `destination_tickets.long_ticket_limit` | integer or null | Maximum number of long-distance destination tickets each player can introduce in total (completed + uncompleted). `null` if no limit applies. |
| `scoring_table` | object | Points awarded per route length. Keys are route lengths (as strings), values are points. Only include lengths that actually exist on the board. |
| `status` | string | Verification status. One of: `"confirmed"` (confirmed against physical rulebook), `"unconfirmed"` (sourced from official manuals/BGG but not personally verified), `"incomplete"` (known missing data). See [Status values](#status-values). |
| `bonuses.longest_route` | integer or null | Points awarded for the longest continuous route. `null` if not applicable. |
| `bonuses.globetrotter` | integer or null | Points awarded to the player(s) who completed the most destination tickets. `null` if not applicable. In case of a tie, all tied players receive the bonus. |
| `bonuses.stations` | object or null | Station bonus. Object with `per_player` (stations each player receives at setup) and `points_per_unused` (points per station not built by game end). `null` if the version has no stations. |
| `bonuses.extra_points` | boolean | `true` if the version has a special mechanic not otherwise modeled (e.g. Germany's passengers) and should offer a free-form extra-points field in the scoring wizard. `false` or absent otherwise. |
| `type` | string or null | Classifies the entry: `"game"` (standalone base game), `"map_expansion"` (a map that requires a base game's pieces), `"game_expansion"` (a card-only expansion for an existing map), `"cities"` (a compact city edition). Informational — doesn't affect scoring. |
| `year_of_release` | integer or null | Official release year. Informational. |
| `related_to_id` | string, array of strings, or null | The `id`(s) of the version(s) this one depends on or is based on (e.g. an expansion's base game). Can be a single id (`"europe"`) or several (`["deutschland", "germany"]`) when a version is tied to more than one base. `null` for standalone base games. |
| `map_collection` | string or null | Groups versions that ship together in the same physical box (e.g. two double-sided map expansions). Use the same string for every version in that box. `null` if it ships on its own. |
| `piece_name_keys` | object or null | Custom name for the route-claiming piece, when it isn't a wagon (taxis, buses, cable cars…). Object with `singular` and `plural`, each a `name_keys`-shaped block (all six languages). Omit entirely for wagon-based versions. |

---

## Status values

| Value | Meaning |
|---|---|
| `"confirmed"` | Verified against your own physical copy of the rulebook. |
| `"unconfirmed"` | Sourced from an official Days of Wonder manual and/or BoardGameGeek, but not personally checked against a physical copy. May contain errors — contributions to verify it are welcome. |
| `"incomplete"` | Known data is missing (e.g. destination tickets not yet added). The version appears in the app, but versions with this status are hidden from the "start a new game" version picker — they're only browsable from Settings. |

When submitting a PR, set `status` to the level that honestly reflects how you obtained the data, and mention your source in the PR description.

The app displays the status visually on each version's detail screen so users are aware of data quality.

---

## Important rules

**`bonuses.stations` is an object, not a number.** Use `{"per_player": 3, "points_per_unused": 4}` — not a plain integer. `null` if the version has no stations.

**Set `status` honestly.** `"confirmed"` only if you've personally checked the data against your own physical copy. `"unconfirmed"` for anything sourced secondhand (official manual, BGG). `"incomplete"` when data is known to be missing.

**Use only official color identifiers in `train_colors`.** Valid values: `"blue"`, `"green"`, `"red"`, `"yellow"`, `"black"`, `"white"`, `"beige"`, `"purple"`, `"turquoise"`, `"pink"`. Do not use hex codes — the app maps these identifiers to their visual representation.

**IDs are permanent.** Once a version is published and users have played games with it, its `id` must never change. Users' game history references the ID directly. If you need to fix a typo in the name, update `name_keys` — never the `id`.

**One entry per physical card.** `destination_tickets.normal` and `destination_tickets.long` should contain one integer per card in the physical deck (or as documented in the source you cite). If there are five cards worth 8 points, add `8` five times.

**Only real route lengths in `scoring_table`.** Some editions don't have routes of every length (e.g. Europe has no 5- or 7-wagon routes). Only include keys for lengths that physically exist on the board.

**Increment `catalog_version` on every change.** Semantic versioning: `"2.0"` → `"2.1"`, etc.

---

## Currently supported versions

### Base games

| ID | Name | Players | Year | Status |
|---|---|---|---|---|
| `ticket_to_ride` | Ticket to Ride | 2–5 | 2004 | ⚠️ Unconfirmed |
| `europe` | Europa / Europe | 2–5 | 2005 | ✅ Confirmed |
| `marklin` | Märklin | 2–5 | 2006 | ⚠️ Unconfirmed |
| `nordic_countries` | Países Nórdicos / Nordic Countries | 2–3 | 2007 | ⚠️ Unconfirmed |
| `deutschland` | Zug um Zug: Deutschland | 2–5 | 2012 | 🚧 Incomplete |
| `ticket_to_ride_10` | Ticket to Ride: 10th Anniversary | 2–5 | 2015 | 🚧 Incomplete |
| `rails_sails_the_great_lakes` | Rails & Sails — The Great Lakes | 2–5 | 2016 | 🚧 Incomplete |
| `rails_sails_the_world` | Rails & Sails — The World | 2–5 | 2016 | 🚧 Incomplete |
| `germany` | Alemania / Germany | 2–5 | 2017 | 🚧 Incomplete |
| `europe_15` | Europa: 15º Aniversario / Europe: 15th Anniversary | 2–5 | 2020 | 🚧 Incomplete |
| `northern_lights` | Northern Lights | 2–5 | 2022 | 🚧 Incomplete |

### Map expansions

| ID | Name | Players | Year | Status |
|---|---|---|---|---|
| `switzerland` | Suiza / Switzerland | 2–3 | 2007 | ⚠️ Unconfirmed |
| `asia` | Asia | 2–3 | 2011 | 🚧 Incomplete |
| `legendary_asia` | Asia Legendaria / Legendary Asia | 2–5 | 2011 | 🚧 Incomplete |
| `india` | India | 2–4 | 2011 | 🚧 Incomplete |
| `switzerland_2011` | Suiza / Switzerland (Map Collection 2) | 2–3 | 2011 | 🚧 Incomplete |
| `the_heart_of_africa` | El Corazón de África / The Heart of Africa | 2–5 | 2012 | 🚧 Incomplete |
| `nederland` | Nederland | 2–5 | 2013 | 🚧 Incomplete |
| `united_kingdom` | Reino Unido / United Kingdom | 2–4 | 2015 | 🚧 Incomplete |
| `pennsylvania` | Pensilvania / Pennsylvania | 2–5 | 2015 | 🚧 Incomplete |
| `france` | Francia / France | 2–5 | 2017 | ⚠️ Unconfirmed |
| `old_west` | El Viejo Oeste / Old West | 2–6 | 2017 | 🚧 Incomplete |
| `poland` | Polonia / Poland | 2–4 | 2019 | 🚧 Incomplete |
| `japan` | Japón / Japan | 2–5 | 2019 | 🚧 Incomplete |
| `italy` | Italia / Italy | 2–5 | 2019 | 🚧 Incomplete |
| `iberia` | Iberia | 2–5 | 2024 | ⚠️ Unconfirmed |
| `south_korea` | Corea del Sur / South Korea | 2–5 | 2024 | 🚧 Incomplete |

### Game (card) expansions

| ID | Name | Players | Year | Status |
|---|---|---|---|---|
| `usa_1910` | USA 1910 | 2–5 | 2006 | 🚧 Incomplete |
| `europe_1912` | Europa 1912 / Europe 1912 | 2–5 | 2009 | ⚠️ Unconfirmed |
| `europe_1912_big_cities` | Europa 1912 — Big Cities | 2–5 | 2009 | ⚠️ Unconfirmed |
| `europe_1912_mega` | Europa 1912 — Mega | 2–5 | 2009 | ⚠️ Unconfirmed |
| `deutschland_1902` | Deutschland 1902 | 2–5 | 2015 | 🚧 Incomplete |

### City editions

| ID | Name | Players | Year | Status |
|---|---|---|---|---|
| `new_york` | Nueva York / New York | 2–4 | 2018 | ⚠️ Unconfirmed |
| `london` | Londres / London | 2–4 | 2019 | 🚧 Incomplete |
| `amsterdam` | Ámsterdam / Amsterdam | 2–4 | 2020 | ⚠️ Unconfirmed |
| `san_francisco` | San Francisco | 2–4 | 2022 | 🚧 Incomplete |
| `berlin` | Berlín / Berlin | 2–4 | 2023 | 🚧 Incomplete |
| `paris` | París / Paris | 2–4 | 2024 | 🚧 Incomplete |

**✅ Confirmed** — Verified against a physical copy.
**⚠️ Unconfirmed** — Sourced from official manuals or BGG, not personally verified.
**🚧 Incomplete** — Known missing data.

---

## How to contribute

Contributions are very welcome, whether you own a physical copy or are working from official manuals and BoardGameGeek.

### Steps

1. **Fork** this repository.
2. Edit `catalog.json` with your data.
3. Make sure your changes follow the structure and rules described above.
4. **Increment `catalog_version`** (e.g. `"2.0"` → `"2.1"`).
5. Open a **Pull Request** describing what you changed and where the data came from (e.g. "Verified destination tickets for Amsterdam against my physical copy" or "Added Switzerland's destination ticket breakdown from the official rulebook + BGG").

### What we need most

- Filling in `destination_tickets` for the many `"incomplete"` versions listed above — most already have a point-value breakdown documented on BGG or in the official rulebook, they just haven't been transcribed into the catalog yet.
- Verifying `"unconfirmed"` entries against a physical copy, and upgrading them to `"confirmed"`.
- Any new editions not yet in the catalog.

### What not to do

- Do not change an existing `id`.
- Do not add route lengths to `scoring_table` that don't exist on the board.
- Do not remove existing versions, even if you think the data is wrong — open a PR with the correction instead.
- Do not guess or invent numbers. If a value is genuinely uncertain, leave the version as `"incomplete"` (or `"unconfirmed"` if you have a source but no physical copy) rather than making something up.

---

## Acknowledgments

This catalog is built from the official Days of Wonder rulebooks and manuals for each version, cross-referenced with the community-maintained data on [BoardGameGeek](https://boardgamegeek.com). Thank you to both for making this information accessible — this project wouldn't be possible without them.

---

## License

The catalog data is released under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/) (public domain). Ticket to Ride is a trademark of Asmodee. This project is not affiliated with or endorsed by Asmodee, Days of Wonder, or BoardGameGeek.
