# Project Plan

Last updated: 2026-10-12

This is the handoff plan. Keep statuses current so a new agent can resume without repeating work. A completed phase means its stated outputs exist and its validation is recorded; it does not mean the whole port is complete.

## Non-negotiable input policy

Bring your own legally obtained game copy. `input/` is intentionally ignored by Git. Do not commit the original game, extracted proprietary assets, or other copyrighted payloads. Keep any analysis local and read-only unless the owner explicitly authorizes a different operation.

## Goal

Create a playable macOS port of the 1990s Windows game identified by the project owner, while keeping the repository free of the original game and proprietary payloads.

## Current status

- [x] Establish repository agent guidance and the local game-copy ignore rule.
- [x] Record the initial metadata-only inventory and open questions.
- [ ] **In progress:** Identify the supplied release/build and map its executables, data files, and runtime dependencies. The supplied read-me documents v3.0 (dated 1996-04-24); 16-bit setup files identify the NE game candidate and the 32-bit installer declares `Carmen32.exe` inside `ARCHIVE.C2K`. The 32-bit executable and archive contents have not been inspected.
- [ ] Describe the core gameplay loop and define a testable minimum playable scope.
- [ ] Choose and document the macOS implementation and local asset-loading strategy.
- [ ] Implement the smallest end-to-end playable slice.
- [ ] Add automated tests/fixtures that do not contain copyrighted game payloads.
- [ ] Verify build/run behavior on the declared macOS target(s), documenting environment and gaps.
- [ ] Document setup, bring-your-own-copy requirements, known limitations, and packaging policy.

## Work sequence and handoff

### Phase 0 — Repository and evidence hygiene (complete)

**Outputs:** `AGENTS.md`, `.gitignore`, this plan, and `docs/findings.md`.

**Validation:** Confirm `input/` is ignored and that no game files are staged/tracked. At this stage no source code, tests, build system, or implementation approach has been established.

### Phase 1 — Static artifact inventory (in progress)

**Questions:** What exact build is represented? Which files are the game executable, resources, data, codecs, or installers? What host/runtime assumptions are supported by static evidence?

**Observed progress (2026-10-12):** The supplied `input/README.TXT` identifies itself as a v3.0 read-me dated 1996-04-24 and documents Windows 3.1/Windows 95 versions and QuickTime 2.1 usage. `input/SETUP16/CARMEN.INF` maps a 558,608-byte `CARMEN` source to installed `carmen.exe`; `input/SETUP16/CARMEN.BDF` also maps the `CARMEN` entry to `carmen.exe`, and `input/SETUP16/CARMEN` matches the size and is identified as a Windows 3.x NE executable. A read-only header/table inspection found 29 in-range segment records and five module-reference names (`GDI`, `KERNEL`, `MMSYSTEM`, `USER`, `WIN87EM`); see `docs/findings.md` for scope and limitations. These setup declarations plus file identity make it a strong 16-bit game-executable candidate, not authenticated runtime evidence. `input/SETUP.CFG` selects 16-bit and 32-bit installer programs; `input/SETUP32/SETUP32.INI` declares installed `Carmen32.exe` from `input/SETUP32/ARCHIVE/ARCHIVE.C2K`, but no loose executable was found and the archive has not been listed or extracted. Its first eight bytes match the documented InstallShield Z v3 signature; list-only open-source tools are documented but not available locally. One representative `input/DATABASE.MHK` was structurally checked read-only against the upstream ScummVM Mohawk parser layout: version 1.0, 203 file-table entries, six type entries, 203 resource entries, and valid in-file table/data bounds. This does not interpret resources or establish game behavior.

**Tool status:** The global `reverse-engineer-anything` skill file (version 40) was read directly; the skill registry lookup is unavailable in this session, while REA tools are connected. REA 6.4.0 reports Ghidra 12.1.4 on macOS arm64 as available; Hopper is unavailable. Ghidra `open_binary` rejects the NE candidate and root `SETUP.EXE`; Apple LLVM `objdump` 21.0.0 also rejected the NE candidate, and `xcrun` could not locate `llvm-readobj`. `bsdtar` cannot list the InstallShield Z archive. The bounded follow-up found no `unshieldv3`, `idecomp`, `isextract`, or `scummvm` command on `PATH`. Open-source InstallShield Z v3 list modes are documented by [unshieldv3](https://github.com/wfr/unshieldv3) and [idecomp](https://github.com/lephilousophe/idecomp), but neither tool is installed or tested. The Unarchiver GUI is installed; its inspected `Contents/MacOS` contains only the GUI executable and its bundle does not declare `.C2K` or `.MHK` document types; it was not launched. No NE instruction analysis or REA function Evidence has been produced. External NE leads are [win16ne](https://github.com/qnighy/win16ne) and [Semblance](https://gitlab.winehq.org/zfigura/semblance); neither is installed or tested. Hopper’s official FAQ documents Mach-O, PE, and ELF, not NE, so installing it is not currently recommended for this blocker. Use actual connected tool inventory if the provider/session changes.

**Next actions:**
1. Ask the owner before acquiring/building a list-only InstallShield Z v3 tool (documented candidates: `unshieldv3` or `idecomp`). If authorized, first list archive metadata/members without extraction; do not run setup or place extracted proprietary payloads in Git.
2. If resource-index detail is needed for scope, extend the read-only structural check to one additional `.MHK` only after identifying a concrete question; do not bulk extract or print payloads. The current check structurally validates only `DATABASE.MHK`.
3. Ask the owner before acquiring/building an NE-capable static-analysis candidate (`win16ne` or Semblance). Choose based on source/build requirements and desired static output; do not install Hopper as the presumed NE solution. Do not execute the game/setup without explicit runtime-test authorization.
4. Record exact hashes, source paths, tool identity, provider limitations, and unanswered questions in `docs/findings.md`; keep Phase 1 open until its exit criteria are met.

**Validation performed:** `git check-ignore -v input` confirms `/input/` is ignored; `git ls-files -- input` returned no tracked paths. Read-only `find`/`file`, `od`, `strings`, and `shasum` inventory checks remain as above. The InstallShield archive header matches the documented v3 signature, but `bsdtar -tf` fails with “Unrecognized archive format”; no list-capable extractor is installed. The Unarchiver app metadata was inspected without launching it. A bounded Python standard-library structural check of `DATABASE.MHK`, based on upstream ScummVM's `resource.cpp` field layout, validated version/header sizes, 203 file-table records, six type entries, 203 resource entries, and referenced bounds; an initial harness assumption of 12-byte file-table rows failed its bounds check, then was corrected to the 10-byte row layout in upstream source and passed. This check read no extracted payloads. Official Hopper FAQ, InstallShield Z v3 tool documentation, and NE candidate documentation were reviewed 2026-10-12. `open_binary` returned `target_unavailable` for the two NE targets; Apple LLVM `objdump` did not recognize the NE candidate. No extraction, modification, game/setup execution, build, or tests were performed.

**Exit criteria:** Findings document identifies supported artifact/build evidence, the game executable and key data-file identities, dependency evidence and limits, plus unresolved format/runtime questions. The 32-bit executable is currently only an installer declaration within an uninspected archive. All payloads remain untracked. Current evidence is not sufficient to exit Phase 1.

### Phase 2 — Behavior and playable-scope definition (blocked on Phase 1)

**Questions:** What is the essential game loop, what state/resources drive it, and what can be reliably tested?

**Next actions:** Derive candidate behavior from static evidence; use runtime observation only for questions static evidence cannot answer and only with explicit, bounded scope. Record observed behavior separately from inferred behavior. Define a small playable milestone, controls, save/state needs, and asset-loading boundary.

**Exit criteria:** A written minimum-playable specification with traceable evidence and explicit unknowns, plus acceptance criteria that do not require bundling original assets.

### Phase 3 — macOS design and prototype (blocked on Phase 2)

**Questions:** Which technology and rendering/input/audio approach fit the verified formats and target macOS versions?

**Next actions:** Compare the simplest viable clean-room/native or compatibility-oriented options against established requirements. Build a thin prototype with synthetic/test-owned assets before implementing game-specific behavior. Decide how a user points the program at local game data without copying that data into the repository or package.

**Exit criteria:** Documented architecture decision, buildable prototype, and repeatable development setup.

### Phase 4 — Implementation, validation, and packaging (blocked on prior phases)

Implement in small vertical slices. Keep tests and fixtures free of copyrighted payloads. Validate functionality on declared macOS targets, include failure handling for missing/incompatible user-provided data, and ensure packaging does not silently include ignored local files.

**Exit criteria:** Reproducible build and test instructions; tested playable loop; clean package-content inspection; documented limitations and user-supplied-copy setup.

## Risks and unresolved decisions

See `docs/findings.md` for the detailed open-question list. Immediate blockers are incomplete build identity, unverified archive/resource formats and runtime behavior, and the lack of a connected NE-capable analyzer for the identified 16-bit game-executable candidate (Ghidra and the checked `objdump` route reject it; Hopper is unavailable and its FAQ does not document NE support). The declared 32-bit `Carmen32.exe` remains unverified inside an InstallShield Z archive that the available `bsdtar` cannot list. The macOS target/implementation strategy is undecided. Legal rights to redistribute the game or its assets are not established; do not include those payloads.

## Session handoff checklist

At the end of each meaningful task:

1. Add dated findings with their source, evidence IDs (if any), confidence, and limitations.
2. Update the status/next action here; do not mark a phase done on filenames or assumptions alone.
3. Record exact validation commands/results, including failures or tests not run.
4. Confirm `input/` remains ignored and unmodified unless explicitly required.
