# SDK Keys

An SDK key is a **read-only credential** that scopes access to the GrowthBook SDK
connection endpoint for a specific environment.

---

## Properties of an SDK key

| Property | Value |
|---|---|
| Access level | Read-only (fetches flag payload; cannot mutate data) |
| Scope | One environment per key |
| Format | Opaque string (e.g. `sdk-abc123XYZ`) |
| Rotation | Supported; old key continues working until explicitly revoked |

> **Security note:** SDK keys are intended to be embedded in client-side SDKs and are
> therefore considered low-sensitivity credentials. However, to minimise exposure,
> store them as Kubernetes Secrets rather than in plain config files.

---

## Where SDK keys are used

### GrowthBook SDK (application code)

```typescript
import { GrowthBook } from "@growthbook/growthbook";

const gb = new GrowthBook({
  apiHost: "https://growthbook.internal.example.com",
  clientKey: "sdk-abc123XYZ",
});
await gb.loadFeatures();
```

### Backstage backend plugin

The backend plugin uses SDK keys server-side to fetch the full flag payload:

```
GET http://growthbook.<ns>.svc.cluster.local:3100/api/features/<sdkKey>
```

Keys are injected via environment variables mapped from Kubernetes Secrets.

---

## Kubernetes Secret setup

### Create the secret

```bash
kubectl create secret generic growthbook-sdk-keys \
  --namespace backstage \
  --from-literal=GROWTHBOOK_SDK_KEY_PROD="sdk-your-prod-key" \
  --from-literal=GROWTHBOOK_SDK_KEY_STAGING="sdk-your-staging-key"
```

### Reference in the Backstage Deployment

```yaml
# backstage-deployment.yaml (excerpt)
env:
  - name: GROWTHBOOK_SDK_KEY_PROD
    valueFrom:
      secretKeyRef:
        name: growthbook-sdk-keys
        key: GROWTHBOOK_SDK_KEY_PROD
  - name: GROWTHBOOK_SDK_KEY_STAGING
    valueFrom:
      secretKeyRef:
        name: growthbook-sdk-keys
        key: GROWTHBOOK_SDK_KEY_STAGING
```

### Backstage app-config.yaml

```yaml
growthbook:
  baseUrl: http://growthbook.<ns>.svc.cluster.local:3100
  sdkKeys:
    prod: ${GROWTHBOOK_SDK_KEY_PROD}
    staging: ${GROWTHBOOK_SDK_KEY_STAGING}
```

---

## Finding your SDK key

1. Log in to the GrowthBook UI
2. Go to **Settings → Environments**
3. Click the environment whose key you need
4. Copy the **Client Key** (starts with `sdk-`)

---

## Rotating an SDK key

See the runbook: [Rotating SDK Keys](../runbooks/rotating-sdk-keys.md).
