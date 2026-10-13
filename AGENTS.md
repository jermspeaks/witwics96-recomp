# Agent Instructions

## Project goal and boundaries

This repository is for exploring a playable macOS port of the 1990s Windows game *Where in the World Is Carmen Sandiego?* The title and goal come from the project owner; do not present them as independently verified artifact facts until supported by evidence.

- Bring your own legally obtained game copy. The root `input/` directory is intentionally Git-ignored. Never stage or commit the original game, extracted proprietary assets, or other copyrighted payloads.
- Do not move, rewrite, or clean up files under `input/` unless the owner specifically requests it. Prefer read-only inspection and keep analysis outputs outside Git or in an explicitly ignored working location.
- Treat filenames, extensions, strings, and static-analysis results as clues, not proof of runtime behavior, file format, licensing, or compatibility.
- Do not execute the original game or extracted code unless a task explicitly calls for runtime testing. Describe any execution risks and use the narrowest appropriate environment and scope.
- Keep implementation, findings, and plans grounded in observed evidence. Separate **observed**, **inferred**, and **unknown** statements.

## Reverse-engineering workflow

When a task depends on the supplied game files, use the globally installed Claude skill `reverse-engineer-anything` when available. Read its `SKILL.md` and only the relevant reference guide(s); follow the connected tool inventory and schemas rather than assuming a particular REA server or provider is available. The installed skill is guidance, not proof that analysis tools are connected. If unavailable, continue with safe repository work and document the blocker; do not invent findings.

For each investigation:

1. State the question and the evidence needed before broadening analysis.
2. Start with inventories and static inspection; inspect only the relevant artifacts.
3. Preserve artifact identity and evidence references when available. Do not claim that static analysis proves execution.
4. Update `docs/findings.md` with dated, source-linked observations and explicit limitations/unknowns.
5. Update `docs/plan.md` so another agent can resume from clear statuses and next actions.
6. Do not call work complete while required questions remain unresolved; mark them as such.

## Repository practices

- Read the relevant source and tests before changing implementation. Make focused changes consistent with existing conventions.
- Prefer clean-room implementation of behavior over copying original code or proprietary payloads into this repository.
- Keep docs concise and actionable. Include commands and validation results only when actually run; distinguish not-run from passed.
- Do not commit, create branches, or change project scope unless explicitly requested.
