# Runbook: Debugging a Flag

Use this runbook when a feature flag is not behaving as expected in any environment.

---

## Common symptoms

| Symptom | Likely cause |
|---|---|
| Flag always returns `defaultValue` | Feature is disabled in the environment, or wrong SDK key |
| Flag returns unexpected variant | Targeting rule order; user attribute mismatch |
| Flag not appearing in Backstage tab | Flag created after last cache refresh; Backstage backend error |
| JSON flag returns `null` or empty | SDK key missing; payload parse error |

---

## Step 1 — Check the flag in GrowthBook UI

1. Open the GrowthBook UI → **Features**.
2. Find the flag by key.
3. Confirm:
   - The feature is **enabled** in the target environment (toggle must be on).
   - The **default value** is what you expect.
   - Rules are in the correct order (rules are evaluated top-to-bottom; first match wins).

---

## Step 2 — Fetch the raw SDK payload

Fetch the SDK payload directly and inspect it:

```bash
# Replace <baseUrl> and <sdkKey> with real values
curl -s "http://growthbook.<ns>.svc.cluster.local:3100/api/features/<sdkKey>" \
  | jq '.features["your-flag-key"]'
```

Expected output for an enabled boolean flag:
```json
{
  "defaultValue": false,
  "rules": []
}
```

If the flag is absent from the payload:
- The feature may not exist yet, or
- It may be disabled in that environment (disabled features are excluded from the payload).

---

## Step 3 — Check the Backstage backend logs

```bash
kubectl logs -n backstage deployment/backstage \
  --container backstage \
  | grep -i growthbook
```

Look for:
- `GrowthBook fetch error` — the backend cannot reach GrowthBook.
- `Cache miss` / `Cache hit` — normal operational messages.
- HTTP 401 / 403 — SDK key is invalid.

---

## Step 4 — Force a cache refresh

The Backstage backend caches the flag list for 60 seconds.
If you need an immediate refresh:

```bash
kubectl rollout restart deployment/backstage -n backstage
kubectl rollout status deployment/backstage -n backstage
```

Then reload the Feature Flags tab in Backstage.

---

## Step 5 — Verify user attributes (for targeting rules)

If the flag exists and the payload is correct but the application receives the wrong
value, the issue is likely in **targeting rule evaluation**:

1. Enable **Dev Mode** in the GrowthBook SDK during local debugging:
   ```typescript
   const gb = new GrowthBook({ ..., enableDevMode: true });
   ```
2. Open the browser console → GrowthBook DevTools extension (if installed).
3. Inspect which rule matched and what attribute values were used.
4. Compare against the rule conditions in the GrowthBook UI.

Common mistakes:
- `hashAttribute` refers to a user attribute that is `undefined` at evaluation time.
- Condition uses `$in` but the attribute value type is wrong (string vs number).

---

## Step 6 — Check application SDK version

Outdated SDK versions may not support newer rule types.

```bash
# Node.js example
npm list @growthbook/growthbook
```

Upgrade if you are more than one major version behind the GrowthBook server.

---

## Escalation

If you cannot resolve the issue after the steps above:

1. Collect:
   - The raw SDK payload for the affected flag (`curl` output from step 2).
   - Backstage backend logs from step 3.
   - Application SDK version.
2. Post in `#platform-feature-flags` with the collected info.
