# Business Workflows

Prebuilt analytical workflows for common automotive business use cases. When a user describes a business need, match it to these patterns.

## Workflow 1: Dealer Competitive Analysis

**User asks:** "Compare pricing across Toyota dealers within 50 miles of Orlando"

```
1. Search listings:
   /listings?vehicle.make=Toyota&zip=32801&distance=50

2. Group results by dealer name

3. Calculate per dealer:
   - Inventory count
   - Average price
   - Price range (min-max)
   - Average days listed (createdAt vs today)
   - Mix: new vs used, body style breakdown

4. Display as comparison table sorted by inventory size

5. Optional: export full dealer comparison to CSV
```

**Output format:**
```
| Dealer | Count | Avg Price | Range | Avg Days Listed | New/Used |
|--------|-------|-----------|-------|-----------------|----------|
| Toyota of Orlando | 145 | $38,200 | $22k-$68k | 23 | 60/40 |
| Central FL Toyota | 98 | $36,800 | $19k-$55k | 31 | 45/55 |
| Kissimmee Toyota | 67 | $35,100 | $18k-$52k | 28 | 50/50 |
```

## Workflow 2: Market Price Analysis

**User asks:** "What's the market rate for a 2023 Honda CR-V in California?"

```
1. Search listings:
   /listings?vehicle.make=Honda&vehicle.model=CR-V&vehicle.year=2023&retailListing.state=CA

2. Calculate statistics:
   - Count of active listings
   - Mean, median, min, max price
   - Standard deviation
   - Price by trim level
   - Price by mileage bracket (0-10k, 10-30k, 30-50k, 50k+)

3. Identify deals: listings priced well below median for their trim/mileage bracket
```

**Output format:**
```
2023 Honda CR-V Market Analysis — California
═══════════════════════════════════════════
Active Listings: 234
Median Price: $29,800
Mean Price: $30,450
Range: $24,500 - $38,900

By Trim:
  LX:      $25,200 avg (42 listings)
  EX:      $28,400 avg (67 listings)
  EX-L:    $31,200 avg (58 listings)
  Touring: $34,800 avg (45 listings)
  Hybrid:  $33,500 avg (22 listings)

By Mileage:
  0-10k mi:   $32,400 avg
  10-30k mi:  $29,800 avg
  30-50k mi:  $27,200 avg
  50k+ mi:    $24,900 avg

Below Market Value (potential deals): 12 listings
```

## Workflow 3: Inventory Aging Report

**User asks:** "Show me which vehicles have been sitting longest at our dealership"

```
1. Search listings filtered to specific dealer:
   /listings?retailListing.state={state}&page=1 through N
   Filter client-side by dealer name

2. Calculate days on market:
   today - createdAt for each listing

3. Group by aging brackets:
   - Fresh (0-15 days)
   - Normal (16-30 days)
   - Aging (31-60 days)
   - Stale (61-90 days)
   - Critical (90+ days)

4. For critical/stale vehicles, compare listing price to median market price
   from /listings search of same make/model/year/trim

5. Flag overpriced stale inventory
```

**Output format:**
```
Inventory Aging Report — [Dealer Name]
══════════════════════════════════════
Total Vehicles: 156

Fresh (0-15 days):     45 vehicles (29%)
Normal (16-30 days):   38 vehicles (24%)
Aging (31-60 days):    32 vehicles (21%)
Stale (61-90 days):    24 vehicles (15%)
Critical (90+ days):   17 vehicles (11%)

Critical Inventory — Needs Attention:
| VIN | Vehicle | Days | Price | Market Value | Delta |
|-----|---------|------|-------|-------------|-------|
| ... | 2023 BMW X3 | 127 | $42,000 | $38,500 | +$3,500 overpriced |
| ... | 2023 Audi Q5 | 104 | $39,900 | $37,200 | +$2,700 overpriced |
```

## Workflow 4: Vehicle Purchase Due Diligence

**User asks:** "I'm considering buying this car, run a full check"

```
1. Decode VIN:
   /vin/{vin} → confirm vehicle identity

2. Get full specs:
   /specs/{vin} → engine, features, dimensions

3. Get factory build:
   /build/{vin} → original options, colors, sticker price

4. Check recalls:
   /recalls/{vin} → full recall history
   /openrecalls/{vin} → unresolved recalls (Scale)

5. Get photos:
   /photos/{vin} → current listing photos

6. Calculate payments:
   /payments/{vin}?price={asking}&zip={buyer_zip}&downPayment={down}

7. Total cost of ownership:
   /tco/{vin}?zip={buyer_zip}

8. Compare asking price to:
   - Original MSRP + options (from build data)
   - Median price of comparable listings (same make/model/year/trim)

9. Generate buy/wait/pass recommendation based on:
   - Price vs comparable listings (deal score)
   - Open recalls (safety risk)
   - TCO (long-term cost)
```

## Workflow 5: Fleet Procurement Analysis

**User asks:** "We need to buy 20 SUVs for our sales team, budget $800k total"

```
1. Calculate per-vehicle budget: $800k / 20 = $40k each

2. Search candidates:
   /listings?vehicle.bodyStyle=SUV&retailListing.price=1-40000&retailListing.state={state}

3. Group by make/model, calculate:
   - Average price per model
   - How many available (can we fill 20?)
   - Average mileage
   - Fuel economy (from /specs)

4. For top 3 model candidates, calculate TCO:
   /tco/{sample_vin} for each model

5. Build comparison:
   | Model | Avg Price | Available | MPG | 5yr TCO | Total Fleet Cost |

6. Factor in:
   - Volume availability (can we source 20 nearby?)
   - Maintenance consistency (same model = bulk service deals)
   - Recall status across model

7. Recommend best value model with sourcing plan
```

## Workflow 6: Price Trend Monitoring

**User asks:** "Track how CX-90 prices have been changing in Florida"

```
1. Search current listings:
   /listings?vehicle.make=Mazda&vehicle.model=CX-90&retailListing.state=FL

2. Analyze listing data:
   - Count of active listings (supply indicator)
   - Average/median current listing price
   - Price distribution by trim and mileage bracket
   - New vs used mix

3. Summarize market:
   "CX-90 in FL: 100 active listings. Median: $42,500.
   Turbo Select avg: $38,200. Turbo S Premium avg: $55,400."

4. Identify best deals:
   Listings priced well below median for their trim/mileage bracket
```

## Workflow 7: New vs Used Value Comparison

**User asks:** "Should I buy new or used for a Toyota Highlander?"

```
1. Search new listings:
   /listings?vehicle.make=Toyota&vehicle.model=Highlander&vehicle.year=2025-2026&retailListing.miles=0-100

2. Search used listings (1-3 years old):
   /listings?vehicle.make=Toyota&vehicle.model=Highlander&vehicle.year=2022-2024&retailListing.miles=1-50000

3. Get specs for sample new and used:
   /specs/{new_vin} and /specs/{used_vin}

4. Calculate TCO for both:
   /tco/{new_vin} and /tco/{used_vin}

5. Calculate payments for both:
   /payments/{new_vin}?price={new_price}&zip={zip}
   /payments/{used_vin}?price={used_price}&zip={zip}

6. Compare:
   | | New 2025 | Used 2023 | Savings |
   |---|---|---|---|
   | Price | $42,000 | $33,000 | $9,000 |
   | Monthly Payment | $720 | $580 | $140/mo |
   | 5yr TCO | $62,000 | $51,000 | $11,000 |
   | Depreciation (yr 1) | $6,300 | $3,200 | $3,100 |
   | Warranty Remaining | 3yr/36k | 1yr/12k | — |

7. Recommendation based on total cost difference vs feature/warranty tradeoff
```

## Workflow 8: Regional Inventory Comparison

**User asks:** "Where are the cheapest Tesla Model 3s — Florida or Texas?"

```
1. Search FL:
   /listings?vehicle.make=Tesla&vehicle.model=Model+3&retailListing.state=FL

2. Search TX:
   /listings?vehicle.make=Tesla&vehicle.model=Model+3&retailListing.state=TX

3. Compare:
   - Listing count (supply)
   - Average/median price
   - Price distribution by trim
   - Average mileage
   - Taxes & fees difference (Scale plan required):
     /taxes/{sample_vin}?zip={fl_zip} vs /taxes/{sample_vin}?zip={tx_zip}

4. Factor in:
   - State sales tax difference
   - Registration fees
   - Transport cost if buying out of state

5. Total landed cost comparison including taxes and transport
```
