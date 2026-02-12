# SDK Payload Format

The GrowthBook SDK connection endpoint returns a JSON payload that SDKs use for
local flag evaluation. This is also the data source for the Backstage **Feature Flags** tab.

---

## Endpoint

```
GET <growthbook.baseUrl>/api/features/<sdkKey>
```

No authentication header is required — the SDK key itself is the credential.

---

## Response structure

```json
{
  "status": 200,
  "features": {
    "dark-mode": {
      "defaultValue": false,
      "rules": []
    },
    "max-items-per-page": {
      "defaultValue": 20,
      "rules": [
        {
          "condition": { "plan": "enterprise" },
          "force": 100
        }
      ]
    },
    "checkout-config": {
      "defaultValue": {
        "layout": "single-page",
        "showPromoCode": true,
        "maxRetries": 3
      },
      "rules": []
    }
  },
  "dateUpdated": "2024-11-15T09:30:00.000Z"
}
```

### Top-level fields

| Field | Type | Description |
|---|---|---|
| `status` | `number` | HTTP-like status; `200` = success |
| `features` | `object` | Map of flag key → feature object |
| `dateUpdated` | `string` | ISO 8601 timestamp of last payload change |

### Feature object fields

| Field | Type | Description |
|---|---|---|
| `defaultValue` | any | Value returned when no rule matches |
| `rules` | `array` | Ordered list of targeting rules (may be empty) |

---

## How the Backstage backend normalises the payload

The `growthbook-flags-backend` plugin fetches this payload and derives:

```
key          = the feature map key (e.g. "dark-mode")
defaultValue = features[key].defaultValue

type detection:
  null                 → "null"
  typeof === "boolean" → "boolean"
  typeof === "number"  → "number"
  typeof === "object"  → "json"   (arrays included)
  typeof === "string":
    try JSON.parse(value):
      success          → "json"   (pretty-print available)
      fail             → "string"

valuePreview = JSON.stringify(defaultValue) truncated to 80 chars
valuePretty  = JSON.stringify(defaultValue, null, 2)  (JSON types only)
```

---

## Caching

The backend caches the normalised list in memory for **60 seconds**.
A forced refresh is triggered on Backstage backend restart.

If you need an immediate refresh (e.g. after a major flag change), restart the
Backstage backend pod or wait for the cache to expire.

---

## Webhook-based push (future)

GrowthBook supports sending the same SDK payload via webhooks whenever features change.
Implementing a webhook receiver in the backend would allow near-real-time cache
invalidation, eliminating the 60-second lag. This is out of scope for the MVP.
