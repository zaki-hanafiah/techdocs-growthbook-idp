# Feature Flags

A GrowthBook *feature* is a named, typed configuration value that can change behaviour
in your application without a code deploy.

---

## Value types

| Type | Example | Use case |
|---|---|---|
| `boolean` | `true` / `false` | Simple on/off toggles |
| `number` | `10`| Numeric config (limits, thresholds) |
| `string` | `"some-text-here"` | Named variants, copy overrides |
| `json` | `{"key": "value"}` | Structured config objects |

---

## Anatomy of a feature

```json
{
  "key": "SPECIAL_BANNER",
  "defaultValue": false,
  "rules": []
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
{ "condition": { "plan": "legacy" }, "force": true }
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
