# V2 Plate-to-VIN API (Scale — $0.55/call)

**GET** `https://api.auto.dev/plate/{state}/{plate}`

Converts a US license plate to VIN and basic vehicle info.

**Note:** Highest per-call cost ($0.55). Always warn user before batch operations.

## Parameters

| Param | Location | Type | Required | Description |
|-------|----------|------|----------|-------------|
| `state` | path | string | Yes | 2-letter state code (CA, NY, TX) |
| `plate` | path | string | Yes | License plate number |

## Supported Plate Formats
- Standard: `ABC123`, `123ABC`, `AB1234`
- Vanity/personalized plates
- Commercial plates (truck, taxi, fleet)
- Hyphenated: `AB-123`

## Response

```json
{
  "vin": "1N4BL4BV3LC205823",
  "year": 2020,
  "make": "Nissan",
  "model": "Altima",
  "trim": "2.5 S Sedan 4D",
  "drivetrain": "FWD",
  "engine": "4-Cyl, 2.5 Liter",
  "transmission": "Automatic, Xtronic CVT",
  "isDefault": true
}
```

`isDefault: true` means this is the base configuration — trim may have multiple possibilities.

## Common Pattern

After resolving plate to VIN, chain to other endpoints:
```
/plate/FL/ABC123 → get VIN → /specs/{vin} + /openrecalls/{vin}
```
