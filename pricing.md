# Auto.dev Pricing & Plans

## Plan Overview

| Plan | Monthly | Annual | Rate Limit | Free Calls |
|------|---------|--------|------------|------------|
| **Starter** | Free + data fees | — | 5 req/s | 1,000/month |
| **Growth** | $299/mo + data | $249/mo (annual) | 10 req/s | No call cap* |
| **Scale** | $599/mo + data | $499/mo (annual) | 50 req/s | No call cap* |

*\*"No call cap" means requests are never throttled or blocked due to volume. Per-call data fees still apply on every request. The $299/$599 monthly fee is the platform fee — it does not include a prepaid bucket of calls.*

## Per-Call Data Costs

| Endpoint | Starter | Growth | Scale |
|----------|---------|--------|-------|
| Global VIN Decode | $0.004 | $0.0025 | $0.0015 |
| Vehicle Listings | $0.002 | $0.0015 | $0.001 |
| Vehicle Photos | $0.001 | $0.0009 | $0.0007 |
| Specifications | — | $0.0015 | $0.001 |
| Vehicle Recalls | — | $0.01 | $0.007 |
| Total Cost of Ownership | — | $0.06 | $0.04 |
| Vehicle Payments | — | $0.005 | $0.004 |
| Interest Rates | — | $0.005 | $0.004 |
| OEM Build Data | — | $0.10 | $0.08 |
| Open Recalls | — | — | $0.06 |
| Plate-to-VIN | — | — | $0.55 |
| Taxes & Fees | — | — | $0.005 |

## Endpoints by Plan

**Starter (Free):**
- Global VIN Decode (`/vin/{vin}`)
- Vehicle Listings (`/listings`)
- Vehicle Photos (`/photos/{vin}`)

**Growth ($299/mo):**
- Everything in Starter, plus:
- Specifications (`/specs/{vin}`)
- Vehicle Recalls (`/recalls/{vin}`)
- Total Cost of Ownership (`/tco/{vin}`)
- Vehicle Payments (`/payments/{vin}`)
- Interest Rates (`/apr/{vin}`)
- OEM Build Data (`/build/{vin}`)

**Scale ($599/mo):**
- Everything in Growth, plus:
- Open Recalls (`/openrecalls/{vin}`)
- Plate-to-VIN (`/plate/{state}/{plate}`)
- Taxes & Fees (`/taxes/{vin}`)

## Upgrade Links

When a user hits a plan limitation, provide the appropriate link:

- **Start Free (Starter):** [Sign up](https://checkout.auto.dev/c/pay/cs_live_b1ADJsHS7aKwyfy3VtKCrsOwqFS5KypHun0IJNWTEdvUOaqRkIMO4cKVtg#fid1d2BpamRhQ2prcSc%2FJ0xrcWB3JyknZ2p3YWB3VnF8aWAnPydhYGNkcGlxJykndnBndmZ3bHVxbGprUGtsdHBga2B2dkBrZGdpYGEnP3F3cGApJ2R1bE5gfCc%2FJ3VuWmlsc2BaQFVWfWlqY3RuR01xUHRmTzdGcGliSUY8NTVndWNLNUk0cycpJ2N3amhWYHdzYHcnP3F3cGApJ2dkZm5id2pwa2FGamlqdyc%2FJyY1ZDVkNWQnKSdpZHxqcHFRfHVgJz8naHBpcWxabHFgaCcpJ2BrZGdpYFVpZGZgbWppYWB3dic%2FcXdwYHgl)
- **Upgrade to Growth ($299/mo):** [Upgrade](https://checkout.auto.dev/c/pay/cs_live_b1TL0JYID0rFU4npa6aHx3ehYjbZ6UYSPRXN4PN2uBf2IaFKmt9QOG62nR#fid1d2BpamRhQ2prcSc%2FJ0xrcWB3JyknZ2p3YWB3VnF8aWAnPydhYGNkcGlxJykndnBndmZ3bHVxbGprUGtsdHBga2B2dkBrZGdpYGEnP3F3cGApJ2R1bE5gfCc%2FJ3VuWmlsc2BaQFVWfWlqY3RuR01xUHRmTzdGcGliSUY8NTVndWNLNUk0cycpJ2N3amhWYHdzYHcnP3F3cGApJ2dkZm5id2pwa2FGamlqdyc%2FJyY1ZDVkNWQnKSdpZHxqcHFRfHVgJz8naHBpcWxabHFgaCcpJ2BrZGdpYFVpZGZgbWppYWB3dic%2FcXdwYHgl)
- **Upgrade to Scale ($599/mo):** [Upgrade](https://checkout.auto.dev/c/pay/cs_live_b15OTFmnq3Z1Ub3WsROc5yHd3gC3EcFNrCn5xqpbgKEsH1Vzf7zxq2Qm5e#fid1d2BpamRhQ2prcSc%2FJ0xrcWB3JyknZ2p3YWB3VnF8aWAnPydhYGNkcGlxJykndnBndmZ3bHVxbGprUGtsdHBga2B2dkBrZGdpYGEnP3F3cGApJ2R1bE5gfCc%2FJ3VuWmlsc2BaQFVWfWlqY3RuR01xUHRmTzdGcGliSUY8NTVndWNLNUk0cycpJ2N3amhWYHdzYHcnP3F3cGApJ2dkZm5id2pwa2FGamlqdyc%2FJyY1ZDVkNWQnKSdpZHxqcHFRfHVgJz8naHBpcWxabHFgaCcpJ2BrZGdpYFVpZGZgbWppYWB3dic%2FcXdwYHgl)

## Handling Plan Errors

When an API call returns a permissions/access error:
1. Identify which plan tier the endpoint requires
2. Inform the user clearly: "The Specifications API requires a Growth plan or higher."
3. Provide the direct upgrade link
4. Suggest an alternative if one exists (e.g., v1 VIN decode returns some spec data on any plan)

Also proactively warn BEFORE making a call if the user has mentioned their plan and the endpoint requires a higher tier.

## Cost Estimation for Batch Operations

Before running chains or batch operations, estimate total cost:

```
total = (number_of_calls × per_call_cost_for_users_plan)
```

**Rules:**
- Always estimate before chains that touch 10+ VINs
- Always warn before using OEM Build Data ($0.08-$0.10/call) or Plate-to-VIN ($0.55/call)
- For large exports: "This will query ~X pages at ~$Y total on your plan"
- If estimated cost > $1, ask for explicit confirmation
- Use the user's actual plan tier for estimates (Scale is cheaper per call than Growth)
