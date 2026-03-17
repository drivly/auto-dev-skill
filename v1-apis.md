# V1 APIs (Supplemental)

**Always prefer V2 APIs.** V1 endpoints below exist only because they have no V2 equivalent. For VIN decoding, listings, and photos, use V2.

Base URL: `https://auto.dev/api`
Auth: `?apikey={key}` (query string, lowercase `apikey`)

---

## Models
**GET** `/models`

Returns all makes and their models in the database. No parameters.

```json
{
  "Toyota": ["Camry", "Corolla", "RAV4", "Tacoma", "..."],
  "Ford": ["F-150", "Mustang", "Explorer", "..."],
  "BMW": ["3 Series", "5 Series", "X3", "X5", "..."]
}
```

65+ manufacturers. Use this to validate make/model before searching listings or to build dropdowns in UI.

---

## V1 VIN Decode
**GET** `/vin/{vin}`

Legacy decode. Returns more data in one call but **prefer V2** `/vin` + `/specs` + `/build` for new projects. V1 decode may be useful as a fallback if you need options/colors/MPG without separate Growth-tier calls.

| Section | Fields |
|---------|--------|
| `years` | Array of year objects with styles and IDs |
| `transmission` | Type, speeds, equipment type, availability |
| `engine` | HP, torque, displacement, cylinders, fuel type, compression ratio |
| `make` / `model` | Manufacturer and model with nice names |
| `price` | MSRP, delivery charges, TMV flag |
| `categories` | Vehicle type, style, size, body, market, EPA class |
| `mpg` | City and highway fuel economy |
| `colors` | Available color options by category |
| `options` | Equipment and features by category |
| `drivenWheels` | Drive configuration |
| `squishVin` | Shortened VIN |

**Prefer V2 for new projects.** Only use v1 decode as a quick fallback when you need a single-call response.

---

## V1 Listings
**GET** `/listings`

Legacy listings endpoint. Response differs from v2:

| Field | Description |
|-------|-------------|
| `totalCount` | Total results |
| `records` | Array of listing objects |
| `records[].year`, `make`, `model`, `trim` | Vehicle info |
| `records[].price`, `priceUnformatted` | Pricing |
| `records[].monthlyPayment` | Estimated payment |
| `records[].state`, `city`, `lat`, `lon` | Location |
| `records[].distanceFromOrigin` | Miles from search point |
| `records[].dealerName` | Dealer |
| `records[].primaryPhotoUrl`, `photoUrls` | Images |
| `records[].active`, `isHot`, `recentPriceDrop` | Status flags |
| `records[].condition` | New/Used |

**Always use V2 `/listings` for new projects** — more filters, structured response, and better pagination. V1 listings is legacy only.

---

## Cities
**GET** `/cities`

Returns all US cities organized by state. No parameters.

```json
{
  "success": true,
  "payload": {
    "fl": { "miami": "Miami", "orlando": "Orlando", "tampa": "Tampa" },
    "ca": { "losangeles": "Los Angeles", "sandiego": "San Diego" },
    "tx": { "houston": "Houston", "dallas": "Dallas", "austin": "Austin" }
  }
}
```

All 50 states. Useful for building location pickers or validating city names before listings search.

---

## ZIP Lookup
**GET** `/zip/{zip}`

| Param | Location | Type | Description |
|-------|----------|------|-------------|
| `zip` | path | string | 5-digit ZIP code |

```json
{
  "success": true,
  "payload": {
    "zip": "33132",
    "state": "FL",
    "city": "Miami",
    "longitude": -80.1867,
    "latitude": 25.7839,
    "dma": { "name": "Miami-Ft. Lauderdale", "code": "528" },
    "defaultSearchRadius": 50
  }
}
```

Use to convert ZIP to coordinates for geo queries, or to get DMA market info.

---

## Autosuggest
**GET** `/autosuggest/{term}`

| Param | Location | Type | Description |
|-------|----------|------|-------------|
| `term` | path | string | Search term (e.g., "cam", "ford f") |

```json
{
  "recordTypes": {
    "makemodel": [{
      "make": "Toyota",
      "model": "Camry",
      "vehicle": {
        "publicationStates": {
          "new": ["2025", "2026"],
          "used": ["2018", "2019", "2020", "..."]
        }
      },
      "inventory": {
        "newYears": ["2025", "2026"],
        "usedYears": ["2018", "2019", "2020"]
      }
    }],
    "fuzzyMakemodel": []
  }
}
```

Returns exact and fuzzy matches with available inventory years. Use to power search-as-you-type or to validate make/model input.
