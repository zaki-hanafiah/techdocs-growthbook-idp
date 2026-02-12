# Feature Flags

A GrowthBook *feature* is a named, typed configuration value that can change behaviour
in your application without a code deploy.

---

## Value types

| Type | Example | Use case |
|---|---|---|
| `boolean` | `true` / `false` | Simple on/off toggles |
| `number` | `42`, `3.14` | Numeric config (limits, thresholds) |
| `string` | `"v2"` | Named variants, copy overrides |
| `json` | `{"limit": 5, "mode": "fast"}` | Structured config objects |

---

## Anatomy of a feature

```json
{
  "key": "new-checkout-flow",
  "defaultValue": false,
  "rules": [
    {
      "condition": { "betaUser": true },
      "force": true
    },
    {
      "variations": [false, true],
      "weights": [0.5, 0.5],
      "key": "new-checkout-flow-exp"
    }
  ]
}
```

| Field | Description |
|---|---|
| `key` | Unique identifier, used in SDK calls |
| `defaultValue` | Returned when no rule matches |
| `rules` | Ordered list of targeting/rollout/experiment rules |

---

## Rule types

### Force rule

Returns a fixed value when the condition matches.

```json
{ "condition": { "plan": "enterprise" }, "force": true }
```

### Rollout rule

Gradually rolls out a value to a percentage of users (by hash).

```json
{ "force": true, "coverage": 0.10, "hashAttribute": "id" }
```

### Experiment rule

Runs an A/B test, splitting traffic across variations.

```json
{
  "variations": ["control", "treatment"],
  "weights": [0.5, 0.5],
  "key": "checkout-exp",
  "hashAttribute": "id"
}
```

---

## Feature status

| Status | Meaning |
|---|---|
| `active` | Enabled in the environment, rules apply |
| `draft` | Not yet enabled; default value is always returned |
| `archived` | Disabled; useful when cleaning up old flags |

---

## Default values in the Backstage plugin

The **Feature Flags tab** shows *default values only* — the value returned when no
targeting rule matches. This is intentional: it gives a single stable reference point
for what the flag produces out of the box, without needing to know the requester's
user attributes.
