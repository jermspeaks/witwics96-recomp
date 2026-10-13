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
- [ ] **In progress:** Identify the supplied release/build and map its executables, data files, and runtime dependencies. The supplied readme documents v3.0 (dated 1996-04-24); a 16-bit executable candidate is identified, but its code and the package's complete contents are not yet analyzed.
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

**Observed progress (2026-10-12):** The supplied `input/README.TXT` identifies itself as a v3.0 read-me dated 1996-04-24 and documents Windows 3.1/Windows 95 versions and QuickTime 2.1 usage. `input/SETUP.CFG` declares a 558,608-byte `CARMEN` source to be installed as `carmen.exe`; `input/SETUP16/CARMEN` matches the size and is identified as a Windows 3.x NE executable. Its SHA-256 is recorded in `docs/findings.md`. The setup mapping plus file identity makes it a strong game-executable candidate, not authenticated runtime evidence.

**Tool status:** The installed `reverse-engineer-anything` Claude skill was read (version 40). REA 6.4.0 is connected and reports Ghidra 12.1.4 on macOS arm64 as available; Hopper is unavailable. Ghidra `open_binary` rejects the NE candidate and root `SETUP.EXE`, so no code analysis has yet occurred. Use the actual session tool inventory, not assumptions in the skill, if the provider/session changes.

**Next actions:**
1. Determine whether the media also contains a distinct 32-bit game executable and map likely setup payloads using filenames and static metadata only; do not run setup.
2. Identify the `.MHK` resource format using a narrowly selected file and a supported static parser, if available. Do not bulk extract.
3. Check for an already available NE-capable analysis route before considering new tools. Do not install tools or execute the game/setup as an unrequested workaround.
4. Record any new artifact hashes, source paths, tool identity, provider limitations, and unanswered questions in `docs/findings.md`.

**Validation performed:** `git check-ignore -v input` confirms `/input/` is ignored; `git ls-files -- input` returned no tracked paths. `file`/`shasum` were used only for read-only type/identity checks. `open_binary` returned `target_unavailable` for the two NE targets. No extraction, modification, game/setup execution, build, or tests were performed.

**Exit criteria:** Findings document identifies supported artifact/build evidence, the game executable and key data-file identities, dependency evidence and limits, plus unresolved format/runtime questions. All payloads remain untracked. Current evidence is not sufficient to exit Phase 1.

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

See `docs/findings.md` for the detailed open-question list. Immediate blockers are incomplete build identity, unverified data formats/runtime behavior, and the lack of a connected NE-capable analyzer for the identified game-executable candidate (Ghidra rejects NE; Hopper is unavailable). The macOS target/implementation strategy is undecided. Legal rights to redistribute the game or its assets are not established; do not include those payloads.

## Session handoff checklist

At the end of each meaningful task:

1. Add dated findings with their source, evidence IDs (if any), confidence, and limitations.
2. Update the status/next action here; do not mark a phase done on filenames or assumptions alone.
3. Record exact validation commands/results, including failures or tests not run.
4. Confirm `input/` remains ignored and unmodified unless explicitly required.
