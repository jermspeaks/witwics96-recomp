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
- [ ] **In progress:** Identify the supplied release/build and map its executables, data files, and runtime dependencies. The supplied read-me documents v3.0 (dated 1996-04-24); 16-bit setup files identify the NE game candidate and the 32-bit installer declares `Carmen32.exe` inside `ARCHIVE.C2K`. The archive member directory has been listed, but the executable bytes/type and installed-file mapping remain unverified.
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

**Observed progress (2026-10-12):** The supplied `input/README.TXT` identifies itself as a v3.0 read-me dated 1996-04-24 and documents Windows 3.1/Windows 95 versions and QuickTime 2.1 usage. `input/SETUP16/CARMEN.INF` and `CARMEN.BDF` map the 558,608-byte `input/SETUP16/CARMEN` file to installed `carmen.exe`; the file is identified as a Windows 3.x NE executable. A read-only header/table inspection found 29 in-range segment records and five module-reference names (`GDI`, `KERNEL`, `MMSYSTEM`, `USER`, `WIN87EM`); see `docs/findings.md` for scope and limitations. These setup declarations plus file identity make it a strong 16-bit game-executable candidate, not authenticated runtime evidence. `input/SETUP.CFG` selects 16-bit and 32-bit installer programs; `input/SETUP32/SETUP32.INI` declares installed `Carmen32.exe` from `ARCHIVE.C2K`. A built, reviewed `unshieldv3` list-only command found 20 archive entries, including `Carmen32.exe` at 425,504 bytes (directory metadata only); no payload was extracted. One representative `input/DATABASE.MHK` was structurally checked read-only against the upstream ScummVM Mohawk parser layout: version 1.0, 203 file-table entries, six type entries, 203 resource entries, and valid in-file table/data bounds. This does not interpret resources or establish game behavior.

**Tool status:** The global `reverse-engineer-anything` skill file (version 40) was read directly; its registry lookup is unavailable in this session, while REA tools are connected. REA 6.4.0 reports Ghidra 12.1.4 on macOS arm64 as available; Hopper is unavailable. Ghidra `open_binary` and Apple LLVM `objdump` reject the NE candidate. `bsdtar` cannot list the InstallShield Z archive. `unshieldv3` was fetched at commit `541a16ca7b38b2fdf466274b4bf53d18528d743a`, source-reviewed, and built in `/private/tmp` (binary SHA-256 `23b83babcc398c6206ec15ac52d1446779a5d3f12987222b8d80618c6b6918f3`); its read-only `info` and `list -v` operations worked on `ARCHIVE.C2K`. `win16ne` source was fetched at commit `4e5700fb97ba9f0338b49c1031834ce42177f265` and reviewed, but `rustc` and `cargo` are absent, so it was not built or run. Its locked crate dependencies have not been downloaded. Hopper’s FAQ documents Mach-O, PE, and ELF, not NE; installing Hopper is not currently recommended for this blocker.

**Next actions:**
1. The InstallShield Z archive has been listed without extraction. Ask before extracting the single `Carmen32.exe` member to a temporary analysis directory; inspect it statically only if authorized. Do not run setup or include proprietary payloads in Git.
2. Ask whether to install a temporary Rust toolchain and download `win16ne`'s locked Cargo dependencies into `/tmp`; if approved, build and run it only against `input/SETUP16/CARMEN` for static parsing/disassembly. No global PATH/config changes; keep tool files outside the repository.
3. If resource-index detail is needed for scope, extend the read-only structural check to one additional `.MHK` only after identifying a concrete question; do not bulk extract or print payloads. The current check structurally validates only `DATABASE.MHK`.
4. Record exact hashes, source paths, tool identity, provider limitations, and unanswered questions in `docs/findings.md`; keep Phase 1 open until its exit criteria are met.

**Validation performed:** `git check-ignore -v input` confirms `/input/` is ignored; `git ls-files -- input` returned no tracked paths. Read-only `find`/`file`, `od`, `strings`, and `shasum` checks remain as above. The InstallShield archive header matches the documented v3 signature; `bsdtar -tf` fails, while the source-reviewed `unshieldv3 info` and `list -v` successfully listed 20 entries without decompression or extraction. Its CMake build succeeded with AppleClang 21; a CMake deprecation warning and non-fatal `xcrun` cache permission warning appeared. The Mohawk structural check of `DATABASE.MHK` passed after correcting the harness's first mistaken 12-byte table-entry assumption to the 10-byte layout documented in upstream ScummVM. `win16ne` source and locked manifest were reviewed but could not be built because Rust/Cargo are unavailable. The three input artifact hashes remain unchanged. No game/setup execution, archive extraction, or tests for a port were performed.

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

See `docs/findings.md` for the detailed open-question list. Immediate blockers are incomplete build identity, unverified resource formats and runtime behavior, and the lack of a runnable local NE analyzer for the identified 16-bit game-executable candidate (Ghidra and the checked `objdump` route reject it; Hopper is unavailable and its FAQ does not document NE support). The InstallShield Z member directory is listed, but the declared 32-bit `Carmen32.exe` bytes/type remain unverified because no member has been extracted. The macOS target/implementation strategy is undecided. Legal rights to redistribute the game or its assets are not established; do not include those payloads.

## Session handoff checklist

At the end of each meaningful task:

1. Add dated findings with their source, evidence IDs (if any), confidence, and limitations.
2. Update the status/next action here; do not mark a phase done on filenames or assumptions alone.
3. Record exact validation commands/results, including failures or tests not run.
4. Confirm `input/` remains ignored and unmodified unless explicitly required.
