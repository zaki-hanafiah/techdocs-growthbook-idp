# Growthbook - Feature Flags

Welcome to the GrowthBook feature flags catalog page.

This entity provides a **live view of all feature flags** and their currently served default values
(per environment) sourced directly from the GrowthBook SDK connection endpoint.

---

## What you will find here

| Section | Purpose |
|---|---|
| **Feature Flags tab** | Live table of every flag, its type, and default value |
| **GrowthBook Concepts** | How GrowthBook works: SDK payload format, environments, SDK keys |
| **How We Use GrowthBook** | Internal architecture, flag naming conventions, lifecycle |
| **Runbooks** | Rotating SDK keys, debugging flags in production |

---

## Quick links

- [GrowthBook Cloud UI](https://app.growthbook.io) — flag management console
- [GrowthBook OSS Docs](https://docs.growthbook.io) — upstream documentation
- Internal Slack: `#platform-feature-flags`

---

## Ownership

| Property | Value |
|---|---|
| Owner | `platform-team` |
| Lifecycle | `production` |
| Type | `service` |

---

## How the Feature Flags tab works

The **Feature Flags** tab in Backstage reads data from the Backstage backend plugin
(`growthbook-flags-backend`). The backend fetches the GrowthBook SDK connection
endpoint payload every ~60 seconds (memory-cached) and normalises each flag into:

```
key         — feature flag identifier
type        — boolean | number | string | json | null
valuePreview — single-line, truncated for the table
valuePretty  — prettified JSON shown in the detail drawer
```

Click any row to open the detail drawer and see the full prettified value.
