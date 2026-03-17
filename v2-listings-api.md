# V2 Vehicle Listings API (Starter — $0.002/call Starter, $0.0015/call Growth, $0.001/call Scale)

## Search Listings
**GET** `https://api.auto.dev/listings`

Returns paginated array of vehicle listings.

### Pagination
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `page` | number | 1 | Page number |
| `limit` | number | 100 | Results per page (1-100) |

### Location Filters
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `zip` | string | — | 5-digit ZIP center point |
| `distance` | number | 50 | Radius in miles from ZIP |

### Vehicle Filters
| Param | Type | Description |
|-------|------|-------------|
| `vehicle.make` | string | Manufacturer (comma-separated) |
| `vehicle.model` | string | Model name (comma-separated) |
| `vehicle.year` | string | Year or range: `2022-2025` |
| `vehicle.trim` | string | Trim level (comma-separated) |
| `vehicle.bodyStyle` | string | SUV, Sedan, Truck, etc. |
| `vehicle.engine` | string | Engine size |
| `vehicle.transmission` | string | Automatic, Manual |
| `vehicle.drivetrain` | string | AWD, FWD, RWD, 4WD |
| `vehicle.exteriorColor` | string | Exterior color |
| `vehicle.interiorColor` | string | Interior color |
| `vehicle.doors` | string | 2, 4, or 5 |
| `vehicle.squishVin` | string | First 11 VIN chars |

### Listing Filters
| Param | Type | Description |
|-------|------|-------------|
| `retailListing.price` | string | Price range: `10000-50000` |
| `retailListing.miles` | string | Mileage range |
| `retailListing.state` | string | State abbreviation: `FL`, `CA` |
| `wholesaleListing.buyNowPrice` | string | Wholesale price range |
| `wholesaleListing.state` | string | State abbreviation |
| `wholesaleListing.miles` | string | Mileage range |

### Negation Filters
Append `.not` to exclude values:
`vehicle.fuel.not=*gas*,diesel`

### Response Fields
```json
{
  "data": [{
    "vin": "string",
    "createdAt": "YYYY-MM-DD HH:MM:SS",
    "location": [longitude, latitude],
    "online": true,
    "vehicle": {
      "year": 2026, "make": "string", "model": "string",
      "trim": "string", "series": "string", "style": "string",
      "bodyStyle": "string", "type": "string",
      "drivetrain": "AWD|FWD|RWD|4WD",
      "engine": "string", "cylinders": 6,
      "transmission": "string", "fuel": "string",
      "exteriorColor": "string", "interiorColor": "string",
      "doors": 4, "seats": 8,
      "baseMsrp": 38800, "baseInvoice": 37830,
      "confidence": 0.995, "squishVin": "string"
    },
    "retailListing": {
      "price": 39520, "miles": 5,
      "dealer": "string", "city": "string",
      "state": "FL", "zip": "string",
      "used": false, "cpo": false,
      "vdp": "https://...", "carfaxUrl": "https://...",
      "primaryImage": "https://...", "photoCount": 12
    },
    "wholesaleListing": null,
    "history": null
  }]
}
```

## Get Single Listing
**GET** `https://api.auto.dev/listings/{vin}`

Returns `{ "data": { single listing object } }` with same fields as above.

## Common Patterns

**New cars under $40k in California:**
`/listings?vehicle.year=2025-2026&retailListing.price=1-40000&retailListing.state=CA`

**Used SUVs near Miami:**
`/listings?zip=33132&distance=50&vehicle.bodyStyle=SUV&retailListing.miles=1-50000`

**Exclude gas vehicles:**
`/listings?vehicle.fuel.not=*gas*`
