# GrowthBook Overview

GrowthBook is an open-source feature flagging and A/B testing platform.
It is available as a **self-hosted** OSS deployment or as a **managed cloud** service.

---

## Core concepts

### Features (Feature Flags)

A *feature* in GrowthBook is a named configuration value that can be toggled or varied
per user, environment, or percentage of traffic.

Every feature has:

- A **key** — the unique identifier used in SDK calls (e.g. `new-checkout-flow`)
- A **value type** — `boolean`, `number`, `string`, or `json`
- A **default value** — the fallback when no targeting rule matches
- **Rules** — ordered list of targeting conditions (force, rollout, experiment)

### Projects

Features are scoped to *projects* for access control and organisation.

### Environments

GrowthBook supports multiple named environments (e.g. `production`, `staging`, `dev`).
Each environment has its own **SDK key** and independently toggles features on/off.

### SDK Connection

SDKs connect to GrowthBook via a **read-only SDK connection endpoint**:

```
GET <baseUrl>/api/features/<sdkKey>
```

The response is a self-contained JSON payload that SDKs evaluate locally — no
real-time network call is needed per flag evaluation.

---

## How evaluation works (client-side)

1. SDK fetches the payload once (and re-fetches periodically or on webhook push).
2. On each `isOn` / `getValue` call the SDK evaluates rules locally against user attributes.
3. No PII is ever sent to GrowthBook servers during flag evaluation.

---

## Further reading

- [GrowthBook Feature Flags](feature-flags.md)
- [SDK Payload Format](sdk-payload.md)
- [Environments](environments.md)
- [SDK Keys](sdk-keys.md)
