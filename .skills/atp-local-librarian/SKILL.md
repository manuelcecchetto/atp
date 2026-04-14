---
name: atp-local-librarian
description: Operate ATP `.atp.json` plans locally without MCP. Use when Codex or another ATP worker needs the ATP librarian behaviors through direct filesystem commands instead of MCP tools: claim the next READY node, inspect graph state, complete or fail a node with a handoff report, decompose an oversized node into subtasks, read local or full graph views, or apply ATP v1.4 future patches during adaptive replanning.
---

# ATP Local Librarian

Use the bundled CLI to mutate ATP plans safely under a local file lock. Treat it as the file-based replacement for the ATP librarian MCP surface.

## Workflow

1. Read [references/command-patterns.md](references/command-patterns.md) first.
2. Resolve the absolute `plan_path` you will operate on.
3. Never edit the `.atp.json` file directly for normal ATP graph mutations. Use the bundled CLI.
4. For normal execution, loop:
   - inspect with `atp-status-summary` or `atp-read-graph`
   - claim with `atp-claim-task`
   - execute or decompose
   - complete with `atp-complete-task`
   - claim again until no READY work remains or a real blocker stops progress
5. Use `atp-apply-future-patch` only for bounded ATP v1.4 future-graph updates after reading the full graph and capturing `meta.graph_version`.

## Command Surface

The CLI lives at [scripts/atp_local_librarian.py](scripts/atp_local_librarian.py).

It exposes one subcommand per former MCP librarian action:

- `atp-claim-task`: recover stale claims, refresh READY nodes, and claim the highest-priority READY node for an `agent_id`
- `atp-complete-task`: mark a node `DONE` or `FAILED`, store the report and artifacts, and unlock downstream work
- `atp-decompose-task`: turn a task into a `SCOPE` and graft a child DAG under it
- `atp-read-graph`: print the full graph JSON with runtime `meta.graph_version`, or a local neighborhood view
- `atp-status-summary`: print the read-only status dashboard that replaces the MCP status resource
- `atp-apply-future-patch`: apply a bounded ATP v1.4 future patch when claimed-node count is zero and the graph version matches

Prefer the explicit `atp-*` subcommand names to keep muscle memory aligned with the former MCP tools.

## Operating Rules

- Always pass an absolute `plan_path`.
- Keep a stable `agent_id`. Default to `CODEX` unless the user or orchestrator specifies another value.
- Write completion reports for downstream workers, not just for the current turn. Include the outcome, key facts learned, files touched, verification, risks, and the recommended next step when useful.
- Prefer `atp-read-graph --view-mode local --node-id <node>` over broad full-graph reads when you only need nearby context.
- Decompose only when the node contains multiple independent outcomes or materially different verification paths.
- Use temp files for `--report-file`, `--subtasks-file`, and `--patch-file` instead of trying to squeeze large JSON or multiline reports into one shell argument.
- Treat `SCOPE` nodes as containers. Do not manually complete or decompose a node that is already a `SCOPE`.

## ATP Protocol Guidance

When this skill is used inside the `atp-protocol` repository, treat these files as canonical guidance:

- [../../README.md](../../README.md)
- [../../atp_schema.json](../../atp_schema.json)
- [../../prompts/atp_decomposer_prompt.md](../../prompts/atp_decomposer_prompt.md)
- [../../prompts/atp_executor_prompt.md](../../prompts/atp_executor_prompt.md)
- [../../prompts/atp_refiner_prompt.md](../../prompts/atp_refiner_prompt.md)

The CLI performs the graph mutation; the protocol docs define what a valid ATP plan and worker handoff should look like.

## Resources

- [references/command-patterns.md](references/command-patterns.md): command shapes, temp-file patterns, and JSON payload examples for decomposition and future patches.
- [scripts/atp_local_librarian.py](scripts/atp_local_librarian.py): local no-MCP ATP librarian CLI.
