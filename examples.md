# Real-World Examples

Actual API responses and agent workflows from live Auto.dev calls. Use these to understand what real output looks like and how to format results for users.

## Example 1: "Find Mazda CX-90s in Florida under $60k"

**API Call:**
```
GET https://api.auto.dev/listings?vehicle.make=Mazda&vehicle.model=CX-90&retailListing.price=1-60000&retailListing.state=FL
```

**Real Response (first listing):**
```json
{
  "vin": "JM3KKAHD5T1379650",
  "createdAt": "2026-03-13 13:27:01",
  "location": [-82.369, 28.246],
  "online": true,
  "vehicle": {
    "baseMsrp": 38800,
    "baseInvoice": 37830,
    "bodyStyle": "SUV",
    "confidence": 0.995,
    "cylinders": 6,
    "doors": 4,
    "drivetrain": "AWD",
    "engine": "3.3L 6Cyl regular unleaded",
    "exteriorColor": "Black",
    "fuel": "Regular Unleaded",
    "interiorColor": "Black",
    "make": "Mazda",
    "model": "CX-90",
    "seats": 8,
    "series": "3.3 Turbo Select 4dr SUV AWD (3.3L 6cyl Turbo gas/electric mild hybrid 8A)",
    "transmission": "Automatic",
    "trim": "3.3 Turbo Select",
    "type": "Crossover",
    "year": 2026
  },
  "retailListing": {
    "carfaxUrl": "https://www.carfax.com/VehicleHistory/p/Report.cfx?vin=JM3KKAHD5T1379650&partner=FRD_2",
    "city": "Wesley Chapel",
    "dealer": "Mazda of Wesley Chapel",
    "miles": 5,
    "photoCount": 0,
    "price": 39520,
    "primaryImage": "https://retail.photos.vin/JM3KKAHD5T1379650-1.jpg",
    "state": "FL",
    "vdp": "https://www.lithia.com/catcher.esl?vin=JM3KKAHD5T1379650"
  }
}
```

**How to present this (inline for small result sets):**

```
Found 100 Mazda CX-90s under $60k in Florida. Here are the top 10 by price:

| Year | Trim | Price | Miles | Color | City | Dealer |
|------|------|-------|-------|-------|------|--------|
| 2025 | 3.3 Turbo Select | $25,832 | 18,420 | Gray | Tampa | Ferman Mazda |
| 2025 | 3.3 Turbo Select | $27,495 | 22,105 | White | Orlando | Sport Mazda |
| 2026 | 3.3 Turbo Select | $38,290 | 12 | Red | Miami | South Motors |
| 2026 | 3.3 Turbo Select | $39,520 | 5 | Black | Wesley Chapel | Mazda of Wesley Chapel |
| ... | | | | | | |

7 trims available: Turbo Select, Turbo Preferred, Premium Plus, Premium Sport,
S Premium, S Premium Plus, S Premium Sport

Want me to filter further, compare specific ones, or export to CSV?
```

## Example 2: "Decode VIN JM3KKAHD5T1379650"

**API Call:**
```
GET https://api.auto.dev/vin/JM3KKAHD5T1379650
```

**Real Response (key fields):**
```json
{
  "vin": "JM3KKAHD5T1379650",
  "vinValid": true,
  "wmi": "JM3",
  "origin": "Japan",
  "squishVin": "JM3KKAHDT1",
  "checkDigit": "5",
  "checksum": true,
  "make": "Mazda",
  "model": "CX-90",
  "trim": "3.3 Turbo Select",
  "style": "SUV 4D 3.3, Straight 6 Cylinder Engine",
  "body": "SUV 4D",
  "engine": "3.3, Straight 6 Cylinder Engine",
  "drive": "All Wheel Drive",
  "transmission": "Manual",
  "vehicle": {
    "year": 2026,
    "make": "Mazda",
    "model": "CX-90",
    "manufacturer": "Mazda Motor Corp"
  }
}
```

**How to present this:**

```
2026 Mazda CX-90 — 3.3 Turbo Select

VIN: JM3KKAHD5T1379650 (valid, checksum OK)
Origin: Japan (Mazda Motor Corp)
Body: SUV 4D
Engine: 3.3L Straight 6 Cylinder
Drivetrain: All Wheel Drive
MSRP: $38,800

Want me to pull full specs, check for recalls, or calculate payments?
```

## Example 3: "What are the monthly payments with $5k down?"

**API Call:**
```
GET https://api.auto.dev/payments/JM3KKAHD5T1379650?price=39520&zip=33132&downPayment=5000&loanTerm=60
```

**Real Response (key fields):**
```json
{
  "vehicle": {
    "vin": "JM3KKAHD5T1379650",
    "year": 2026,
    "make": "Mazda",
    "model": "CX-90"
  },
  "paymentsData": {
    "loanAmount": 37904.40,
    "loanMonthlyPayment": 588.43,
    "loanMonthlyPaymentWithTaxes": 646.12,
    "totalTaxesAndFees": 3384.40
  },
  "apr": {
    "36": 7.116,
    "48": 7.116,
    "60": 6.987,
    "72": 6.994,
    "84": 7.594
  },
  "taxes": {
    "combinedSalesTax": 2766.40,
    "stateSalesTax": 2371.20,
    "countySalesTax": 395.20,
    "gasGuzzlerTax": 0
  },
  "fees": {
    "titleFee": 77.25,
    "registrationFee": 312.75,
    "dmvFee": 28.00,
    "docFee": 200.00,
    "combinedFees": 618.00
  }
}
```

**How to present this:**

```
2026 Mazda CX-90 3.3 Turbo Select — Payment Breakdown

Price: $39,520
Down Payment: -$5,000
Loan Amount: $37,904

| Term | APR | Monthly (P+I) | Monthly (w/ tax & fees) |
|------|-----|---------------|-------------------------|
| 36 mo | 7.12% | $1,173 | $1,267 |
| 48 mo | 7.12% | $909 | $982 |
| 60 mo | 6.99% | $751 | $811 |
| 72 mo | 6.99% | $646 | $698 |
| 84 mo | 7.59% | $580 | $626 |

Taxes & Fees (Miami-Dade, FL):
  Sales Tax: $2,766 (7% combined)
  Title: $77 | Registration: $313 | DMV: $28 | Doc: $200
  Total Taxes & Fees: $3,384
```

## Example 4: "Check recalls on this Ford Explorer"

**API Call:**
```
GET https://api.auto.dev/recalls/1FMUK7DHXSGA27345
```

**Real Response (first of 15 recalls):**
```json
{
  "manufacturer": "Ford Motor Company",
  "nhtsaCampaignNumber": "24V634000",
  "parkIt": false,
  "parkOutSide": false,
  "overTheAirUpdate": false,
  "reportReceivedDate": "23/08/2024",
  "component": "FUEL SYSTEM, GASOLINE:FUEL INJECTION SYSTEM:INJECTORS",
  "summary": "Ford Motor Company (Ford) is recalling certain 2023 Lincoln Corsair, 2023-2024 Escape, 2024 Mustang, and 2025 Explorer vehicles. The fuel injectors may have been assembled without O-ring support discs, which can result in a fuel leak.",
  "consequence": "A fuel leak in the presence of an ignition source increases the risk of a fire.",
  "remedy": "Dealers will install new fuel injector O-rings and O-ring support discs, free of charge."
}
```

**How to present this:**

```
2025 Ford Explorer — 15 Recalls Found

⚠️ SAFETY ALERTS:
  None with "Park It" or "Park Outside" flags — vehicle is safe to drive.

Recent Recalls:
1. Fuel Injectors (24V634000) — Aug 2024
   Issue: Fuel injectors may lack O-ring support discs, causing fuel leak
   Risk: Fire hazard
   Fix: Dealer installs new O-rings, free of charge

2. [additional recalls...]

Want me to check if any are still unresolved? (Requires Scale plan for Open Recalls API)
```

## Example 5: "Tell me the 5-year cost of owning this Wagoneer"

**API Call:**
```
GET https://api.auto.dev/tco/1C4SJVBP5RS114977?zip=33132
```

**Real Response:**
```json
{
  "vehicle": {
    "vin": "1C4SJVBP5RS114977",
    "year": 2024,
    "make": "Wagoneer",
    "model": "Wagoneer"
  },
  "tco": {
    "total": {
      "depreciation": 39258,
      "fuel": 12482,
      "insurance": 3824,
      "maintenance": 5499,
      "repairs": 1446,
      "taxesAndFees": 5027,
      "financeInterest": 12376,
      "tcoPrice": 79912,
      "averageCostPerMile": 1.07
    },
    "years": {
      "1": { "depreciation": 18763, "fuel": 2351, "insurance": 713, "maintenance": 257, "repairs": 0, "taxesAndFees": 4843, "financeInterest": 4241, "tcoPrice": 31168 },
      "2": { "depreciation": 4989, "fuel": 2422, "insurance": 738, "maintenance": 720, "repairs": 0, "taxesAndFees": 46, "financeInterest": 3421, "tcoPrice": 12336 },
      "3": { "depreciation": 4720, "fuel": 2494, "insurance": 764, "maintenance": 369, "repairs": 212, "taxesAndFees": 46, "financeInterest": 2541, "tcoPrice": 11146 },
      "4": { "depreciation": 5540, "fuel": 2569, "insurance": 791, "maintenance": 2433, "repairs": 503, "taxesAndFees": 46, "financeInterest": 1594, "tcoPrice": 13476 },
      "5": { "depreciation": 5246, "fuel": 2646, "insurance": 818, "maintenance": 1720, "repairs": 731, "taxesAndFees": 46, "financeInterest": 579, "tcoPrice": 11786 }
    }
  }
}
```

**How to present this:**

```
2024 Wagoneer — 5-Year Total Cost of Ownership (Miami, FL)

Total 5-Year Cost: $79,912 ($1.07/mile)

| Category | Year 1 | Year 2 | Year 3 | Year 4 | Year 5 | Total |
|----------|--------|--------|--------|--------|--------|-------|
| Depreciation | $18,763 | $4,989 | $4,720 | $5,540 | $5,246 | $39,258 |
| Fuel | $2,351 | $2,422 | $2,494 | $2,569 | $2,646 | $12,482 |
| Finance Interest | $4,241 | $3,421 | $2,541 | $1,594 | $579 | $12,376 |
| Maintenance | $257 | $720 | $369 | $2,433 | $1,720 | $5,499 |
| Taxes & Fees | $4,843 | $46 | $46 | $46 | $46 | $5,027 |
| Insurance | $713 | $738 | $764 | $791 | $818 | $3,824 |
| Repairs | $0 | $0 | $212 | $503 | $731 | $1,446 |
| **Annual Total** | **$31,168** | **$12,336** | **$11,146** | **$13,476** | **$11,786** | **$79,912** |

Key insights:
- Year 1 is the most expensive ($31k) due to steep initial depreciation
- Depreciation is the biggest cost ($39k, 49% of total)
- Maintenance spikes in years 4-5 as warranty coverage ends
```

## Example 6: Full Listings-to-CSV Export

**User:** "Export all Mazda CX-90s under $60k in Florida to CSV"

**What the agent does:**
1. Calls `/listings?vehicle.make=Mazda&vehicle.model=CX-90&retailListing.price=1-60000&retailListing.state=FL`
2. Pages through results if more than 100
3. Flattens nested JSON to CSV columns
4. Saves to user's specified path

**CSV columns generated:**
```
VIN, Year, Make, Model, Trim, Series, Body_Style, Type,
Exterior_Color, Interior_Color, Engine, Cylinders, Transmission,
Drivetrain, Fuel, Doors, Seats, Price, Miles, Dealer, City, State,
VDP_URL, Carfax_URL, Primary_Image, Photo_Count, Base_MSRP,
Base_Invoice, Created_At
```

**Agent response:**
```
Saved 50 Mazda CX-90 listings to mazda_cx90_fl_listings.csv

Summary:
- Price range: $25,832 – $58,870
- 7 trims found
- 7 cities across Florida
- 29 columns per vehicle
```

## Quickstart: Verify Your Setup

After installing the skill and setting your API key, try this to confirm everything works:

> "Decode VIN WP0AA2A92PS206106"

Expected: Your agent should return details for a 2019 Porsche 911 GT3 RS, manufactured in Germany. If you see vehicle data, your API key and skill are working correctly.
