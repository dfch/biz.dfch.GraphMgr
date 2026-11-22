# biz.dfch.GraphMgr

A manager for a generic, typed node/edge graph. The data is stored as git-trackable text files. Access goes through one core library (ABI), with a REST API and an MCP server as thin wrappers.

## Repository layout

- `docs/`: specification documents (managed with specmgr): problem, goals, decisions, requirements, risks, use cases, verification.
- `diagrams/`: PlantUML use-case, sequence and reference diagrams.
- `docs/history/`: the original design discussion (2026-10-10/11), held under the earlier working name "RelMgr". It is a historical record and is left unchanged.
- `src/biz_dfch_graphmgr/`: the Python package (not implemented yet).
