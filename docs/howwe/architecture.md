# Architecture

This page describes how GrowthBook is deployed and integrated within our platform.

---

## Workflow / Deployment topology

<img width="779" height="391" alt="image" src="https://github.com/user-attachments/assets/9c9a25e5-5c9f-476b-b887-e65185620954" />

---

## Data flow for the Feature Flags tab (Backstage as Consumer)

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

GrowthBook is deployed as self-hosted service.

| Property | Value |
|---|---|
| Namespace | `growthbook` (or as configured) |
| Service DNS | `/growthbook` |
| Port | `3100 || as configured` |
| Database | MongoDB (persistent volume) |

---

## Secrets management

All SDK keys are stored as Kubernetes Secrets and injected as environment variables
into the Backstage backend pod. No plaintext secrets appear in any config file committed
to source control.

See [SDK Keys](../growthbook/sdk-keys.md) for the full setup.

---

## Flag evaluation in application code

Each SDK instance fetches the payload directly from the GrowthBook service and evaluates rules locally with in-memory caching, meaning **zero network latency** per flag check after the initial fetch.
