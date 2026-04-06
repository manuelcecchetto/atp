Agent Task Protocol (ATP)
=========================

**A Schema-First Standard for Autonomous AI Planning & Execution.**

ATP is a protocol that decouples **Planning** (The Architect) from **Execution** (The Agent). Instead of rigid chains or chaotic swarms, ATP uses a strictly typed **Directed Acyclic Graph (DAG)** stored in a single `.atp.json` file. This repo ships the schema and prompt templates; it does not bundle an official agent.

ATP v1.4 adds an adaptive judge model for bounded future replanning. Workers still execute one claimed node at a time, but a post-node judge may reshape only future work through MCP-owned patch operations when claimed-node count is zero.

🚀 The "Democratized" Workflow
------------------------------

ATP bridges the gap between **Business Vision** and **Technical Execution**.

### Phase 1: The Business Architect (Human + AI)

-   **User (Non-Technical):** "I want a Uber for Lawn Mowing."

-   **Prompt:** Uses `atp_business_architect_prompt.md`.

-   **Output:** A high-level graph containing business nodes like *"Driver Payout Calculation"* or *"GPS Tracking Rules"*.

### Phase 2: The Fractal Decomposition (Autonomous)

-   **The Agent:** Picks up the *"Driver Payout Calculation"* node.

-   **Complexity Check:** Realizes this is a high-level business concept, not a code task.

-   **Action:** Calls the **Decomposer**.

-   **Result:** Explodes the node into technical tasks: *"Create Payout Table SQL"*, *"Implement Stripe Connect API"*, *"Write Calculation Unit Tests"*.

### Phase 3: The Execution (Code)

-   **The Agent:** Picks up the atomic *"Create Payout Table SQL"* node.

-   **Action:** Writes the actual code.

🧠 Key Features
---------------

### 1\. Schema-Driven Interoperability

The core of ATP is `atp_schema.json`. Any tool that adheres to this schema can interact with the swarm.

-   **Validators** ensure plans are executable.
-   **Visualizers** render the JSON as an interactive map.
-   **Agents** (Python/Node/Go/anything) read the JSON to find work.

**Cross-Agent Rule:** Regardless of the LLM or runtime (Claude, Codex, etc.), every agent must read and write the *same* `.atp.json` file. All updates—decomposition, execution reports, replanning—must be committed back into that graph so other agents stay in sync.

### 2\. Fractal Planning (Recursive Decomposition)

ATP v1.3 introduces **Just-in-Time Planning**. Agents don't just execute; they evaluate complexity.

-   **The Fractal Rule:** If a task cannot be done in one file/function, it is not a task; it is a project.

-   **The Decomposer:** The agent pauses, calls the **Decomposer LLM**, and "explodes" the single node into a new sub-graph of atomic tasks, rewriting the JSON in real-time.

### 3\. Action/Report Separation

ATP distinguishes between **Artifacts** (the code/files created) and **Reports** (the knowledge transfer).

-   *Artifacts:* Live on disk.

-   *Report:* Lives in the graph. Only the report is passed to the next agent or adaptive judge to save context tokens.

### 4\. Adaptive Replanning (v1.4)

ATP v1.4 adds a bounded adaptive judge that operates between worker turns.

- The judge reads the full graph plus completed/failed node reports.
- Full graph reads expose `meta.graph_version`.
- Future patch apply uses `atp_apply_future_patch(...)`.
- Apply is legal only when claimed-node count is zero.
- The judge may mutate only future work: add nodes, update future instructions/context, rewire future dependencies, or close obsolete future nodes.

This keeps ATP mutable under discovery without dragging raw prior conversation into every future turn.

📂 Repository Structure
-----------------------

```
├── README.md
├── atp_schema.json                  # Strict JSON schema definition (v1.3 plan format; v1.4 is behavior/tooling additive)
├── examples/                        # Ready-to-use ATP graph samples
│   ├── non_technical_bike_share/    # Neighborhood bike share scenario
│   ├── non_technical_meal_kit/      # Meal kit subscription scenario
│   └── technical_oauth_feature/     # OAuth login upgrade scenario
├── docs/specs/
│   └── atp_v1_4.md                  # Adaptive judge and future-patch spec
└── prompts/
    ├── atp_business_architect_prompt.md  # For non-technical Product Managers
    ├── atp_decomposer_prompt.md          # For expanding nodes into sub-graphs
    ├── atp_executor_prompt.md            # For executing READY tasks and writing back to ATP
    └── atp_refiner_prompt.md             # For refining an existing graph

```

⚡ Quick Start
-------------

### 1\. Define the Business Logic

Copy `prompts/atp_business_architect_prompt.md` into an LLM.

> **User:** "Plan a subscription service for coffee beans."

Save the output as `plan.atp.json`.

### 2\. Bring Your Agent

Load that plan into your ATP-compatible agent/orchestrator. When a node is too abstract, use the Decomposer prompt to explode it into atomic tasks and update the graph. This repo does not include a reference executor.

All agents—even across different LLMs or runtimes—must persist their changes back into the same `.atp.json` file; there is no alternate state store.

For ATP v1.4 adaptive flows, the runner or orchestrator must treat raw conversation as audit/debug state only. Default cross-turn handoff should come from node reports, durable memory, and current graph state.

### 3\. Visualize

Load `plan.atp.json` into your preferred graph viewer or editor to watch your abstract business idea turn into concrete blue and green code nodes.

📦 Examples
-----------

Each scenario folder under `examples/` contains four artifacts that mirror the lifecycle:

- `user_query.txt` — the original ask.
- `atp_initial.atp.json` — first ATP graph from the business/user request.
- `atp_decomposed.atp.json` — technical decomposition for execution.
- `atp_replanned.atp.json` — revised plan after a user change request.

Included scenarios: neighborhood bike share, meal kit subscription, and OAuth login upgrade.

🔄 The Workflow Visualized
--------------------------

```
graph TD
    User[Business User] -->|Prompts| BA[Business Architect]
    BA -->|Generates| B(plan.atp.json)

    B --> C{Agent Watcher}
    C -->|Finds 'Business' Node| D[Complexity Check]

    D -->|Too Abstract?| E[Decomposer LLM]
    E -->|Translates to Tech Specs| B

    D -->|Atomic Code Task?| F[Execute Action]
    F -->|Creates| G[Artifacts / Files]
    F -->|Writes| H[Report]
    H --> B

```

📜 License
----------

ATP is an open standard. Feel free to build your own Agents, Visualizers, or Architects on top of the protocol.
