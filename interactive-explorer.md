# Interactive Explorer

Conversational workflow for refining vehicle searches across multiple messages. The agent maintains state and progressively narrows results.

## How It Works

When a user starts a broad search, treat it as the beginning of an interactive session. Keep the current result set in memory and apply new filters incrementally.

## Conversation Flow

```
User: "Show me SUVs under $50k in Florida"
Agent: [searches, shows top 10 of 247 results as table]
       "Found 247 SUVs under $50k in Florida. Want to narrow it down?"

User: "Only AWD"
Agent: [adds vehicle.drivetrain=AWD, re-searches]
       "Narrowed to 89 AWD SUVs. Here are the top 10:"

User: "Nothing over 30k miles"
Agent: [adds retailListing.miles=1-30000]
       "Down to 42 vehicles under 30k miles:"

User: "Show me only Toyota and Mazda"
Agent: [adds vehicle.make=Toyota,Mazda]
       "18 Toyota and Mazda AWD SUVs under $50k with <30k miles:"

User: "Any of these have open recalls?"
Agent: [calls /openrecalls/{vin} for each, or /recalls if on Growth]
       "3 vehicles have open recalls. Here are the 15 clean ones:"

User: "What would the payments be on the top 5 with $8k down?"
Agent: [calls /payments/{vin} for top 5]
       "Monthly payments with $8k down at current rates:"

User: "Export the final list to CSV"
Agent: [exports current filtered set with all enrichment data]
```

## State Management Rules

1. **Accumulate filters** — each user message adds to the current filter set, doesn't replace it
2. **Track the active filter set** — summarize current filters when showing results
3. **Support removal** — "remove the mileage filter" or "show all drivetrains again"
4. **Support reset** — "start over" or "new search" clears all filters
5. **Preserve enrichment data** — once recalls/payments are fetched for a VIN, keep them

## Filter Display

After each refinement, show the active filters:

```
Active filters: SUV | AWD | Under $50k | Under 30k miles | Toyota, Mazda | FL
Results: 18 vehicles
```

## Pagination

- Show top 10 results by default as a compact table
- "Show more" or "next page" loads the next set
- "Show all" exports to CSV (for large sets, warn about cost first)

## Enrichment Triggers

Certain user questions trigger enrichment API calls. Recognize these patterns:

| User Says | Action |
|-----------|--------|
| "any recalls?" / "are these safe?" | Call /recalls or /openrecalls for each |
| "what's the payment?" / "monthly cost?" | Call /payments for specified vehicles |
| "compare the top N" | Call /specs for each, display side-by-side |
| "tell me more about this one" | Full vehicle report (Pattern 4 from chaining-patterns.md) |
| "is this a good deal?" | Compare price to similar listings (same make/model/year/trim) |
| "how much to own?" / "cost of ownership?" | Call /tco for specified vehicles |
| "what options does it have?" | Call /build for factory build data |
| "show me photos" | Call /photos for images |

## Sort Support

The API does not have a sort parameter — sort client-side after fetching results.

| User Says | Sort By (Client-Side) |
|-----------|----------------------|
| "cheapest first" / "lowest price" | price ascending |
| "most expensive" | price descending |
| "lowest miles" / "newest" | miles ascending |
| "closest to me" | calculate distance from ZIP using location[lon,lat] |
| "best deal" | lowest price relative to similar listings (same make/model/year) |

## Comparison Mode

When user asks to compare 2-5 vehicles:

```
User: "Compare the RAV4 and the CX-5"

Build a comparison table:
| | 2025 Toyota RAV4 XLE | 2025 Mazda CX-5 Turbo |
|---|---|---|
| Price | $34,500 | $36,200 |
| Miles | 12,000 | 8,500 |
| Drivetrain | AWD | AWD |
| Engine | 2.5L 4-Cyl | 2.5L 4-Cyl Turbo |
| HP | 203 | 256 |
| MPG | 30 combined | 27 combined |
| Monthly Payment | $485 | $510 |
| 5yr TCO | $48,200 | $51,300 |
| Open Recalls | None | None |
```

Fetch /specs, /payments, /tco, and /recalls in parallel for both vehicles.

## Cost Awareness

- Listing searches are cheap ($0.002 Starter, $0.0015 Growth, $0.001 Scale) — refine freely
- Enrichment calls add up — estimate before batch operations
- "Check recalls on all 42 results" = 42 × $0.01 = $0.42 on Growth ($0.29 on Scale) — warn first
- Always show estimated cost before enriching more than 10 vehicles
