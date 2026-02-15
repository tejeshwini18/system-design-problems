# System Design Document Template

Every design in this folder follows this process and structure.

---

## System Design Process

### Step 1: Clarify Requirements
- **Functional:** What the system must do (features, use cases).
- **Non-functional:** Latency, availability, scale, consistency.
- **Constraints & assumptions:** Tech stack, existing systems, scope.

### Step 2: High-Level Design
- **Components:** Main services/stores and their responsibilities.
- **Interfaces:** How components talk (REST, events, etc.).
- **Data flow:** Step-by-step for main flows (e.g. create, read).
- **Flow diagram:** Architecture and/or sequence (Mermaid + Eraser).

### Step 3: Detailed Design
- **Database:** SQL vs NoSQL choice; schema or key model.
- **API design:** Full list of endpoints (method, path, purpose).
- **Key algorithms:** Critical logic (e.g. short code generation, rate limit algorithm).

### Step 4: Scale & Optimize
- **Load balancing:** How traffic is distributed.
- **Sharding:** How data is partitioned.
- **Caching:** What is cached, where, invalidation.

---

## Document Structure

### HLD.md
1. Title & overview
2. **System Design Process** (Steps 1–4 as above)
3. **High-Level Architecture** — Component diagram (boxes: clients, services, data stores; connections). Use Mermaid `flowchart` with subgraphs + optional Eraser block.
4. **Flow Diagram** — Main flow (sequence of steps or message flow). Use Mermaid `sequenceDiagram` or `flowchart` for process flow.
5. API endpoints table (required)
6. Capacity estimation (if applicable)
7. Trade-offs table

### LLD.md
1. Note that LLD elaborates Step 3 (and supports Step 2)
2. **API endpoints (complete)** — table with method, path, request/response
3. **High-level architecture** (optional, if different from HLD) — component view
4. **Flow diagram** — sequence or flowchart for main flow(s)
5. Database schema / data model
5. Key classes or modules
6. Algorithms / logic
7. Error handling

---

## Diagrams

- **Mermaid:** Renders in GitHub, VS Code, many IDEs. Use `flowchart`, `sequenceDiagram`, etc. Every HLD and (where useful) LLD includes a Mermaid flow or sequence diagram.
- **Eraser:** Paste the code block into [Eraser.io](https://app.eraser.io) → New diagram → Diagram as code. Use Eraser DSL for architecture and sequence. Many HLDs include an Eraser-style code block you can paste into Eraser to edit or export.
