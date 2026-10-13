# Gameplay and Minimum-Playable Specification (draft)

Last updated: 2026-10-13. Status: **draft, Phase 2 in progress**. Code-level rules (marked *code*) come from static analysis of `Carmen32.exe`; addresses are in `findings.md`.

This describes the 1996 v3.0 Windows release's behavior in clean-room terms, as a basis for the port. Every statement is labeled **Observed**, **Inferred**, or **Unknown**; evidence is in [`findings.md`](findings.md). This document deliberately contains no game text (clues, dialogue, character names, essays) or other payload. A port must read those at runtime from the user's own copy.

## 1. Core loop

| Step | Behavior | Confidence | Basis |
|---|---|---|---|
| Sign-in | The player types or selects a name. Names can be deleted, and progress (rank) is kept per name. | Inferred | UI string list `C2K STRL 2`; `User.Dat`/`Fame.Dat` file names; rank labels |
| Case assignment | The Chief briefs the player: a crook has stolen a country-specific treasure, and the trail starts in that country. | Inferred | 50 loot strings, one per country (`COUNTRY STRL 2001`); `GREET*`/`LOOT*` movie templates; briefing/“Babel-Link charging” UI strings |
| Investigate | Each stop has 3–5 witnesses, depending on rank. On the trail, a witness can give a clue to the next country and a trait clue. Off the trail, witnesses only give dead ends. | Observed (code) | §3.3 |
| Guide | The player can ask a named guide for help. | Inferred | “Request Guide” label and guide names in `C2K STRL 100`; `GUIDES.MHK` |
| Travel | The player picks a destination country. A flight screen shows departure/arrival, flight time, and time zones crossed. | Inferred | `C2K STRL 100` flight labels; per-country lat/long table (§3.2) |
| Resource limit | The Babel-Link battery starts at 100 for every rank. Each flight costs 5, flat. Witness actions cost 1 each (one action is free). Other actions cost 2–5 by rank: one set, probably the guides, costs 2,2,3,3,3,4,4,5; a text-query screen costs 3,3,4,4,4,5,5,5. At 0 the case resolves as a loss. | Observed (code); guide attribution inferred | `cw_DrainBattery` |
| Warrant | The player fills in five traits and issues a warrant. A warrant can be changed. | Inferred | Warrant/trait labels in `C2K STRL 100` |
| Arrest | At the final location, the player must pick the crook from among innocent bystanders. A crook-specific capture animation then plays. | Inferred | Capture statements ending each clue pool; `ARREST1/2.MHK`; public description |
| Outcome | Win (loot returned, possible promotion), or one of the lose cases in §2. | Inferred | Win/lose headline strings; `WIN*`/`LOSE*`/`ENDING*` movie templates |
| Promotion | There are 8 ranks (0–7). Rank goes up when total wins reach 2, 4, 9, 15, 22, 30, or 39. Crooks 0–38 are chosen at random from those not yet caught. Once 39 are caught, the case is against Carmen (crook 39). Losing to Carmen reduces the caught count by 3 and re-randomizes some caught crooks. | Observed (code) | `cw_CheckPromotion`, `cw_NewCase` |

## 2. End states (code)

The arrest resolves as a **win** only if all of these hold: battery > 0, a warrant has been issued, the warrant is correct, and the chosen person is the crook. Otherwise the outcome is one of these losses, checked in this order:

| Outcome | Movie |
|---|---|
| Battery depleted | `LOSEPWR1–5` |
| No warrant | `LOSENOW1–5` |
| Right person, wrong warrant | `LOSEWRG1–5` |
| Wrong person | `LOSEPER1–5` |
| Loss against Carmen | `LOSEFIN2` |

On a win, the game plays a capture animation from `ARREST1/2.MHK` (chosen by the crook's sex) and then a `WINA`/`WINB` clip. A Carmen win plays `ENDING*` instead.

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

### 3.3 Crooks, voices, and clues (code-confirmed)

The executable's own resource labels identify: `STRL/BYTS 101` as villain names, `BYTS 102` as voice groupings, `BYTS 103` as a time-zone table, `BBOX/STRL 210` as physical evidence, and `STRL 30000+v` as voice text.

- **Crooks:** there are 40. Crooks 20 and above are female. `BYTS 101` holds 5 bytes per crook: hair, height, weight, accessory, and voice + 1.
- **Voices:** the 40 `STRL 30000+v` lists belong to **witness voices**, not to crooks. In each list:
  - Strings 1–37 are country clues of the form `countryIndex,difficulty,text`.
  - Strings 38–52 are trait phrasings (category bases 38/43/46/49).
  - String 55 is the off-trail dead end.
  - String 56 is the final-stop line.
- **Case setup:**
  - The start (loot) country is random. It is never the current country or the previous start country.
  - The route has 4, 4, 5, 5, 6, 6, 7, or 7 stops for ranks 0–7.
  - The route is built incrementally. Each time the player reaches the trail head, the next country is picked at random.
  - The flight screen offers 6 destinations: recent countries, the trail link, and random fillers, sorted.
- **Trait evidence:** each of the 4 traits (hair, height, weight, accessory) is placed at a different random stop, as a `STRL 210` physical-evidence entry.
- **Witnesses on the trail:** each witness gets an unused voice. Voices 15 and 19 are excluded for female crooks. The game scans strings 1–37 from a random start for a clue naming the next country. The clue's difficulty must lie between min `{0,0,0,0,1,1,1,2}` and max `{0,1,1,1,2,2,2,2}` by rank; the minimum is relaxed after 40 tries.
- **Witness actions:** one action asks for the destination clue and one for the trait clue, each costing 1 battery.
- **Final stop:**
  - There are 4, 4, 5, 5, 5, 6, 6, 6 people, by rank.
  - Witness 0 is the crook.
  - Bystanders are look-alikes sharing at least 2, 2, 2, 3, 3, 3, 4, 4 traits with the crook.
- **Fail-safe:** the original falls back to a fail-safe path when its random searches run long. A port should bound these loops explicitly.

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
2. Starts a case following §3.3: crook, loot country, and route length by rank.
3. Each stop shows the country background and the rank-based number of witnesses. Clues are selected as in §3.3.
4. Travel costs 5 battery and witness actions cost 1, starting from 100. A wrong destination gives dead-end replies.
5. The warrant can be issued and changed, and arresting at the final stop resolves to win, wrong warrant, no warrant, or wrong person.
6. Running out of battery ends in a loss.
7. Plays the matching `.MOV` clips if a QuickTime decoder is available. If not, it skips them without breaking the flow.
8. Automated tests use synthetic Mohawk fixtures only, with no game bytes.

Later milestones (out of M1 scope): animated witness and guide characters (`tSCR` scripts), arrest animations, rank progression and save files, the almanac database, speech, the opening movie, and the 16-bit build.

## 5. Open questions blocking a final spec

Answered on 2026-10-13 from code: case generation, battery, witness clue selection, arrest outcome, promotion, and the movie mapping (findings §Phase 2). Remaining:

1. Which UI the rank-costed actions belong to: buttons 3–9 (probably guides) and the text-query screen.
2. Where the warrant-issued and warrant-correct flags are set, and what the case-start drain of 5 pays for.
3. The `User.Dat` field layout. Only the XOR checksum is known; `Fame.Dat` is 20-byte records plus a checksum.
4. Flight-time and time-zone computation (`INTS 2004/2006`, `BYTS 103`). This is display only; it doesn't affect battery.
5. `tSCR` opcodes 3, 7, 9, and 0xF, and lip-sync timing (see findings).
6. The meaning of voice strings 53, 54, and 57, and confirmation of the trait-string order.
