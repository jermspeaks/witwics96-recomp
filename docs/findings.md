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

## Analysis performed so far

Only repository/file metadata inventory was performed. No proprietary file contents were opened, extracted, decompiled, hashed, or executed. No REA analysis was run, and no test/build system exists in the initial repository evidence collected so far.

## Evidence and change log

| Date | Observation | Source / scope | Limits |
|---|---|---|---|
| 2026-10-12 | Initial repository and `input/` inventory; counts above | Project tree listing, `git status`, `find` file count and extension count, `du -sh input` | Metadata only; not evidence of content, runtime behavior, or rights |
| 2026-10-12 | Global Claude skill is installed at `~/.claude/skills/reverse-engineer-anything/SKILL.md` | Local skill file; version 40 from its front matter | Instructions only; does not establish that REA MCP tools/providers are connected |

## Open questions

1. What exact release/build is represented by the user-supplied copy, and what are its key executable and data-file identities?
2. What data/resource formats and asset dependencies are required to reproduce the playable loop?
3. What interactions and state transitions define a minimally playable version, and which require runtime observation to verify?
4. Which implementation/packaging strategy works on the target Mac and which dependencies does it require?
5. What local-only asset handling is needed, and what can be included in a distributable port without including copyrighted payloads?
6. What minimum macOS version, Apple Silicon/Intel support, and accessibility/input expectations should be targeted?

Update this ledger after each investigation. Replace questions only when evidence resolves them; otherwise retain them with a status and next probe in `docs/plan.md`.
