# API Chaining Patterns

The real power of Auto.dev with AI agents: composing multiple endpoints into workflows that answer complex questions in a single conversation.

## Pattern 1: Search then Enrich

**User asks:** "Find Toyota RAV4s under $35k near Austin and show me the specs"

```
1. /listings?vehicle.make=Toyota&vehicle.model=RAV4&retailListing.price=1-35000&zip=73301
2. For top results → /specs/{vin} for each
3. Display comparison table
```

**Cost awareness (Growth plan):** 1 listings search ($0.0015) + 10 specs calls ($0.0015 each) = ~$0.02 total.
Starter plan listings cost $0.002/call instead of $0.0015.

## Pattern 2: Search then Safety Check

**User asks:** "Find me SUVs under $25k in Florida with no open recalls"

```
1. /listings?vehicle.bodyStyle=SUV&retailListing.price=1-25000&retailListing.state=FL
2. For each result → /openrecalls/{vin}
3. Filter out vehicles with unresolved recalls
4. Display clean results
```

**Plan note:** Open Recalls requires Scale. If user is on Growth, use /recalls/{vin} instead and note it includes resolved recalls too.

## Pattern 3: Search then Finance

**User asks:** "Find Mazda CX-90s in FL and calculate payments with $10k down at 720 credit"

```
1. /listings?vehicle.make=Mazda&vehicle.model=CX-90&retailListing.state=FL
2. For top results → /apr/{vin}?year={y}&make=Mazda&model=CX-90&zip={zip}&creditScore=720
3. For top results → /payments/{vin}?price={price}&zip={zip}&downPayment=10000
4. Display with monthly payment comparison
```

## Pattern 4: Full Vehicle Report

**User asks:** "Tell me everything about VIN JM3KKAHD5T1379650"

```
1. /vin/{vin}                    — basic decode
2. /specs/{vin}                  — full specifications
3. /build/{vin}                  — factory options & colors
4. /photos/{vin}                 — images
5. /recalls/{vin}                — recall history
6. /tco/{vin}?zip={userZip}     — ownership costs
```

**Cost:** ~$0.18 total on Growth. Warn user before running.

## Pattern 5: Plate then Full Lookup

**User asks:** "Look up plate ABC123 from California"

```
1. /plate/CA/ABC123              — resolve to VIN ($0.55)
2. /vin/{vin}                    — decode
3. /openrecalls/{vin}            — safety check
```

**Cost:** ~$0.62 on Scale. Highest-cost chain — always confirm first.

## Pattern 6: Market Price Analysis

**User asks:** "What's the price range for 2023 BMW X5s?"

```
1. /listings?vehicle.make=BMW&vehicle.model=X5&vehicle.year=2023
2. Calculate min/max/avg/median from results
3. Break down by trim, mileage bracket, location
4. Identify listings priced well below average
```

Uses listings data only — no additional API calls needed.

## Pattern 7: Batch Export

**User asks:** "Export all Honda Civics under $20k in Texas to CSV"

```
1. Page through /listings until all results collected:
   /listings?vehicle.make=Honda&vehicle.model=Civic&retailListing.price=1-20000&retailListing.state=TX&page=1
   /listings?...&page=2  (if more results)
2. Flatten to CSV with key fields
3. Save to user-specified path
```

**Rate limits:** Starter 5 req/s, Growth 10 req/s, Scale 50 req/s.
Add delay between pages if needed.

## Pattern 8: Compare Two Vehicles

**User asks:** "Compare these two VINs side by side"

```
1. /specs/{vin1} + /specs/{vin2}       — parallel
2. /tco/{vin1} + /tco/{vin2}           — parallel
3. /payments/{vin1} + /payments/{vin2}  — parallel (same terms)
4. Display side-by-side comparison table
```

Run independent API calls in parallel for speed.

## Cost Estimation

Before running any chain, estimate total cost:

```
total = (listings_pages x listings_cost)
      + (vins_to_enrich x enrichment_cost_per_vin)
```

**Rules:**
- Always estimate before chains that touch 10+ VINs
- Always warn before using Build ($0.10/call) or Plate ($0.55/call)
- For large exports, tell user: "This will query ~X pages at $Y total"
- If estimated cost > $1, ask for explicit confirmation

## Error Handling in Chains

- If an enrichment call 404s for one VIN, skip it and continue the chain
- If a permissions error occurs, stop the chain and suggest plan upgrade
- If rate limited (429), back off and retry with delay
- Always report partial results if chain is interrupted
