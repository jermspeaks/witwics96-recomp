# Project Plan

Last updated: 2026-10-13

This is the handoff plan. Keep statuses current so a new agent can resume without repeating work. A completed phase means its stated outputs exist and its validation is recorded; it does not mean the whole port is complete.

## Non-negotiable input policy

Bring your own legally obtained game copy. `input/` is intentionally ignored by Git. Do not commit the original game, extracted proprietary assets, or other copyrighted payloads. Keep any analysis local and read-only unless the owner explicitly authorizes a different operation.

## Goal

Create a playable macOS port of the 1990s Windows game identified by the project owner, while keeping the repository free of the original game and proprietary payloads.

## Current status

- [x] Establish repository agent guidance and the local game-copy ignore rule.
- [x] Record the initial metadata-only inventory and open questions.
- [x] Identify the supplied release/build and map its executables, data files, and runtime dependencies (Phase 1, closed 2026-10-12; unknowns carried forward in `docs/findings.md`).
- [x] Describe the core gameplay loop and define a testable minimum playable scope (Phase 2, closed 2026-10-13; M1 criteria in `docs/gameplay-spec.md` §4).
- [x] Choose and document the macOS implementation and local asset-loading strategy (Phase 3, closed 2026-10-13: ScummVM `csworld` engine; prototype renders locations and plays movies).
- [ ] **Next:** Implement the smallest end-to-end playable slice (M1, `gameplay-spec.md` §4).
- [ ] Add automated tests/fixtures that do not contain copyrighted game payloads.
- [ ] Verify build/run behavior on the declared macOS target(s), documenting environment and gaps.
- [ ] Document setup, bring-your-own-copy requirements, known limitations, and packaging policy.

## Work sequence and handoff

### Phase 0 — Repository and evidence hygiene (complete)

**Outputs:** `AGENTS.md`, `.gitignore`, this plan, and `docs/findings.md`.

**Validation:** Confirm `input/` is ignored and that no game files are staged/tracked. At this stage no source code, tests, build system, or implementation approach has been established.

### Phase 1 — Static artifact inventory (complete, 2026-10-12)

**Outputs:** Both executables are identified in `docs/findings.md` with hashes. The 16-bit build is NE `input/SETUP16/CARMEN`, module `CARMEN`, “Carmen World 3.0”. The 32-bit build is `Carmen32.exe`: PE32 i386, Borland C++, extracted with owner approval from `ARCHIVE.C2K`. The ledger also covers `.MHK` (Mohawk) and `.MOV` (QuickTime) data identities, static imports, dynamically named DLLs (`QTIM32.DLL`, `CMGR32.DLL`, `DSOUND.DLL`), and the referenced config/data names.

**Local analysis state (outside Git):**
- `/private/tmp/witwics96-unshieldv3/` holds unshieldv3 at `4e5700f…` and its build.
- `/private/tmp/witwics96-win16ne/checkout/` holds win16ne at `541a16c…`.
- `/private/tmp/witwics96-rust/` holds a scoped rustup/cargo toolchain, the win16ne build in `target/release/win16ne`, the `extract-one` helper, and `extracted/Carmen32.exe`.
- `/private/tmp/witwics96-verify/` holds a reproduced `Carmen32.exe` (same SHA-256) and the win16ne output in `ne-describe.txt`.

To use the toolchain, set `RUSTUP_HOME`/`CARGO_HOME` to the `rustup`/`cargo` subdirectories. `/private/tmp` may be cleared on reboot. If it is, re-create these from the commits and hashes in the findings.

**Validation performed:** The `Carmen32.exe` extraction was reproduced byte-for-byte (SHA-256 `c476e52d…e646`, 425,504 bytes, matching the archive directory). `file` reports PE32 i386 GUI. REA `open_binary`, `binary_overview`, and `inspect_pe_resources` succeeded. win16ne exited 0 on the NE file with no stderr. Input hashes for `CARMEN`, `ARCHIVE.C2K`, and `DATABASE.MHK` are unchanged. Nothing was executed or installed, and there are no port tests.

**Carried-forward unknowns:** Media authenticity; where `SYSTEM.MHK`, `MOHAWK.INI`, `MIDIMAP.DAT`, `QTIM32.DLL`, and `CMGR32.DLL` come from; the structure of the other 14 `.MHK` files; and all runtime behavior.

### Phase 2 — Behavior and playable-scope definition (complete, 2026-10-13)

**Questions:** What is the essential game loop, what state/resources drive it, and what can be reliably tested?

**Progress (2026-10-12):** The draft spec `docs/gameplay-spec.md` covers the loop, end states, the archive/data model, and proposed M1 acceptance criteria. All 15 `.MHK` directories are surveyed. Clue pools, the country/loot/geo tables, the crook roster, and the trait table are identified from data, but not yet confirmed in code. The build is pinned to ScummVM's `csworld` v3.0 detection hash. The owner approved extracting `Carmen32.cfg`/`mohawk.w32`; it was done, and neither locates `SYSTEM.MHK`. The owner chose Apple Silicon on recent macOS as the target.

**Helpers (outside Git, `/private/tmp/witwics96-tools/`):** `mhk_types.py` (directory summary), `mhk_strl.py` (string-list summary), `mhk_peek.py` (hexdump of one resource), `mhk_clue_fields.py` (clue prefix tabulation), and `extract-allowed` (allowlisted archive extractor). Run Python with `-I` from `/private/tmp`. Do not paste game text into the repository.

**Progress (2026-10-13):** Code-level rules for case generation, battery, witness clues, arrest, promotion, and movie selection are recorded in findings and spec §§1–3. The `tSCR` framing is validated on 3,820 scripts. ScummVM's Mohawk `View`/`Feature` framework already loads `tCNT`, `tSCR`, and `tBMH`.

**Closed 2026-10-13:** The warrant rules, guide buttons, and almanac lookup were resolved from code (see findings). The M1 acceptance criteria are final in `gameplay-spec.md` §4, and deferred questions are mapped to later milestones in §5. Validation is static analysis only. The original game was not run, and no port code or tests exist yet.

### Phase 3 — macOS design and prototype (complete, 2026-10-13)

**Questions:** Which technology and rendering/input/audio approach fit the verified formats and target macOS versions? Target: Apple Silicon, recent macOS (owner, 2026-10-12). **Decision (owner, 2026-10-13): option A.** Build a clean-room `csworld` engine inside ScummVM (GPLv3). It reuses ScummVM's Mohawk archive, bitmap, palette, and sound support, its QuickTime playback, and its macOS packaging. The rejected alternative was a standalone native Swift/SDL app.

**Development setup (2026-10-13, verified):**
- **Fork:** a ScummVM clone at `/Users/jeremywong/Downloads/scummvm-csworld` (`--filter=blob:none`, full history), on local branch `csworld` based on `master` `cbc6b583c8e71d885f16ebe6b2b143430b1243ca`. It is outside this repository, has no commits, and has not been pushed or forked on GitHub.
- **Build:** `./configure --disable-all-engines --enable-engine=mohawk,cstime,myst,mystme,riven --enable-debug && make -j$(sysctl -n hw.ncpu)` succeeded in about 23 s on arm64 (SDL 2.32.70 via Homebrew `sdl2-compat`). The 34 warnings are all in OpenGL shader code, none in mohawk.
- **Detection:** `./scummvm --config=/private/tmp/witwics96-scummvm.ini --detect --path=<repo>/input` reports `mohawk:csworld  Where in the World is Carmen Sandiego? (Windows/English)`. Starting it would fail with `GAME_NOT_IMPLEMENTED`, because there is no `GType_CSWORLD` case in `metaengine.cpp`.
- **Config:** always pass `--config=/private/tmp/...` so the user's real ScummVM preferences aren't created or touched.
- **Asset loading:** the user adds their own game folder in ScummVM. Nothing is copied into the fork or the package.

**Prototype status (2026-10-13):**
- **Done:** wired the `csworld` subengine (item 1 of the earlier plan); opened all 15 archives; rendered a country panorama with its palette and a witness's first frame; played a Cinepak clip; adopted the true-color screen with an 8-bit back buffer; and added development PNG dumps.
- **Uncommitted:** the fork changes are local only. Commit them when the owner asks.
- **Running it:**
  1. Build: `./configure --disable-all-engines --enable-engine=mohawk,cstime,csworld,myst,mystme,riven --enable-debug && make -j8`.
  2. Run: `./scummvm --config=/private/tmp/witwics96-scummvm.ini csworld-win`.
  3. The ini has a `[csworld-win]` section with `gameid=csworld`, `engineid=mohawk`, and `path=<repo>/input`, plus the optional development keys listed in findings.
  4. The live window works too: arrow keys scroll and change country, and space pauses.

**Architecture (decided):**
- **Engine:** a `csworld` subengine in ScummVM's `engines/mohawk`, reusing `MohawkArchive`, `MohawkBitmap`, `VideoManager` (QuickTime/Cinepak/CDToons), and ScummVM's macOS packaging.
- **Screen:** true color, with an 8-bit back buffer and a panorama buffer converted using the merged game palette.
- **Code layout:** pure data and logic in `csworld_data.*`, unit-tested with synthetic fixtures under `test/engines/csworld/`.
- **Game data:** the user adds their own game folder in ScummVM; nothing is copied.

**Closed 2026-10-13:**
- **Scene composition** is implemented from code analysis.
- **CDToons** variant is fixed.
- **Data readers** are tested (411/411).
- **Interface panel** is drawn: `C2K tBMP 90`.

**Carried into Phase 4:** witness placement in panorama coordinates; mouse edge scrolling; the interface palette overlap; the night palette; and regression-testing the CDToons fix against `cstime` before any upstream submission.

**Exit criteria:** Documented architecture decision, buildable prototype, and repeatable development setup.

### Phase 4 — Implementation, validation, and packaging (next)

Implement in small vertical slices. Keep tests and fixtures free of copyrighted payloads. Validate functionality on declared macOS targets, include failure handling for missing/incompatible user-provided data, and ensure packaging does not silently include ignored local files.

**Exit criteria:** Reproducible build and test instructions; tested playable loop; clean package-content inspection; documented limitations and user-supplied-copy setup.

## Risks and unresolved decisions

See `docs/findings.md` for the detailed open-question list. The 32-bit `Carmen32.exe` is now extracted and loadable in Ghidra, and win16ne parses the 16-bit NE, so static code analysis is no longer blocked. Remaining risks are unverified resource semantics and runtime behavior, plus unlocated dependencies (`SYSTEM.MHK`, `MOHAWK.INI`, `QTIM32.DLL`, `CMGR32.DLL`). The macOS target/implementation strategy is undecided. Legal rights to redistribute the game or its assets are not established; do not include those payloads.

## Session handoff checklist

At the end of each meaningful task:

1. Add dated findings with their source, evidence IDs (if any), confidence, and limitations.
2. Update the status/next action here; do not mark a phase done on filenames or assumptions alone.
3. Record exact validation commands/results, including failures or tests not run.
4. Confirm `input/` remains ignored and unmodified unless explicitly required.
