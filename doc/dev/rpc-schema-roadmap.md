# RPC Schema Roadmap

This document tracks our plan for addressing [issue #29912](https://github.com/bitcoin/bitcoin/issues/29912): providing a formal, machine‑readable description of the JSON‑RPC API and tooling around it.

## Goals

1. **Expose machine-readable RPC metadata** so clients no longer have to duplicate parameter definitions, defaults, and result structures.
2. **Generate a formal specification** (OpenAPI or similar) from that metadata.
3. **Keep the spec accurate over time**, ideally with automation in CI and supporting documentation.

## Work Plan

### 1. Gather Existing Work (Week 1)
- Review #29912, Casey Rodarmor’s previous “schema” RPC, and nervana21’s `2025-07-schema-generation` branch.
- Audit current metadata sources (`RPCHelpMan`, `RPCArg`, help text) to understand what can be auto-derived.
- Identify fields that must be captured: method name, arguments (types, optional/defaults), result structure, error codes, examples.

### 2. Design the Schema Format (Week 1–2)
- Draft a structured JSON format returned by a new schema RPC (or an extended `help` call).
- Prototype the design against a few sample commands (`getblockchaininfo`, `sendrawtransaction`) to ensure nested/optional arguments are represented cleanly.
- Share the proposal in #29912 for maintainer and client-developer feedback.

### 3. Implement Core Extraction (Week 2–4)
- Add a helper (e.g. `RPCHelpMan::ToJson()`) that mirrors help metadata in structured form.
- Introduce an admin RPC (`schema` or `getrpcschema`) returning `{ "method": { "args": [...], "result": {...} } }`.
- Ensure deterministic ordering so schema diffs are stable.
- Guard access using normal RPC authentication; treat the new RPC as admin-only.

### 4. Testing & Validation (Week 3–4)
- Add a functional test that exercises the schema RPC and asserts key fields for several representative commands.
- Optionally add a lint/test that regenerates the schema and compares it against a recorded snapshot to ensure determinism.

### 5. OpenAPI (or Alternative) Export (Week 4–5)
- Build a generator (e.g. `contrib/devtools/rpc-openapi.py`) that converts schema JSON to OpenAPI v3 YAML.
- Decide whether the generated spec should land in-tree or be produced on-demand; document the regeneration command either way.
- Keep the generator lightweight and dependency-free where possible.

### 6. Documentation & CI Integration (Week 5–6)
- Add `doc/rpc-schema.md` describing the schema format, regeneration workflow, and examples.
- Update CI or lint checks so schema drift is detected automatically.
- Highlight in developer docs that RPC changes now require schema updates.

### 7. Coordination & Iteration (Ongoing)
- Engage with #29912 stakeholders (maflcko, casey, nervana21, client authors) for review and adoption.
- Break work into PR-sized chunks:
  1. Schema extraction RPC.
  2. Spec generator + docs.
  3. Optional follow-ups (language bindings, CI hooks).
- After the first milestone, collect feedback from downstream consumers and iterate on the schema structure to cover corner cases.

## Next Steps

1. Review historical comments and prototypes linked from #29912.
2. Build a minimal proof-of-concept schema RPC for discussion.
3. Iterate with stakeholders, expanding coverage and tooling once the core approach is accepted.

## Comment Review Notes (Issue #29912)

- **Initial problem statement (maflcko):** downstream RPC clients are hand-maintained; documentation sources like bitcoincore.org or third-party sites are incomplete, outdated, and not machine-readable.
- **Existing documentation:** auto-generated HTML/Markdown help pages exist but lack structured output; they are insufficient for codegen.
- **OpenAPI vs OpenRPC:** several contributors suggested OpenAPI or OpenRPC; uncertainty remains about how well these formats model JSON-RPC. Need to evaluate tooling and unit/enum support.
- **RPCHelpMan metadata:** laanwj confirmed it already captures most of what we need; idea is to expose the raw structure via RPC.
- **Prior prototypes:**
  - Casey Rodarmor built a `schema` RPC exporting an ad-hoc JSON description; later experimented with JSON Schema.
  - kilianmh produced an OpenRPC spec (manual extraction) up through v0.27.
  - laanwj explored JSON Schema exports, aligning with c-lightning’s approach (reuse tooling).
  - nervana21 consumes Casey’s schema to auto-generate Rust client bindings and maintains a branch with the RPC patch.
- **Design considerations discussed:**
  - JSON Schema vs custom format: JSON Schema offers standard tooling but may struggle with quirks (e.g., return type varying by verboseness).
  - Need to capture units (BTC vs sat, BTC/kvB) and optional/conditional fields precisely.
  - Support for multiple RPC aliases and test-only commands surfaced in discussions.
  - Potential to enforce schemas at test time by validating RPC calls against the generated spec.

These notes should guide follow-up design discussions and ensure we address previously identified pitfalls.



