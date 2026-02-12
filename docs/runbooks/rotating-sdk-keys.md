# Runbook: Rotating SDK Keys

Use this runbook when you need to rotate a GrowthBook SDK key for any environment.

**Estimated time:** 15–30 minutes
**Impact:** Zero downtime if steps are followed in order (old key stays active until step 5).

---

## When to rotate

- Suspected key compromise.
- Scheduled key rotation (company policy).
- Offboarding a team member who had access to the key value.
- Key was accidentally committed to source control.

---

## Prerequisites

- Access to the GrowthBook UI (admin or environment-manager role).
- `kubectl` access to the `backstage` namespace.
- Ability to roll out the Backstage backend Deployment.

---

## Steps

### 1. Generate a new SDK key in GrowthBook

1. Log in to the GrowthBook UI.
2. Go to **Settings → Environments**.
3. Select the environment you are rotating (e.g. `production`).
4. Click **Add SDK Connection** (or **Rotate Key** if available in your version).
5. Copy the new key — it starts with `sdk-`.

> At this point GrowthBook has **both** the old and new key active.
> No services are broken yet.

### 2. Update the Kubernetes Secret

```bash
# Patch the secret with the new key value
kubectl patch secret growthbook-sdk-keys \
  --namespace backstage \
  --type='json' \
  -p='[{"op":"replace","path":"/data/GROWTHBOOK_SDK_KEY_PROD","value":"'$(echo -n "sdk-NEW-KEY-HERE" | base64)'"}]'
```

Verify:
```bash
kubectl get secret growthbook-sdk-keys -n backstage \
  -o jsonpath='{.data.GROWTHBOOK_SDK_KEY_PROD}' | base64 -d
```

### 3. Restart the Backstage backend pod

The pod must restart to pick up the updated Secret value:

```bash
kubectl rollout restart deployment/backstage -n backstage
kubectl rollout status deployment/backstage -n backstage
```

### 4. Verify the Feature Flags tab is working

1. Open the **Growthbook - Feature Flags** entity in Backstage.
2. Navigate to the **Feature Flags** tab.
3. Confirm the flag list loads without error.

If you see an error, the new SDK key may be incorrect — check the secret value and
repeat from step 2.

### 5. Revoke the old SDK key

Once you have confirmed the new key works:

1. Go back to **GrowthBook UI → Settings → Environments**.
2. Find the old SDK connection entry.
3. Click **Delete** (or **Revoke**).

> Do **not** revoke the old key before verifying the new key works.

### 6. Rotate application SDK keys (if applicable)

If application pods also use the same SDK key (they should use a **separate** SDK
connection for isolation), repeat the secret update and pod rollout for those deployments.

---

## Rollback

If anything goes wrong before step 5:

- The old key is still active in GrowthBook.
- Revert the Kubernetes Secret to the old key value and restart the Backstage pod.

After step 5 (old key revoked), rollback requires generating a new key again.

---

## Post-rotation checklist

- [ ] New key is in Kubernetes Secret
- [ ] Backstage pod restarted and healthy
- [ ] Feature Flags tab loads correctly
- [ ] Old key revoked in GrowthBook UI
- [ ] Rotation event logged in incident management system (if required by policy)
