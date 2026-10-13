# Gameplay and Minimum-Playable Specification (draft)

Last updated: 2026-10-12. Status: **draft, Phase 2 in progress**.

This describes the 1996 v3.0 Windows release's behavior in clean-room terms, as a basis for the port. Every statement is labeled **Observed**, **Inferred**, or **Unknown**; evidence is in [`findings.md`](findings.md). This document deliberately contains no game text (clues, dialogue, character names, essays) or other payload. A port must read those at runtime from the user's own copy.

## 1. Core loop

| Step | Behavior | Confidence | Basis |
|---|---|---|---|
| Sign-in | The player types or selects a name. Names can be deleted, and progress (rank) is kept per name. | Inferred | UI string list `C2K STRL 2`; `User.Dat`/`Fame.Dat` file names; rank labels |
| Case assignment | The Chief briefs the player: a crook has stolen a country-specific treasure, and the trail starts in that country. | Inferred | 50 loot strings, one per country (`COUNTRY STRL 2001`); `GREET*`/`LOOT*` movie templates; briefing/“Babel-Link charging” UI strings |
| Investigate | At each location, the player questions people. They answer with a destination clue (country), a suspect-trait clue, or a dead end. | Inferred from data shape | Clue pools (§3.3); public description of bystanders and paper scraps |
| Guide | The player can ask a named guide for help. | Inferred | “Request Guide” label and guide names in `C2K STRL 100`; `GUIDES.MHK` |
| Travel | The player picks a destination country. A flight screen shows departure/arrival, flight time, and time zones crossed. | Inferred | `C2K STRL 100` flight labels; per-country lat/long table (§3.2) |
| Resource limit | Every action drains the ACME Babel-Link battery. When it runs out, the case is lost. Public sources call this a “fuel limit”. | Inferred | “Babel-Link battery” lose string; `LOSEPWR%d.MOV` template |
| Warrant | The player fills in five traits and issues a warrant. A warrant can be changed. | Inferred | Warrant/trait labels in `C2K STRL 100` |
| Arrest | At the final location, the player must pick the crook from among innocent bystanders. A crook-specific capture animation then plays. | Inferred | Capture statements ending each clue pool; `ARREST1/2.MHK`; public description |
| Outcome | Win (loot returned, possible promotion), or one of the lose cases in §2. | Inferred | Win/lose headline strings; `WIN*`/`LOSE*`/`ENDING*` movie templates |
| Promotion | There are 8 ranks, from Rookie to Super Sleuth. Final cases appear to involve Carmen (`FNLBRF%d`, `ENDING*`). | Inferred | Rank labels; roster's last entry; movie templates |

## 2. End states (observed strings; trigger conditions inferred)

- Win: suspect arrested with the correct warrant, and loot returned. Five headline variants.
- Lose: no warrant obtained (`LOSENOW`?).
- Lose: improper warrant presented (`LOSEWRG`?).
- Lose: innocent tourist arrested.
- Lose: Babel-Link battery depleted (`LOSEPWR`).
- Lose: suspect escapes or the player loses the trail (`LOSEPER`?).

The mapping from lose condition to movie name is **Unknown**. Confirm it in code.

## 3. Data model (from the player's own copy)

### 3.1 Archives

All 15 `.MHK` files are Mohawk v1 archives. Their structure validated, and every resource lies within its file.

| Archive | Inferred role | Resource types |
|---|---|---|
| `C2K.MHK` | Core UI, clue pools, crook tables, shared art/sound | `STRL`, `BYTS`, `BBOX`, `tBMP`, `tPAL`, `tCNT`, `tSCR`, `tWAV` |
| `COUNTRY.MHK` | Per-country/continent backgrounds, sound, name/loot/geo tables | `STRL`, `INTS`, `BYTS`, `tBMP`, `tPAL`, `tCNT`, `tWAV` |
| `DATABASE.MHK` | Country almanac (fact sheets, captioned photos) | `BYTS`, `STRL`, `tBMP`, `tPAL`, `tCNT`, `tWAV` |
| `DIALOG.MHK` | Spoken dialogue (2,314 sounds) | `tWAV` |
| `GUIDES.MHK` | Guide characters | `tBMP`, `tBMH`, `tSCR`, `tWAV`, `INTS`, `BYTS` |
| `CHR1`–`CHR8.MHK` | Animated witnesses/characters | `tBMH`, `tSCR`, `tCNT`, `tPAL` |
| `ARREST1/2.MHK` | Arrest animations | `tBMP`, `tSCR`, `tWAV`, `tCNT`, `tPAL` |
| `MOVIES/*.MOV` | QuickTime full-motion video, selected by numbered name templates | — |

Per ScummVM `engines/mohawk/resource.h` comments, `tSCR` is “Script (CSWorld…)”, `tCNT` is “Shape Count”, `STRL` is “String List”, `BYTS` is “Database Entry”, and `INTS` is a uint16 array. The `tBMP`/`tBMH`/`tPAL`/`tWAV` codecs are the standard Mohawk formats ScummVM already decodes.

### 3.2 Countries

- **Observed:** There are 50 countries, indexed 0–49 alphabetically, followed by 7 continent entries and a glossary (`COUNTRY STRL 2000`).
- **Observed:** There is one loot item per country (`COUNTRY STRL 2001`, 50 entries).
- **Inferred:** `COUNTRY INTS 2006` holds 50 signed latitude/longitude pairs; two spot checks matched real coordinates. They probably drive the flight time and time-zones-crossed display.
- **Unknown:** The meaning of `INTS 2000`, `INTS 2004` (50 x/y-like pairs, possibly map positions), `BYTS 2001` (values 1–7, possibly a region), and `BYTS 2002` (0/1 flags). Per-country background art is `tCNT`/`tPAL` IDs `2000 + 100·index`; this pattern is inferred.

### 3.3 Crooks and clue pools

- **Observed:** The roster has 40 crooks (`C2K STRL 101`). The labels say 20 are male and 20 are female, and the last entry is Carmen.
- **Observed:** There are 40 clue pools (`C2K STRL 30000`–`30039`) of 57 strings each. Each starts with about 37 country clues encoded as `countryIndex,difficulty(0–2),text`. These are followed by about 15 trait clues (hair ×5 phrasings, height ×3, weight ×3, accessory ×4), stock greeting and dead-end replies, and one capture statement.
- **Inferred:** There is one pool per crook, ordered like the roster. Difficulty 0–2 probably scales clue obscurity with rank.
- **Inferred:** `C2K BYTS 101` is a 40 × 5 crook trait table: hair (0–4), height (0–2), weight (0–2), accessory (0–3), and an unknown fifth column (1–255). Sex comes from the roster position.
- **Unknown:** `BYTS 102` (40 distinct values, apparently a permutation of 1–40), `BYTS 103`, and `BBOX 210`.

### 3.4 Warrant traits (observed labels)

| Trait | Values |
|---|---|
| Sex | Male, Female |
| Hair | Blonde, Brown, Black, Red, Gray |
| Height | Tall, Medium, Short |
| Weight | Thin, Average, Heavy |
| Wearing | Hat, Camera, Watch, Glasses |

## 4. Minimum playable milestone (proposed acceptance criteria)

Proposed milestone **M1, one complete case**, running on macOS 14+ on Apple Silicon:

1. Points at a user-selected local game folder, validates that `C2K.MHK` matches the known v3.0 detection hash, and fails clearly otherwise.
2. Starts a case with a randomly chosen crook, loot country, and a route of N countries (N to be confirmed from code).
3. Each stop shows the country background and offers at least three witnesses. Their clues come from that crook's pool, and their difficulty follows the rank.
4. Travel to a country costs battery. A wrong destination gives dead-end replies.
5. The warrant can be issued and changed, and arresting at the final stop resolves to win, wrong warrant, no warrant, or wrong person.
6. Running out of battery ends in a loss.
7. Plays the matching `.MOV` clips if a QuickTime decoder is available. If not, it skips them without breaking the flow.
8. Automated tests use synthetic Mohawk fixtures only, with no game bytes.

Later milestones (out of M1 scope): animated witness and guide characters (`tSCR` scripts), arrest animations, rank progression and save files, the almanac database, speech, the opening movie, and the 16-bit build.

## 5. Open questions blocking a final spec

Each should be answered from `Carmen32.exe` code (Ghidra) unless stated otherwise:

1. How a case is generated: route length per rank, how stops are chosen, and how the final location and bystanders are set up.
2. Battery cost per action (travel by distance? questioning?) and the starting charge per rank.
3. How witnesses pick clue type and difficulty, and the meaning of the clue pools' difficulty field.
4. Promotion rules and the format of the `User.Dat`/`Fame.Dat` save files.
5. The `tSCR` script format: whether witness animation and lip-sync need a script interpreter for M1.
6. Lose-condition → movie mapping, and the role of `GREET`, `EXPLORE`, `JANITOR`, and `Extra` clips.
