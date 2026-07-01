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

## License

AGPL-3.0-or-later.
