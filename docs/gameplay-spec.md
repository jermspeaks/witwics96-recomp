# Gameplay and Minimum-Playable Specification

Last updated: 2026-10-13. Status: **Phase 2 complete (2026-10-13)**; items in §5 are deferred past M1. Code-level rules (marked *code*) come from static analysis of `Carmen32.exe`; addresses are in `findings.md`.

This describes the 1996 v3.0 Windows release's behavior in clean-room terms, as a basis for the port. Every statement is labeled **Observed**, **Inferred**, or **Unknown**; evidence is in [`findings.md`](findings.md). This document deliberately contains no game text (clues, dialogue, character names, essays) or other payload. A port must read those at runtime from the user's own copy.

## 1. Core loop

| Step | Behavior | Confidence | Basis |
|---|---|---|---|
| Sign-in | The player types or selects a name. Names can be deleted, and progress (rank) is kept per name. | Inferred | UI string list `C2K STRL 2`; `User.Dat`/`Fame.Dat` file names; rank labels |
| Case assignment | The Chief briefs the player: a crook has stolen a country-specific treasure, and the trail starts in that country. | Inferred | 50 loot strings, one per country (`COUNTRY STRL 2001`); `GREET*`/`LOOT*` movie templates; briefing/“Babel-Link charging” UI strings |
| Investigate | Each stop has 3–5 witnesses, depending on rank. On the trail, a witness can give a clue to the next country and a trait clue. Off the trail, witnesses only give dead ends. | Observed (code) | §3.3 |
| Guide | Each country has a guide (`COUNTRY BYTS 2001`). A free tour (Request Guide) unlocks that guide from the next flight on. Asking an unlocked guide charges 2,2,3,3,3,4,4,5 battery by rank; the guide speaks one line judging how many warrant fields are right, and can be asked once per unlock. | Observed (code) | `0x421a70`, `0x428e09`, `0x416d1e` |
| Travel | The player picks one of 6 destination countries and pays 5 battery, whatever the distance. A flight screen shows departure/arrival, flight time, and time zones crossed. The flight time and time-zone figures are display only. | Observed (code); display computation Unknown | `cw_TravelTo`, `cw_EnterCountry_BuildFlightChoices` |
| Almanac | A free-to-browse database of 58 documents (50 countries, 7 continents, a glossary; `DATABASE BYTS 20099+doc`) with flag, map and photo/video views. Only **Find**, a case-insensitive substring search, costs battery: 3,3,4,4,4,5,5,5 by rank, charged even when nothing matches. | Observed (code) | `0x423b38` → `0x424fda` |
| Resource limit | The Babel-Link battery starts at 100 for every rank, and every cost below is taken from it. At 0 the case resolves as a loss. | Observed (code) | `cw_DrainBattery` |
| Warrant | The player fills in five traits and issues a warrant, which then locks. **Change Warrant** unlocks it. The warrant counts as correct only if all five traits are filled and all match the crook, so a partial warrant is wrong. Issuing costs 4 battery. | Observed (code) | Handler at `0x421337`–`0x421434` |
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

## 4. Minimum playable milestone M1 (accepted criteria)

M1 is **one complete case**, running as a `csworld` engine in a ScummVM fork on macOS 14+ on Apple Silicon. The original game data is read from the user's own copy at runtime.

1. **Detection:** ScummVM detects the user's folder through the existing `csworld` v3.0 entry (`C2K.MHK` hash) and refuses unknown variants with a clear message.
2. **Case setup** (§3.3): a crook is chosen from those not yet caught; a random start/loot country is chosen; the route has 4–7 stops depending on rank; and trait evidence is placed at random stops.
3. **Arrival at a country:** the country's background (`COUNTRY tCNT/tPAL/tBMP`) is shown. The rank-dependent number of witnesses are drawn as static first frames of their characters (`CHR* tBMH`; full `tSCR` playback is M2). Their clue and trait text is shown, selected by the §3.3 rules.
4. **Battery:** starts at 100. Travel costs 5 and each witness question costs 1; the free and cost-4 actions follow `cw_WitnessAction`. Issuing a warrant costs 4. At 0, the battery-loss outcome triggers.
5. **Travel:** the flight screen offers 6 choices. Reaching the trail head advances the trail; going elsewhere gives dead-end replies.
6. **Warrant:** five trait fields with issue/change. Correctness follows the §1 rule.
7. **Final stop:** the crook stands among look-alikes, and arrest resolves to win or to the loss outcomes in §2.
8. **Movies:** clips are chosen as in `cw_PlayMovie`, using ScummVM's QuickTime decoder. A missing or failed movie is skipped without breaking the flow.
9. **Tests:** automated tests use only synthetic Mohawk/STRL fixtures created by the project, never game bytes. Case setup, the clue window, battery accounting, and warrant checking are unit-tested with a seeded random generator.

**Out of M1**, deferred to M2+ in roughly this order:
- animated characters (`tSCR`) and speech (`DIALOG.MHK`)
- guides
- the almanac
- the arrest animations
- rank progression and saves (`Fame.Dat`/`User.Dat`)
- the flight-time display
- the Carmen endgame
- the 16-bit build

Before guides are implemented, the M1 UI hides or disables their buttons.

## 5. Deferred questions (not blocking M1)

Answered 2026-10-13 from code: case generation, battery costs (including guides, almanac, and warrant), witness clue selection, warrant validation, arrest outcome, promotion, and the movie mapping. Still open, each to be resolved in the milestone that needs it:

1. What guides say, and how guide clue content is chosen (M2).
2. The `User.Dat` field layout. `Fame.Dat` is 20-byte records plus an XOR checksum (M3, saves).
3. Flight-time and time-zone display (`INTS 2004/2006`, `BYTS 103`), and what the case-start drain of 5 pays for.
4. `tSCR` opcodes 3, 7, 9, and 0xF, and lip-sync timing (M2).
5. The meaning of voice strings 53, 54, and 57, and confirmation of the trait-string order. Check these with a synthetic-fixture test against our own reader, not against shipped text.
6. Witness actions 3 and 4: confirm which UI they map to.
