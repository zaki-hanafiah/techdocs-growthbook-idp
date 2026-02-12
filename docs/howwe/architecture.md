# Architecture

This page describes how GrowthBook is deployed and integrated within our platform.

---

## Deployment topology

```
┌─────────────────────────────────────────────────────────────┐
│  Kubernetes cluster                                         │
│                                                             │
│  ┌─────────────────┐        ┌──────────────────────────┐   │
│  │  Application    │        │  GrowthBook              │   │
│  │  Pods           │──────▶│  Service                  │   │
│  │  (SDK clients)  │  SDK   │  growthbook.<ns>.svc:3100│   │
│  └─────────────────┘  fetch └──────────────────────────┘   │
│                                        │                    │
│  ┌─────────────────┐                   │ /api/features/     │
│  │  Backstage      │                   │  <sdkKey>          │
│  │  Backend        │───────────────────┘                    │
│  │  (plugin proxy) │                                        │
│  └────────┬────────┘                                        │
│           │  /api/growthbook-flags/flags?env=prod           │
│  ┌────────▼────────┐                                        │
│  │  Backstage      │                                        │
│  │  Frontend       │                                        │
│  │  (entity tab)   │                                        │
│  └─────────────────┘                                        │
└─────────────────────────────────────────────────────────────┘
```

---

## Data flow for the Feature Flags tab

1. User opens the `growthbook-feature-flags` entity in Backstage.
2. The **Frontend plugin** (`EntityGrowthbookFlagsContent`) calls
   `GET /api/growthbook-flags/flags?env=prod`.
3. The **Backend plugin** checks its 60-second in-memory cache.
   - **Cache hit**: returns cached normalised list.
   - **Cache miss**: fetches `GET <baseUrl>/api/features/<sdkKey>` from GrowthBook,
     normalises, caches, and returns.
4. Frontend renders the flag table. Click a row → detail drawer with prettified JSON.

---

## GrowthBook service

GrowthBook is deployed as an internal Kubernetes service.

| Property | Value |
|---|---|
| Namespace | `growthbook` (or as configured) |
| Service DNS | `growthbook.<ns>.svc.cluster.local` |
| Port | `3100` |
| Database | MongoDB (persistent volume) |

The service is **not** exposed externally. Only workloads inside the cluster can reach it.

---

## Secrets management

All SDK keys are stored as Kubernetes Secrets and injected as environment variables
into the Backstage backend pod. No plaintext secrets appear in any config file committed
to source control.

See [SDK Keys](../growthbook/sdk-keys.md) for the full setup.

---

## Flag evaluation in application code

Application pods include the GrowthBook JavaScript/Go/Python SDK.
Each SDK instance fetches the payload directly from the GrowthBook service (cluster-internal URL)
and evaluates rules locally, meaning **zero network latency** per flag check after the initial fetch.
