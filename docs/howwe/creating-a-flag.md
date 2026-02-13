# Creating a Flag

Follow this guide when adding a new feature flag to GrowthBook.

---

## Before you start

- Decide the **value type**: boolean (most common), number, string, or JSON.
- Choose a **key** following the naming convention (see below).
- Agree on the **default value** — this is what the Backstage Feature Flags tab will show.
- Confirm which **environment** to enable it in first.

---

## Naming convention

| Pattern | Example | When to use |
|---|---|---|
| `<area>-<feature>` | `SPECIAL_BANNER` | Boolean flag for a internal features area |
| `<area>-<setting>-config` | `REMOTE_CONFIG` | JSON config object |
| `<area>-<metric>-limit` | `API_RATE_LIMIT` | Numeric threshold |

Rules:
- UPPERCASE, hyphen-separated, SCREAMING.
- No environment name in the key (environments are handled by GrowthBook, not the key).
- Be descriptive; the key is visible in code and in the Backstage UI.

---

## Step-by-step

### 1. Create the feature in GrowthBook UI

1. Open the GrowthBook UI and navigate to **Features**.
2. Click **Add Feature**.
3. Fill in:
   - **Feature Key** — follow the naming convention above.
   - **Value Type** — `boolean`, `number`, `string`, or `JSON`.
   - **Default Value** — the safe fallback when no rule matches.
   - **Description** — one sentence explaining what the flag controls.
   - **Tags** — add the owning team tag (e.g. `platform`, `growth`).
4. Click **Save**.

### 2. Add targeting rules (if any)

- For a simple toggle: leave rules empty and toggle the feature on/off per environment.
- For a phased rollout: add a **Rollout** rule with an initial coverage of `0.0` and
  increase it over time.
- For an A/B test: add an **Experiment** rule.

### 3. Enable in the target environment

1. In the feature detail page, find the environment row (e.g. `staging`).
2. Toggle it **on**.
3. Verify in staging before enabling in production.

### 4. Update application code

Add the SDK call in your service:

```typescript
// boolean flag
const isEnabled = gb.isOn("SPECIAL_BANNER");

// JSON config
const config = gb.getFeatureValue("REMOTE_CONFIG", defaultConfig);
```

### 5. Verify in Backstage

Open the **Growthbook - Feature Flags** entity in Backstage and confirm:
- The new flag appears in the Feature Flags tab.
- The `defaultValue` matches what you set in step 1.
- The `type` is correct.

---

## Checklist

- [ ] Key follows naming convention
- [ ] Default value is safe (feature is off / minimal by default)
- [ ] Description is filled in GrowthBook UI
- [ ] Owning team tag is set
- [ ] Enabled in `staging` and tested
- [ ] Enabled in `production` only when ready
- [ ] Application code uses the flag (no dead flags)
