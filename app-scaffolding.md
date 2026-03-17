# App Scaffolding

When a developer asks to build a complete application with Auto.dev, use these templates as starting points. Generate the full project structure with working code.

## Template 1: Used Car Search App (Next.js)

**User asks:** "Build me a used car search app" or "Create a vehicle marketplace"

### Project Structure
```
car-search/
  app/
    layout.tsx              — root layout with nav
    page.tsx                — home page with search form
    results/
      page.tsx              — search results grid
    vehicle/
      [vin]/
        page.tsx            — vehicle detail page
    api/
      listings/
        route.ts            — proxy to Auto.dev listings
      vin/
        [vin]/
          route.ts          — proxy to Auto.dev VIN decode
      payments/
        [vin]/
          route.ts          — proxy to Auto.dev payments
  components/
    SearchForm.tsx           — make/model/price/location filters
    VehicleCard.tsx          — listing card with image, price, specs
    VehicleGrid.tsx          — responsive grid of VehicleCards
    ComparisonTable.tsx      — side-by-side vehicle comparison
    PaymentCalculator.tsx    — interactive payment estimator
    RecallBadge.tsx          — safety indicator
    Pagination.tsx           — page navigation
  lib/
    autodev.ts              — API client (from code-patterns.md)
  types/
    autodev.ts              — TypeScript types (from code-patterns.md)
  .env.local                — AUTODEV_API_KEY
```

### Key Pages

**Search Page** — form with:
- Make dropdown (populated from v1 /models)
- Model dropdown (filtered by selected make)
- Price range slider
- Year range
- Body style checkboxes
- ZIP + distance radius
- State selector

**Results Page** — displays:
- Result count and active filters
- Sort options (price, miles, newest) — client-side sort, API has no sort param
- Vehicle cards in responsive grid
- Pagination
- "Compare" checkbox on each card
- "Export to CSV" button

**Vehicle Detail Page** — shows:
- Photo gallery (from /photos)
- Full specs table (from /specs)
- Factory build data (from /build)
- Recall status with safety badges (from /recalls)
- Payment calculator widget (from /payments)
- Price comparison vs similar listings
- TCO breakdown chart (from /tco)
- Similar vehicles section

### Critical Implementation Notes
- All Auto.dev API calls go through API routes — never expose key client-side
- Use React Server Components for initial data fetch
- Client components only for interactive elements (filters, calculator)
- Implement loading skeletons for API-dependent sections
- Cache VIN decode and specs data (rarely changes)
- Don't cache listings or prices (change frequently)

---

## Template 2: Dealer Inventory Dashboard (Next.js)

**User asks:** "Build a dashboard for a car dealership" or "inventory management tool"

### Project Structure
```
dealer-dashboard/
  app/
    layout.tsx
    page.tsx                — dashboard overview
    inventory/
      page.tsx              — full inventory list with filters
    vehicle/
      [vin]/
        page.tsx            — single vehicle management
    analytics/
      page.tsx              — pricing and inventory analytics
    recalls/
      page.tsx              — recall monitoring
    api/
      ...                   — API proxy routes
  components/
    Dashboard/
      StatsCards.tsx         — inventory count, avg price, avg days
      AgingChart.tsx         — inventory aging distribution
      PriceDistribution.tsx  — price histogram
    Inventory/
      InventoryTable.tsx     — sortable, filterable table
      QuickActions.tsx       — price check, recall check buttons
    Analytics/
      MarketComparison.tsx   — your price vs market
      TrendChart.tsx         — price trends over time
      CompetitorGrid.tsx     — nearby dealer comparison
```

### Dashboard Features

**Overview Cards:**
- Total active listings
- Average listing price
- Average days on lot
- Vehicles with open recalls
- Vehicles priced above market

**Inventory Aging Chart:**
- Color-coded bars: green (fresh) → red (critical)
- Click to drill into aging bracket

**Market Position:**
- Your avg price vs market avg (from listings search)
- Per-model comparison
- Suggested price adjustments for stale inventory

---

## Template 3: Vehicle Comparison Tool (React SPA)

**User asks:** "Build a car comparison tool" or "help buyers compare vehicles"

### Project Structure
```
vehicle-compare/
  src/
    App.tsx
    pages/
      ComparePage.tsx        — main comparison view
    components/
      VinInput.tsx           — VIN entry with validation
      VehicleColumn.tsx      — single vehicle data column
      SpecsComparison.tsx    — specs row-by-row comparison
      CostComparison.tsx     — TCO and payments comparison
      SafetyComparison.tsx   — recalls side-by-side
      WinnerBadge.tsx        — highlights better value per row
    hooks/
      useVehicleData.ts      — fetches and caches vehicle data
    api/
      server.ts              — Express backend for API proxy
```

### Comparison Features

**Input:** 2-4 VINs or search-and-select

**Comparison Rows:**
- Basic info (year, make, model, trim)
- Price and market value
- Engine (HP, torque, cylinders)
- Fuel economy (city/hwy/combined)
- Dimensions and seating
- Drivetrain and transmission
- Monthly payment (same terms)
- 5-year TCO
- Recall count and severity
- Factory options
- Color-coded "winner" per row

---

## Template 4: VIN Scanner / Lookup Tool (Mobile-First)

**User asks:** "Build a VIN lookup app" or "VIN scanner tool"

### Project Structure
```
vin-lookup/
  app/
    page.tsx                — VIN input (type or paste)
    result/
      [vin]/
        page.tsx            — full vehicle report
    history/
      page.tsx              — recent lookups (localStorage)
    api/
      ...
  components/
    VinInput.tsx            — large input with validation feedback
    VehicleReport.tsx       — comprehensive single-vehicle view
    ReportSection.tsx       — collapsible section (specs, recalls, etc.)
    ShareButton.tsx         — share report link
    HistoryList.tsx         — recent VIN lookups
```

### Mobile UX
- Large touch-friendly VIN input field
- Real-time VIN validation as user types
- Auto-uppercase, strip spaces/dashes
- Collapsible report sections to manage screen space
- Swipe between report sections
- Share report via link

---

## Template 5: Price Alert Service (Node.js Backend)

**User asks:** "Build a price alert system" or "notify me when prices drop"

### Project Structure
```
price-alerts/
  src/
    index.ts                — Express server
    routes/
      alerts.ts             — CRUD for alert subscriptions
      webhooks.ts           — incoming webhook handlers
    jobs/
      price-checker.ts      — scheduled price monitoring
      alert-sender.ts       — notification dispatch
    models/
      alert.ts              — alert subscription model
    lib/
      autodev.ts            — API client
      notifications.ts      — email/slack/webhook senders
    types/
      index.ts
  prisma/
    schema.prisma           — database schema
```

### Database Schema
```prisma
model Alert {
  id        String   @id @default(cuid())
  email     String
  make      String
  model     String
  maxPrice  Int
  state     String?
  zip       String?
  distance  Int?
  active    Boolean  @default(true)
  lastCheck DateTime?
  createdAt DateTime @default(now())
}

model PriceSnapshot {
  id       String   @id @default(cuid())
  alertId  String
  vin      String
  price    Int
  date     DateTime @default(now())
  alert    Alert    @relation(fields: [alertId], references: [id])
}
```

### Job Flow
```
Every 6 hours:
  1. Load all active alerts
  2. For each alert, search listings with alert criteria
  3. Compare results to previous snapshots
  4. Identify: new listings, price drops, removed listings
  5. Send notifications for significant changes
  6. Store new snapshots
```

---

## Common Patterns Across All Templates

**API Key Security:**
- Never include API key in client-side code
- Always proxy through server-side API routes
- Use environment variables for key storage

**Error Boundaries:**
- Wrap Auto.dev API calls in try/catch
- Show user-friendly error messages
- Suggest plan upgrades when hitting access errors

**Performance:**
- Cache static data (specs, build data) aggressively
- Don't cache dynamic data (listings, prices)
- Use parallel API calls where possible
- Implement pagination for listing searches

**SEO (for public-facing apps):**
- Use Server Components for initial render
- Generate meta tags from vehicle data
- Structured data (JSON-LD) for vehicle listings
