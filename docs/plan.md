# Project Plan

Last updated: 2026-10-12

This is the handoff plan. Keep statuses current so a new agent can resume without repeating work. A completed phase means its stated outputs exist and its validation is recorded; it does not mean the whole port is complete.

## Non-negotiable input policy

Bring your own legally obtained game copy. `input/` is intentionally ignored by Git. Do not commit the original game, extracted proprietary assets, or other copyrighted payloads. Keep any analysis local and read-only unless the owner explicitly authorizes a different operation.

## Goal

Create a playable macOS port of the 1990s Windows game identified by the project owner, while keeping the repository free of the original game and proprietary payloads.

## Current status

- [x] Establish repository agent guidance and the local game-copy ignore rule.
- [x] Record an initial metadata-only inventory and open questions.
- [ ] Identify the supplied release/build and map its executables, data files, and runtime dependencies.
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

### Phase 1 — Static artifact inventory (not started)

**Questions:** What is the exact build? Which files are executable entry points, resources, data, codecs, or installers? What host/runtime assumptions can be evidenced statically?

**Next actions:** First confirm the local game copy remains ignored. Use the installed `reverse-engineer-anything` skill for relevant executable/archive analysis when tools are available. Begin with artifact inventory and binary metadata; preserve hashes/Evidence IDs and avoid extracting or executing broadly. Inspect only targeted files after the inventory identifies them.

**Exit criteria:** Findings document identifies artifact/build evidence, entry points, key dependencies, confidence and unresolved format/runtime questions. All payloads remain untracked.

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

See `docs/findings.md` for the detailed open-question list. Immediate blockers are unknown build identity, unverified file formats/runtime behavior, and undecided macOS target/implementation strategy. Legal rights to redistribute the game or its assets are not established; do not include those payloads.

## Session handoff checklist

At the end of each meaningful task:

1. Add dated findings with their source, evidence IDs (if any), confidence, and limitations.
2. Update the status/next action here; do not mark a phase done on filenames or assumptions alone.
3. Record exact validation commands/results, including failures or tests not run.
4. Confirm `input/` remains ignored and unmodified unless explicitly required.
