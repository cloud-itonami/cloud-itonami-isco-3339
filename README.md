# cloud-itonami-isco-3339

Open Occupation Blueprint for **ISCO-08 3339**: Business Services Agents Not Elsewhere Classified.

This repository designs a forkable OSS business for an independent business-services agent: a document-handling robot performs client-record scanning and filing under a governor-gated actor, so the practice keeps its own transaction records instead of renting a closed brokerage SaaS.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a document-handling robot performs client-record scanning and filing for agency transactions under an actor that proposes
actions and an independent **Business Services Governor** that gates them. The governor never
dispatches hardware itself; `:high`/`:safety-critical` actions (such as
handling client funds, or brokering contracts above a threshold value) require human sign-off.

A live sample of the operator console (robotics safety console, shared template) is rendered in [docs/samples/operator-console.html](docs/samples/operator-console.html) — pure-data HTML output of `kotoba.robotics.ui`.

## Core Contract

```text
client request + service scope + fee agreement
        |
        v
Agency Advisor -> Business Services Governor -> match/coordinate, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses, suppress
an operating record, or disclose sensitive data without governor approval and
audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `3339`). Required capabilities:

- :robotics
- :identity
- :forms
- :dmn
- :bpmn
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## Reference implementation (`:maturity :implemented`)

Full itonami Actor pattern (per ADR-2607011000 / CLAUDE.md's Actors
section, alongside `cloud-itonami-isco-6130`, `-8160`, `-2166`, `-2641`,
`-2651`, `-2652`, `-2654`, `-1219`, `-1223`, `-1330`, `-1341`, `-1349`,
`-1412`, `-1439`, `-2144`, `-2320`, `-2411`, `-2422`, `-2431`, `-2621`,
`-2634`, `-3122`, `-3123`, `-3141` and `-3255`): a real
[`kotoba-lang/langgraph`](https://github.com/kotoba-lang/langgraph)
`StateGraph`, with the Advisor and Governor as distinct graph nodes and
human-in-the-loop interrupt/resume via checkpointing.

```text
:intake -> :advise -> :govern -> :decide -+-> :commit            (:ok? true)
                                           +-> :request-approval   (:escalate? true, interrupt-before)
                                           +-> :hold               (:hard? true)
```

- `src/business_services_agency/store.kotoba` — `Store` protocol +
  `MemStore`: registered clients, committed records, an append-only
  audit ledger.
- `src/business_services_agency/advisor.kotoba` — `Advisor` protocol;
  `mock-advisor` (deterministic, default) proposes an agency operation
  from a request; `llm-advisor` wraps a `langchain.model/ChatModel` —
  either way the advisor only ever produces a `:propose`-effect
  proposal, never a committed record, and LLM parse failures always
  yield `confidence 0.0` (forces escalation, never fabricated
  confidence).
- `src/business_services_agency/governor.kotoba` —
  `BusinessServicesGovernor/check`: a pure function, wired as its own
  `:govern` node. Hard invariants (unregistered client, a proposal
  whose `:effect` isn't `:propose`) always route to `:hold`. Escalation
  invariants (`:handle-client-funds`, `:broker-above-threshold`, or low
  advisor confidence) always route to `:request-approval` — an
  `interrupt-before` node that the graph checkpoints and only resumes
  on explicit human approval (`actor/approve!`), matching the README's
  robotics-premise statement that handling client funds and brokering
  contracts above a threshold value always require human sign-off.
- `src/business_services_agency/actor.kotoba` — `build-graph`,
  `run-request!`, `approve!`: the `langgraph.graph/state-graph` wiring
  itself.

```bash
clojure -M:test
```

This is what backs this repo's `:maturity :implemented` entry in
[`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation).

## License

AGPL-3.0-or-later.
