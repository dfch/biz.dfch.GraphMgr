# AGENTS.md

## Project state

- Early stage: `src/biz_dfch_graphmgr/` is a hello-world placeholder; the system is **not implemented yet**. The design is fully specified in `docs/` — read those before implementing.
- Python >=3.13 (`.python-version`), uv toolchain (`uv_build` backend). No dependencies, no `uv.lock`, no tests, no lint/typecheck config, no CI. Do not assume a test runner or formatter exists.
- Working branch is `dev` (`origin/HEAD`); `main` also exists. Remote: `github.com/dfch/biz.dfch.GraphMgr`.

## Commands

- Run the entrypoint: `uv run biz-dfch-graphmgr` (console script → `biz_dfch_graphmgr:main`).
- Add dependencies with `uv add <pkg>`; commit the resulting `uv.lock` (it is not gitignored).
- `.venv/` is uv-managed (CPython 3.13) and gitignored — never edit it directly.

## docs/ — specmgr-managed specs (source of truth)

- Files are `<type>-<uuid>-<slug>.md` under `docs/{prb,gol,dec,req,rsk,uc,vcr,qa}/`. Change them via the specmgr MCP tools (`create_*`, `update`, `edit`, `set_status`, ...), not by hand: ids, frontmatter, and `updated` timestamps are tool-managed.
- `update`/`edit` replace body only (no frontmatter). A document that fails to parse cannot be fixed with the generic `update` tool — use the `repair` skill / doc-repairer agent instead.
- Accepted decisions to follow when implementing: `dec-2c4a32c5` (bespoke system following the specmgr pattern; type schemas as data-driven JSON Schema files under `schemas/node/` and `schemas/edge/` — those directories do not exist yet), `dec-a6235507` (storage: one file per entity, 256 id-prefix shard dirs, nodes as folders with attachments), `dec-4a3b078a` (derived sqlite index, full rebuild on startup), `dec-3cccd4cb` (git-independent filesystem manifest + hybrid watch/poll change detection).
- `docs/history/` is a verbatim historical record from the old "RelMgr" working name — do not modify it.
- `session-first-conversation-ses_ed8c.md` in the repo root is a local-only raw transcript, excluded by `.gitignore` — never commit it.
- Language variants exist under `docs/req/{de,fr}/` (specmgr i18n). Both currently fail to parse: the `de` variant uses a translated RFC-level word (`Level` must stay English `MUST|SHOULD|MUST NOT|SHOULD NOT|MAY` even in translations), and `fr` has no packaged heading-mapping table, so fr documents cannot be read or written until one exists. Check variant status with `list_languages` / `check_translation_drift` before touching them.

## diagrams/

- PlantUML sources: `uc/*.puml` (usecase + sequence per UC), `refs/*.puml`, with committed `.svg`/`.png` renders. Keep renders in sync when a `.puml` changes.

## Planned architecture (per accepted decisions)

- One core library is the single write path (ABI); the REST API and MCP server are thin, logic-free wrappers over it.
- Every API write: in-process type-level lock → validation → one git commit per write.
- Edges are first-class stored entities; referential integrity is enforced at edge write time (a write is rejected if `src`/`dst` does not resolve to an existing node).
