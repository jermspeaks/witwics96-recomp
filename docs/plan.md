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

**Observed progress (2026-10-12):** The supplied `input/README.TXT` identifies itself as a v3.0 read-me dated 1996-04-24 and documents Windows 3.1/Windows 95 versions and QuickTime 2.1 usage. `input/SETUP16/CARMEN.INF` maps a 558,608-byte `CARMEN` source to installed `carmen.exe`; `input/SETUP16/CARMEN.BDF` also maps the `CARMEN` entry to `carmen.exe`, and `input/SETUP16/CARMEN` matches the size and is identified as a Windows 3.x NE executable. A read-only header/table inspection found 29 in-range segment records and five module-reference names (`GDI`, `KERNEL`, `MMSYSTEM`, `USER`, `WIN87EM`); see `docs/findings.md` for the scope and limitations. Its SHA-256 is recorded there. These setup declarations plus file identity make it a strong 16-bit game-executable candidate, not authenticated runtime evidence. Separately, `input/SETUP.CFG` selects 16-bit and 32-bit installer programs; `input/SETUP32/SETUP32.INI` declares installed `Carmen32.exe` from `input/SETUP32/ARCHIVE/ARCHIVE.C2K`, but no loose `Carmen32.exe` was found and the archive member table has not been enumerated or extracted.

**Tool status:** The installed `reverse-engineer-anything` Claude skill was read (version 40). REA 6.4.0 is connected and reports Ghidra 12.1.4 on macOS arm64 as available; Hopper is unavailable. Ghidra `open_binary` rejects the NE candidate and root `SETUP.EXE`. Apple LLVM `objdump` 21.0.0 also rejected the NE candidate, and `xcrun` could not locate `llvm-readobj`. Follow-up bounded checks found that `bsdtar` recognizes `ARCHIVE.C2K` as InstallShield Z via `file` but cannot list it; The Unarchiver GUI is installed; its inspected `Contents/MacOS` contains only the GUI executable and its bundle does not declare `.C2K` or `.MHK` document types. No local NE analyzer, InstallShield Z lister, or Mohawk reader was identified. Hopper’s official FAQ documents Mach-O, PE, and ELF inputs, not NE, so installing it is not currently a recommended way to unblock analysis. An external `win16ne` parser/disassembler is only a research lead; it is not installed or validated here. A read-only standard-library inspection has parsed the candidate’s NE header, segment table, and five module-reference names, but no instruction decompilation, call graph, or REA function Evidence has been produced. Use the actual session tool inventory, not assumptions in the skill, if the provider/session changes.

**Next actions:**
1. Investigate a dedicated, read-only InstallShield Z archive lister. `bsdtar` was tried and failed; The Unarchiver’s declared types do not include `.C2K` (though a generic handler is declared). If no approved/local parser is available, ask before acquiring one. Do not run setup or place extracted proprietary payloads in Git.
2. Seek an existing or documented Mohawk/MHWK parser; bounded checks found no local `scummvm` command or obvious parser. Validate one representative `.MHK` archive structurally only if a suitable read-only route is available; do not bulk extract.
3. Resolve the NE analysis blocker with a route that explicitly supports Windows 3.x NE. Ghidra and Apple LLVM `objdump` rejected the candidate. Hopper is not installed, and its official FAQ does not document NE support; do not ask the owner to install Hopper as the presumed solution. The external `win16ne` project is a lead to evaluate, not a verified or installed tool. Do not install tools or execute the game/setup without an explicit decision.
4. Record new hashes, source paths, tool identity, provider limitations, and unanswered questions in `docs/findings.md`.

**Validation performed:** `git check-ignore -v input` confirms `/input/` is ignored; `git ls-files -- input` returned no tracked paths. `find`/`file`, `od`, `strings`, and `shasum` were used for read-only inventory, header, static-string, and identity checks. Follow-up: `file` identifies `ARCHIVE.C2K` as InstallShield Z; `bsdtar -tf` fails with “Unrecognized archive format”; bounded `command -v`, Homebrew Cellar, app-bundle, and Python-module checks did not identify a usable InstallShield/Mohawk/NE tool. The Unarchiver app metadata was inspected without launching it. Official Hopper FAQ reviewed 2026-10-12: documented formats are Mach-O, PE32/PE32+/64, and ELF, not NE. `open_binary` returned `target_unavailable` for the two NE targets; Apple LLVM `objdump` did not recognize the NE candidate. A bounded Python standard-library parse inspected NE header/segment/import metadata only; no instruction decompilation or runtime analysis was performed. No extraction, modification, game/setup execution, build, or tests were performed.

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
