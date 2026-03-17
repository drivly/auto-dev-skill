# Error Recovery & Input Validation

Proactively validate and correct inputs before making API calls to avoid wasted calls and confusing errors.

## Model Name Normalization

Auto.dev model names use specific formats. Common mismatches:

| User Input | Correct Format | Notes |
|-----------|----------------|-------|
| `cx90`, `cx 90`, `CX90` | `CX-90` | Mazda SUVs use hyphens |
| `cx5`, `cx 5`, `CX5` | `CX-5` | Same pattern |
| `cx30`, `cx 30`, `CX30` | `CX-30` | Same pattern |
| `cx50`, `cx 50`, `CX50` | `CX-50` | Same pattern |
| `rav4`, `RAV 4` | `RAV4` | Toyota — no hyphen, no space |
| `cr-v`, `CRV`, `cr v` | `CR-V` | Honda — with hyphen |
| `hr-v`, `HRV` | `HR-V` | Honda — with hyphen |
| `f150`, `F 150` | `F-150` | Ford trucks use hyphen |
| `f250`, `f-250` | `F-250` | Same pattern |
| `model 3`, `Model3` | `Model 3` | Tesla — with space |
| `model y`, `ModelY` | `Model Y` | Same pattern |
| `3 series`, `3-series` | `3 Series` | BMW — space, no hyphen |
| `c class`, `c-class` | `C-Class` | Mercedes — with hyphen |
| `grand cherokee` | `Grand Cherokee` | Jeep — title case |
| `wrangler` | `Wrangler` | Title case |

**Strategy:** When a listings search returns empty results:
1. Try the v1 `/autosuggest/{term}` endpoint to find the correct model name
2. Try the v1 `/models` endpoint to list all models for that make
3. Retry with the corrected name
4. Tell the user what correction was made

## VIN Validation

Validate before calling any VIN endpoint:

```
Rules:
- Exactly 17 characters
- Only alphanumeric (A-Z, 0-9)
- Cannot contain I, O, or Q (ambiguous with 1, 0)
- 9th character is the check digit
- First 3 characters = World Manufacturer ID (WMI)
```

**Common mistakes:**
- User includes spaces: `JM3 KKAHD 5T1379650` → strip spaces
- User includes dashes: `JM3-KKAHD-5T1379650` → strip dashes
- Lowercase: `jm3kkahd5t1379650` → uppercase
- Too short (partial VIN): warn and ask for full 17 characters
- Contains O/I/Q: flag the specific character and ask user to verify

**Apply corrections silently** for whitespace/case. **Ask the user** for ambiguous characters or wrong length.

## ZIP Code Validation

```
Rules:
- Exactly 5 digits
- Must be numeric
- Cannot be 00000
```

**When ZIP is needed but not provided:**
1. Check if user mentioned a city/state instead
2. Use v1 `/cities` to find the city
3. Suggest a representative ZIP for that area
4. Or ask the user for their ZIP

## State Abbreviation Normalization

| User Input | Correct |
|-----------|---------|
| `Florida`, `florida` | `FL` |
| `California`, `california`, `Cali` | `CA` |
| `New York`, `new york`, `NY` | `NY` |
| `Texas`, `texas` | `TX` |

Always convert full state names to 2-letter abbreviations before API calls.

## Price Range Formatting

Auto.dev expects format: `min-max`

| User Input | Correct Format |
|-----------|----------------|
| "under $30k" | `1-30000` |
| "under 30000" | `1-30000` |
| "$20k to $40k" | `20000-40000` |
| "around $25,000" | `20000-30000` (suggest range) |
| "25000" (single number) | Ask: max price or exact? |

Always use `1` as minimum, not `0` — avoids edge cases.

## Empty Results Recovery

When a search returns `"data": []`:

1. **Check model name** — most common cause. Use autosuggest to verify.
2. **Broaden filters** — remove one filter at a time:
   - Remove state filter (search nationwide)
   - Widen price range
   - Remove trim filter
   - Remove color filter
3. **Try v1 listings** — v1 and v2 may index differently
4. **Report what was tried** — tell user which filters were loosened

## API Error Recovery

| Error | Recovery |
|-------|----------|
| `INVALID_PARAMETER` | Check param name against docs. Common: `rows` doesn't exist, use `limit` |
| `INVALID_VIN_FORMAT` | Apply VIN validation rules above, correct and retry |
| `VIN_NOT_FOUND` | VIN is valid format but no data exists. Try v1 decode as fallback. |
| `RESOURCE_NOT_FOUND` | Endpoint or resource doesn't exist. Check URL construction. |
| `SOURCE_ERROR` (503) | Upstream service down. Wait 5 seconds and retry once. If still failing, inform user. |
| Rate limited (429) | Back off. Starter: wait 200ms. Growth: wait 100ms. Scale: wait 20ms. |
| Permission denied | User's plan doesn't include this endpoint. Show upgrade link from SKILL.md. |

## Body Style Normalization

| User Input | Correct Filter |
|-----------|---------------|
| "SUV", "suv" | `SUV` |
| "truck", "pickup" | `Truck` |
| "sedan", "car" | `Sedan` |
| "coupe", "2-door" | `Coupe` |
| "van", "minivan" | `Van` or `Minivan` |
| "wagon", "estate" | `Wagon` |
| "convertible", "drop top" | `Convertible` |
| "hatchback", "hatch" | `Hatchback` |
| "crossover" | `Crossover` (or try `SUV` — some listed as SUV) |

## Drivetrain Normalization

| User Input | Correct Filter |
|-----------|---------------|
| "all wheel drive", "all-wheel" | `AWD` |
| "four wheel drive", "4x4", "4wd" | `4WD` |
| "front wheel drive", "fwd" | `FWD` |
| "rear wheel drive", "rwd" | `RWD` |
