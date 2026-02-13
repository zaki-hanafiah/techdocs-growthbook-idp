# Flag Lifecycle

Every feature flag has a lifecycle. Managing it actively keeps the codebase clean
and prevents "flag debt" from accumulating.

---

## Lifecycle stages

```
  Draft ──▶ Active ──▶ Rolled Out ──▶ Cleaned Up
              │                            ▲
              └──── Experiment ────────────┘
```

| Stage | Description | GrowthBook status |
|---|---|---|
| **Draft** | Created, not yet enabled in any environment | Feature exists, all envs toggled off |
| **Active** | Enabled and routing traffic (toggle or rollout) | Enabled in ≥1 environment |
| **Rolled Out** | Feature is at 100 % for all users | Boolean `true` / full rollout |
| **Cleaned Up** | Flag removed from code and archived in GrowthBook | Archived |

---

## When to clean up a flag

Clean up a flag when **all** of these are true:

- The feature has been fully rolled out (or permanently disabled) for **≥ 4 weeks**.
- No active experiments reference the flag.
- The code path gated by the flag has been hardcoded (or deleted).

Leaving stale flags in GrowthBook creates confusion in the Backstage Feature Flags tab
and in the codebase.

---

## Cleanup process

1. **Hardcode the winning variant** in application code (remove the SDK call).
2. Deploy the code change.
3. **Archive the flag** in the GrowthBook UI:
   - Open the feature → **Actions → Archive**.
   - Archived features are hidden from the main list but retained for audit.
4. Create a follow-up ticket if DB migrations or other cleanup is needed.


---

## Flag debt review

The platform team runs a **quarterly flag debt review**:
- Open the GrowthBook UI → **Features** → filter by `rolled out` or `stale (>90 days)`.
- Cross-reference with the Backstage Feature Flags tab.
- Raise cleanup tickets for any flag in the "Rolled Out" stage for > 90 days.
