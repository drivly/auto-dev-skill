# V2 VIN-Based APIs

All endpoints: `https://api.auto.dev/{endpoint}/{vin}`
Auth: `Authorization: Bearer {key}` or `?apiKey={key}`

---

## VIN Decode (Starter)
**GET** `/vin/{vin}`

| Field | Type | Description |
|-------|------|-------------|
| `vin` | string | 17-char VIN |
| `vinValid` | boolean | Format validity |
| `wmi` | string | World Manufacturer ID (first 3 chars) |
| `origin` | string | Manufacturing country |
| `squishVin` | string | VIN without year/check digit |
| `checkDigit` | string | 9th character |
| `checksum` | boolean | Checksum validity |
| `make` | string | Manufacturer |
| `model` | string | Model name |
| `trim` | string | Trim level |
| `style` | string | Body configuration |
| `body` | string | Body class |
| `engine` | string | Engine specs |
| `drive` | string | Drivetrain |
| `transmission` | string | Transmission type |
| `ambiguous` | boolean | Multiple trim possibilities |
| `vehicle.vin` | string | VIN (echoed back) |
| `vehicle.year` | number | Model year |
| `vehicle.make` | string | Manufacturer |
| `vehicle.model` | string | Model name |
| `vehicle.manufacturer` | string | Full manufacturer name |

---

## Vehicle Photos (Starter)
**GET** `/photos/{vin}`

Response:
```json
{
  "data": {
    "retail": [
      "https://api.auto.dev/photos/retail/{vin}-1.jpg",
      "https://api.auto.dev/photos/retail/{vin}-2.jpg"
    ]
  }
}
```
Photos ordered by importance: exterior, interior, engine, details.
Empty array if no photos available.

---

## Specifications (Growth — $0.0015/call)
**GET** `/specs/{vin}`

Returns comprehensive specs including:

| Section | Fields |
|---------|--------|
| `vehicle` | vin, year, make, model, manufacturer |
| `specs.name` | Full trim description |
| `specs.price` | baseMsrp, baseInvoice |
| `specs.totalSeating` | Seat count |
| **Engine** | size, cylinders, horsepower, torque, valves, injection, cam type |
| **Measurements** | length, height, wheelbase, curb weight, gross weight, payload, turning circle, 0-60 |
| **Fuel Economy** | fuel type, combined/city/highway MPG, tank capacity, range |
| **Drivetrain** | drive type, transmission, differential |
| **Colors** | exterior/interior options with RGB hex values |
| **Warranty** | basic, powertrain, roadside, rust coverage |
| **Safety** | features list |

---

## OEM Build Data (Growth — $0.10/call)
**GET** `/build/{vin}`

Factory-installed configuration for a specific VIN.

| Field | Type | Description |
|-------|------|-------------|
| `build.vin` | string | VIN |
| `build.year` | number | Model year |
| `build.make` | string | Manufacturer |
| `build.model` | string | Model |
| `build.trim` | string | Full trim designation |
| `build.series` | string | Vehicle series |
| `build.style` | string | Body style (e.g., "2D Coupe") |
| `build.drivetrain` | string | RWD, AWD, etc. |
| `build.engine` | string | Full spec with HP |
| `build.transmission` | string | Transmission type |
| `build.confidence` | number | Accuracy score 0-1 |
| `build.interiorColor` | object | `{"Color Name": "#hex"}` |
| `build.exteriorColor` | object | `{"Color Name": "#hex"}` |
| `build.options` | object | `{"code": "description"}` |
| `build.optionsMsrp` | number | Total options MSRP in USD |

**Note:** High cost per call ($0.10 Growth, $0.08 Scale). Warn user before batch operations.

---

## Vehicle Recalls (Growth — $0.01/call)
**GET** `/recalls/{vin}`

Response: `{ "data": [array of recall records] }`

| Field | Type | Description |
|-------|------|-------------|
| `manufacturer` | string | Issuing manufacturer |
| `nhtsaCampaignNumber` | string | NHTSA campaign ID |
| `parkIt` | boolean | DO NOT DRIVE until repaired |
| `parkOutSide` | boolean | Fire/explosion risk |
| `overTheAirUpdate` | boolean | Software update fix available |
| `reportReceivedDate` | string | Date (DD/MM/YYYY) |
| `component` | string | Affected component |
| `summary` | string | Issue description |
| `consequence` | string | Safety consequences |
| `remedy` | string | Repair instructions |
| `notes` | string | Contact info |
| `modelYear` | string | Affected year |
| `make` | string | Make |
| `model` | string | Model |

Empty `data: []` means no recalls.

---

## Open Recalls (Scale — $0.06/call)
**GET** `/openrecalls/{vin}`

Same fields as Recalls plus:

| Field | Type | Description |
|-------|------|-------------|
| `recallStatus` | string | Open / In Progress / Remedy Available |
| `expectedRemediationDate` | string | Fix availability date |

Only returns **unresolved** recalls. Use this to check if a vehicle is safe to buy.

---

## Total Cost of Ownership (Growth — $0.06/call)
**GET** `/tco/{vin}`

| Query Param | Type | Required | Description |
|-------------|------|----------|-------------|
| `zip` | string | No | ZIP for insurance/tax calc (auto-detected) |
| `fromZip` | string | No | ZIP for delivery/transport calc |

Response:
```json
{
  "vehicle": { "vin": "string", "year": 2024, "make": "string", "model": "string" },
  "tco": {
    "total": {
      "federalTaxCredit": 0,
      "insurance": 3824,
      "maintenance": 5499,
      "repairs": 1446,
      "taxesAndFees": 5027,
      "financeInterest": 12376,
      "depreciation": 39258,
      "fuel": 12482,
      "tcoPrice": 79912,
      "averageCostPerMile": 1.07
    },
    "years": {
      "1": { "insurance": 713, "maintenance": 257, "repairs": 0, "taxesAndFees": 4843, "financeInterest": 4241, "depreciation": 18763, "fuel": 2351, "tcoPrice": 31168, "averageCostPerMile": 1.07 },
      "2": { "...same fields..." },
      "3": {},
      "4": {},
      "5": {}
    }
  }
}
```

---

## Vehicle Payments (Growth — $0.005/call)
**GET** `/payments/{vin}`

| Query Param | Type | Required | Default | Description |
|-------------|------|----------|---------|-------------|
| `price` | string | Yes | — | Sale price |
| `zip` | string | Yes | — | ZIP for tax jurisdiction |
| `docFee` | number | No | — | Documentation fee |
| `tradeIn` | string | No | 0 | Trade-in value |
| `downPayment` | number | No | — | Down payment |
| `loanTerm` | number | No | 60 | Term in months |

Response:
```json
{
  "vehicle": { "vin": "string", "year": 2021, "make": "string", "model": "string" },
  "paymentsData": {
    "loanAmount": 20000,
    "loanMonthlyPayment": 380,
    "loanMonthlyPaymentWithTaxes": 425,
    "totalTaxesAndFees": 2700
  },
  "taxes": {
    "combinedSalesTax": 1500,
    "stateSalesTax": 1200,
    "gasGuzzlerTax": 0
  },
  "fees": {
    "titleFee": 75,
    "registrationFee": 225,
    "dmvFee": 50,
    "docFee": 200,
    "combinedFees": 550,
    "dmvFees": [
      { "name": "Filing Fee", "amount": 10 },
      { "name": "License Plate Fee", "amount": 25 }
    ]
  },
  "apr": {
    "36": 7.2, "48": 7.3, "60": 7.5, "72": 7.7, "84": 8.2
  }
}
```

---

## Interest Rates (Growth — $0.005/call)
**GET** `/apr/{vin}`

| Query Param | Type | Required | Description |
|-------------|------|----------|-------------|
| `year` | number | Yes | Model year |
| `make` | string | Yes | Manufacturer |
| `model` | string | Yes | Model name |
| `zip` | string | Yes | 5-digit ZIP |
| `creditScore` | string | Yes | Buyer credit score |
| `vehicleAge` | number | No | Age in years |
| `vehicleMileage` | string | No | Current mileage |

Response:
```json
{
  "vehicle": { "vin": "string", "year": 2021, "make": "string", "model": "string" },
  "zip": "37129",
  "creditScore": "720",
  "apr": {
    "36": 7.644, "48": 7.644, "60": 7.524,
    "72": 7.663, "84": 8.263
  }
}
```

---

## Taxes & Fees (Scale — $0.005/call)
**GET** `/taxes/{vin}`

| Query Param | Type | Description |
|-------------|------|-------------|
| `price` | number | Sale price |
| `zip` | string | Tax jurisdiction ZIP |
| `docFee` | number | Documentation fee |
| `tradeIn` | number | Trade-in value (deducted from taxable) |
| `rate` | number | Interest rate |
| `downPayment` | number | Down payment |
| `months` | number | Loan term |

Response includes `totalTaxesAndFees`, itemized state/county/city/district taxes, and itemized fees (title, registration, DMV, doc).

---

## Error Codes (All Endpoints)

| Status | Code | Meaning |
|--------|------|---------|
| 400 | `INVALID_VIN_FORMAT` | VIN not 17 chars or contains I/O/Q |
| 400 | `INVALID_PARAMETER` | Unknown query parameter |
| 400 | `INVALID_LOCATION` | Bad ZIP format |
| 404 | `VIN_NOT_FOUND` | No data for this VIN |
| 404 | `RESOURCE_NOT_FOUND` | Resource doesn't exist |
| 503 | `SOURCE_ERROR` | Upstream service temporarily down |
