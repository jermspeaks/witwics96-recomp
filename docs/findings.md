# Findings Ledger

Last updated: 2026-10-12

This is the shared, resumable ledger for verified observations and unresolved questions. Record the source and scope of each finding. Label statements **Observed**, **Inferred**, or **Unknown**; keep REA Evidence IDs or other reproducible references when generated. Do not treat missing files, filenames, or incomplete searches as proof of absence.

## Project baseline

- **Observed (project brief):** The project owner wants a playable macOS port of *Where in the World Is Carmen Sandiego?*, described as a 1990s Windows game.
- **Observed (repository):** At initial setup, the repository contained a one-line `README.md` and a local `input/` directory; no `.gitignore` was present. Git reported the root `.DS_Store` and `input/` as untracked before this setup.
- **Observed (filesystem inventory, 2026-10-12):** `input/` occupied approximately 635 MB according to `du -sh` and contained 432 regular files. The inventory included the directories `AUTORUN`, `E_REG`, `MOVIES`, `SETUP16`, `SETUP32`, `UTILS`, `WORLD16`, and `WORLDNET`.
- **Observed (filename/extension inventory only):** The tree contains 218 `.MOV`-named files, 15 `.MHK`-named files, and 19 `.EXE`-named files, as well as setup, library, font, configuration, and other files. These counts describe names only; no formats, contents, code, or behavior were analyzed.
- **Observed (repository policy):** The owner requires a bring-your-own legally obtained copy. The local `input/` directory must stay out of Git, as must the game, extracted proprietary assets, and other copyrighted payloads.
- **Inferred:** The directory names and extensions look consistent with Windows installation/media data, but this has not been validated. Do not use this as a compatibility or format claim.
- **Unknown:** The exact release/version, authenticity, executable architecture, file-format details, runtime dependencies, licensing/redistribution rights, and whether any original component runs on current macOS have not been established.
- **Unknown:** The intended technical approach (native reimplementation, compatibility layer, or another architecture), minimum macOS version, and acceptable local asset workflow have not been decided.

## Phase 1 — static artifact reconnaissance (in progress)

- **Observed (supplied documentation):** `input/README.TXT` identifies itself as a read-me for *Where in the World Is Carmen Sandiego?* v3.0 and is dated 1996-04-24. It documents Windows 3.1 and Windows 95 variants, references QuickTime 2.1 for digital video/opening animation, and discusses 256-color and 16-bit-color modes. These are claims in a supplied read-me, not independent authentication of the copy or proof of runtime behavior.
- **Observed (supplied setup metadata):** `input/AUTORUN.INF` declares autorun, installation, registration, and read-me commands, including separate `SETUP16` and `SETUP32` installer entries. `input/SETUP.CFG` declares a source file named `CARMEN` to be installed as `carmen.exe`, with size 558,608 bytes.
- **Observed (file identity):** `input/SETUP16/CARMEN` is 558,608 bytes, and the host `file` utility identifies it as an MS-DOS NE executable for Windows 3.x. Its SHA-256 is `246b63519dd84f1fac070045972c759dd4c38bc5110d6c74106f7903ff49f83e`.
- **Inferred (bounded):** The matching setup entry, file name, and declared size make `input/SETUP16/CARMEN` a strong candidate for the Windows 3.1 game executable. The setup table is declarative; installation was not performed and no execution was observed.
- **Observed (data type survey):** A focused `file` scan classified the 218 `.MOV`-named files as 211 fast-start and 7 unoptimized Apple QuickTime movies. Fifteen items in the selected `.MHK`/`.DAT`/`.BMP`/`.HTM`/`.Z` set were reported only as generic `data`; five were identified as InstallShield Z archives. The `.MHK` format and contents remain unknown. These are signature-based classifications, not proof of how the game consumes the files.
- **Observed (analysis tooling):** REA package 6.4.0 is connected. Ghidra 12.1.4 is available on macOS arm64; Hopper is unavailable. `open_binary` rejected both `input/SETUP.EXE` and `input/SETUP16/CARMEN` as unsupported NE executables. No function decompilation or REA Evidence record was produced for these targets. The installed skill file is version 40 while the live server reports expected skill version 37; the actual tool inventory and schemas were used.
- **Observed (repository safety):** `git check-ignore -v input` confirms the root `/input/` ignore rule; `git ls-files -- input` returned no tracked paths. The inspections were read-only: no game file was extracted, modified, or executed.
- **Unknown:** Whether a separate 32-bit game executable is included in the supplied media, how the InstallShield data maps to installed files, the exact formats/contents of the `.MHK` data, the complete runtime dependency set, and the game's runtime behavior remain unresolved. No build or tests exist/run for a port at this stage.

## Evidence and change log

| Date | Observation | Source / scope | Limits |
|---|---|---|---|
| 2026-10-12 | Initial repository and `input/` inventory; counts above | Project tree listing, `git status`, `find` file count and extension count, `du -sh input` | Metadata only; not evidence of content, runtime behavior, or rights |
| 2026-10-12 | Global Claude skill is installed at `~/.claude/skills/reverse-engineer-anything/SKILL.md` | Local skill file; version 40 from its front matter | Instructions only; does not establish that REA MCP tools/providers are connected |
| 2026-10-12 | Project's supplied read-me documents v3.0 and 1996-04-24 | `input/README.TXT`; SHA-256 `7b5548978a8e1cfb1582c6f6608175215d57ec0eefd994183db799d82ea1a9f9` | Document claim; does not authenticate media/release or prove runtime behavior |
| 2026-10-12 | Autorun/install declarations and CARMEN-to-`carmen.exe` size mapping | `input/AUTORUN.INF` (SHA-256 `1557a345cdbac95f75c39288ebb8e6cc1bb681c70f3119c2c4bda7e140e3bd51`), `input/SETUP.CFG` (SHA-256 `9fe452f1c94a0c9ed92bfd897973b2beb18eea70d20c82f2f54fd26a129cd766`) | Static declarations only; installation not performed |
| 2026-10-12 | 16-bit executable candidate identified and its original-file digest recorded | `input/SETUP16/CARMEN`; 558,608 bytes; SHA-256 `246b63519dd84f1fac070045972c759dd4c38bc5110d6c74106f7903ff49f83e`; `file` classification | Ghidra rejected NE; no decompilation or runtime observation |
| 2026-10-12 | Candidate inventory and media signature summary | Read-only `find`/`file` scans over named executables and selected media extensions | Extensions/signatures do not establish application use; generic data remains unidentified |
| 2026-10-12 | REA readiness/provider observations | `binary_session`; server/package 6.4.0; Ghidra 12.1.4 on darwin/arm64; open attempts above | No accepted target, so no native-analysis Evidence ID; Hopper unavailable |

## Open questions

1. **Partially answered:** What release/build is represented? The supplied read-me says v3.0 (1996-04-24), but authenticity and exact media build remain unverified. Next: determine whether a distinct 32-bit game program is included and identify the installed-file map without running setup.
2. **Partially answered:** What data/resource formats and dependencies are required? The read-me references QuickTime 2.1; the `.MOV`-named files are QuickTime-classified and the `.MHK` items remain generic data. Next: select a small, representative data file for safe format identification; do not bulk extract.
3. **Unknown:** What interactions and state transitions define a minimally playable game, and which need runtime observation? Static game-code analysis is blocked on a supported NE analyzer. Next: recover enough static visibility first; request bounded runtime testing only if static evidence cannot answer a specific question.
4. **Unknown:** Which macOS implementation/packaging strategy and dependencies are appropriate? Defer until executable/data formats and playable scope are better established.
5. **Policy established; rights unknown:** What local-only asset workflow and distributable content are permissible? Keep all supplied and derived proprietary payloads out of Git; do not assume redistribution rights.
6. **Unknown:** Minimum macOS version, Apple Silicon/Intel support, and accessibility/input expectations remain owner decisions.

Update this ledger after each investigation. Replace questions only when evidence resolves them; otherwise retain them with a status and next probe in `docs/plan.md`.
