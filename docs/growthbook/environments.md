# Environments

GrowthBook uses *environments* to isolate feature flag state across deployment tiers.
Each environment is independently togglable and has its own SDK key.

---

## Default environments

GrowthBook ships with two built-in environments:

| Environment | Purpose |
|---|---|
| `production` | Live traffic; changes here affect real users immediately |
| `development` | Local and CI flag evaluation; safe to experiment |

Additional environments (e.g. `staging`, `canary`) can be added by organisation admins
in the GrowthBook UI under **Settings → Environments**.

---

## How environments affect feature evaluation

A feature can be **enabled** or **disabled** per environment independently of its rules.

- If a feature is **disabled** in an environment, the SDK always returns `defaultValue`,
  regardless of any targeting rules.
- If a feature is **enabled**, rules are evaluated in order.

This lets you safely test targeting logic in `staging` before enabling a flag in `production`.

---

## Environments in the Backstage plugin

The Backstage **Feature Flags** tab can be scoped to a single environment via the entity
annotation:

```yaml
# catalog-info.yaml
annotations:
  growthbook.io/env: "prod"
```

The backend plugin maps the annotation value to a configured SDK key:

```yaml
# app-config.yaml (Backstage backend)
growthbook:
  sdkKeys:
    prod: ${GROWTHBOOK_SDK_KEY_PROD}
    staging: ${GROWTHBOOK_SDK_KEY_STAGING}
```

---

## Adding a new environment

1. Go to GrowthBook UI → **Settings → Environments**
2. Click **Add Environment**, give it a name (e.g. `canary`)
3. GrowthBook generates an SDK key for the new environment
4. Add the key to the Backstage backend `app-config.yaml` under `growthbook.sdkKeys`
5. Store the key value in the K8s Secret (see [SDK Keys](sdk-keys.md))
6. Update `catalog-info.yaml` with `growthbook.io/env: "canary"` if needed

---

## Environment promotion checklist

When promoting a flag from `staging` to `production`:

- [ ] Verify the flag behaves as expected in `staging` (check logs / experiments)
- [ ] Review the `defaultValue` — it will be shown in the Backstage Feature Flags tab
- [ ] Enable the flag in `production` via the GrowthBook UI
- [ ] Monitor rollout metrics for the first 30 minutes
- [ ] Update flag description/tags in GrowthBook if needed
