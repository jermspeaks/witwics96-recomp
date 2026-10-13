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
- [x] Identify the supplied release/build and map its executables, data files, and runtime dependencies (Phase 1, closed 2026-10-12; unknowns carried forward in `docs/findings.md`).
- [ ] **Next:** Describe the core gameplay loop and define a testable minimum playable scope.
- [ ] Choose and document the macOS implementation and local asset-loading strategy.
- [ ] Implement the smallest end-to-end playable slice.
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

### Phase 2 — Behavior and playable-scope definition (next)

**Questions:** What is the essential game loop, what state/resources drive it, and what can be reliably tested?

**Next actions:**
1. Open `/private/tmp/witwics96-verify/Carmen32.exe` in REA/Ghidra. Use `search_strings` + `xrefs` to find the code that uses `MOHAWK.INI`, `SYSTEM.MHK`, the `.MHK` names, the movie templates (`%sLOCN%d.MOV` and others), `Fame.Dat`/`User.Dat`, and `CDTInit`. Then decompile those procedures to recover startup, resource loading, and the case/travel/warrant/arrest loop. Record Evidence IDs.
2. Determine whether `Carmen32.cfg`/`mohawk.w32` config values (not yet extracted) are needed. Ask before extracting any further archive member.
3. Use the 16-bit NE (win16ne `-d`) only to cross-check behavior where the 32-bit build is ambiguous.
4. Extend the Mohawk structural check to the other `.MHK` files and list resource types per file (metadata only), so data needs can be mapped to gameplay.

Derive candidate behavior from static evidence; use runtime observation only for questions static evidence cannot answer and only with explicit, bounded scope. Record observed behavior separately from inferred behavior. Define a small playable milestone, controls, save/state needs, and asset-loading boundary.

**Exit criteria:** A written minimum-playable specification with traceable evidence and explicit unknowns, plus acceptance criteria that do not require bundling original assets.

### Phase 3 — macOS design and prototype (blocked on Phase 2)

**Questions:** Which technology and rendering/input/audio approach fit the verified formats and target macOS versions?

**Next actions:** Compare the simplest viable clean-room/native or compatibility-oriented options against established requirements. Build a thin prototype with synthetic/test-owned assets before implementing game-specific behavior. Decide how a user points the program at local game data without copying that data into the repository or package.

**Exit criteria:** Documented architecture decision, buildable prototype, and repeatable development setup.

### Phase 4 — Implementation, validation, and packaging (blocked on prior phases)

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
