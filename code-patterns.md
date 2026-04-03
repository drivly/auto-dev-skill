## SDK (Recommended for JS/TS projects)

Install:

```bash
npm install @auto.dev/sdk
```

Initialize:

```typescript
import { AutoDev } from '@auto.dev/sdk'

const auto = new AutoDev({ apiKey: process.env.AUTODEV_API_KEY })
```

Core methods:

```typescript
// VIN decode
const vehicle = await auto.decode('1HGCM82633A004352')

// Search listings
const listings = await auto.listings({
  'vehicle.make': 'Toyota',
  'vehicle.year': '2024',
  'retailListing.price': '10000-40000',
  'retailListing.state': 'FL',
})

// Payment calculation
const payments = await auto.payments('1HGCM82633A004352', {
  price: 35000,
  zip: '90210',
  downPayment: 5000,
  loanTerm: 60,
})

// Interest rates
const apr = await auto.apr('1HGCM82633A004352', {
  year: 2024,
  make: 'Honda',
  model: 'Accord',
  zip: '90210',
  creditScore: '750',
})

// Total cost of ownership
const tco = await auto.tco('1HGCM82633A004352', { zip: '90210' })

// Plate to VIN
const plate = await auto.plate('CA', 'ABC1234')

// Taxes and fees
const taxes = await auto.taxes('1HGCM82633A004352', { price: 35000, zip: '90210' })

// Account usage
const usage = await auto.usage()
```

All methods return:

```typescript
{
  data: T,
  meta: {
    requestId: string,
    tier: string,
    usage?: { remaining: number }
  }
}
```

Use the SDK for all new JS/TS projects. Fall back to raw fetch only when
the SDK is unavailable or the user is using a non-JS environment.

---

# Code Patterns

Framework-specific patterns for integrating Auto.dev APIs. When a developer asks to build something, generate complete, working code using these patterns — not pseudocode.

## Authentication Setup

### Environment Variable (All Frameworks)

```bash
# .env or .env.local
AUTODEV_API_KEY=sk_ad_your_key_here
```

### Node.js / Next.js Helper

```typescript
// lib/autodev.ts
const AUTODEV_V2_BASE = 'https://api.auto.dev';
const AUTODEV_V1_BASE = 'https://auto.dev/api';

async function autodevFetch<T>(
  endpoint: string,
  params?: Record<string, string>,
  version: 'v1' | 'v2' = 'v2'
): Promise<T> {
  const base = version === 'v1' ? AUTODEV_V1_BASE : AUTODEV_V2_BASE;
  const url = new URL(endpoint, base);

  if (params) {
    Object.entries(params).forEach(([k, v]) => url.searchParams.set(k, v));
  }

  const apiKey = process.env.AUTODEV_API_KEY;
  if (!apiKey) throw new Error('AUTODEV_API_KEY environment variable is not set');

  // V1 uses query string auth, V2 uses Bearer header
  if (version === 'v1') {
    url.searchParams.set('apikey', apiKey);
  }

  const res = await fetch(url.toString(), {
    headers: {
      ...(version === 'v2' ? { 'Authorization': `Bearer ${apiKey}` } : {}),
      'Content-Type': 'application/json',
    },
  });

  if (!res.ok) {
    const error = await res.json();
    throw new AutodevError(error.error?.status, error.error?.code, error.error?.error);
  }

  return res.json();
}

class AutodevError extends Error {
  constructor(
    public status: number,
    public code: string,
    message: string
  ) {
    super(message);
    this.name = 'AutodevError';
  }
}

export { autodevFetch, AutodevError, AUTODEV_V2_BASE, AUTODEV_V1_BASE };
```

### Python Helper

```python
# autodev.py
import os
import requests
from dataclasses import dataclass

AUTODEV_V2_BASE = 'https://api.auto.dev'
AUTODEV_V1_BASE = 'https://auto.dev/api'


@dataclass
class AutodevError(Exception):
    status: int
    code: str
    message: str


def autodev_fetch(endpoint: str, params: dict = None, version: str = 'v2') -> dict:
    base = AUTODEV_V1_BASE if version == 'v1' else AUTODEV_V2_BASE
    api_key = os.environ.get('AUTODEV_API_KEY')
    if not api_key:
        raise ValueError('AUTODEV_API_KEY environment variable is not set')

    # V1 uses query string auth, V2 uses Bearer header
    headers = {'Content-Type': 'application/json'}
    query_params = dict(params or {})

    if version == 'v1':
        query_params['apikey'] = api_key
    else:
        headers['Authorization'] = f'Bearer {api_key}'

    response = requests.get(f'{base}{endpoint}', headers=headers, params=query_params)

    if not response.ok:
        error = response.json().get('error', {})
        raise AutodevError(
            status=error.get('status', response.status_code),
            code=error.get('code', 'UNKNOWN'),
            message=error.get('error', 'Unknown error'),
        )

    return response.json()
```

---

## TypeScript Types

Generate these types when building TypeScript projects with Auto.dev:

```typescript
// types/autodev.ts

interface Vehicle {
  year: number;
  make: string;
  model: string;
  trim: string;
  series: string;
  style: string;
  bodyStyle: string;
  type: string;
  drivetrain: 'AWD' | 'FWD' | 'RWD' | '4WD';
  engine: string;
  cylinders: number;
  transmission: string;
  fuel: string;
  exteriorColor: string;
  interiorColor: string;
  doors: number;
  seats: number;
  baseMsrp: number;
  baseInvoice: number;
  confidence: number;
  squishVin: string;
  vin: string;
}

interface RetailListing {
  price: number;
  miles: number;
  dealer: string;
  city: string;
  state: string;
  zip: string;
  used: boolean;
  cpo: boolean;
  vdp: string;
  carfaxUrl: string;
  primaryImage: string;
  photoCount: number;
}

interface Listing {
  '@id': string;
  vin: string;
  createdAt: string;
  location: [number, number]; // [longitude, latitude]
  online: boolean;
  vehicle: Vehicle;
  retailListing: RetailListing | null;
  wholesaleListing: WholesaleListing | null;
  history: unknown | null;
}

interface ListingsResponse {
  data: Listing[];
  links: {
    self: string;
    first: string;
    prev: string;
    next: string;
  };
}

interface VinDecode {
  vin: string;
  vinValid: boolean;
  wmi: string;
  origin: string;
  squishVin: string;
  checkDigit: string;
  checksum: boolean;
  make: string;
  model: string;
  trim: string;
  style: string;
  body: string;
  engine: string;
  drive: string;
  transmission: string;
  ambiguous: boolean;
  vehicle: {
    year: number;
    manufacturer: string;
  };
}

interface PaymentsResponse {
  vehicle: { vin: string; year: number; make: string; model: string };
  paymentsData: {
    loanAmount: number;
    loanMonthlyPayment: number;
    loanMonthlyPaymentWithTaxes: number;
    totalTaxesAndFees: number;
  };
  taxes: {
    combinedSalesTax: number;
    stateSalesTax: number;
    gasGuzzlerTax: number;
  };
  fees: {
    titleFee: number;
    registrationFee: number;
    dmvFee: number;
    docFee: number;
    combinedFees: number;
    dmvFees: Array<{ name: string; amount: number }>;
  };
  apr: Record<string, number>;
}

interface RecallRecord {
  manufacturer: string;
  nhtsaCampaignNumber: string;
  parkIt: boolean;
  parkOutSide: boolean;
  overTheAirUpdate: boolean;
  reportReceivedDate: string;
  component: string;
  summary: string;
  consequence: string;
  remedy: string;
  notes: string;
  modelYear: string;
  make: string;
  model: string;
}

interface TcoCostBreakdown {
  federalTaxCredit?: number;
  insurance: number;
  maintenance: number;
  repairs: number;
  taxesAndFees: number;
  financeInterest: number;
  depreciation: number;
  fuel: number;
  tcoPrice: number;
  averageCostPerMile: number;
}

interface TcoResponse {
  vehicle: { vin: string; year: number; make: string; model: string };
  tco: {
    total: TcoCostBreakdown;
    years: {
      '1': TcoCostBreakdown;
      '2': TcoCostBreakdown;
      '3': TcoCostBreakdown;
      '4': TcoCostBreakdown;
      '5': TcoCostBreakdown;
    };
  };
}

interface AprResponse {
  vehicle: { vin: string; year: number; make: string; model: string };
  zip: string;
  creditScore: string;
  apr: Record<string, number>;
}

interface PlateResponse {
  vin: string;
  year: number;
  make: string;
  model: string;
  trim: string;
  drivetrain: string;
  engine: string;
  transmission: string;
  isDefault: boolean;
}

interface BuildData {
  build: {
    vin: string;
    year: number;
    make: string;
    model: string;
    trim: string;
    series: string;
    style: string;
    drivetrain: string;
    engine: string;
    transmission: string;
    confidence: number;
    interiorColor: Record<string, string>;
    exteriorColor: Record<string, string>;
    options: Record<string, string>;
    optionsMsrp: number;
  };
}

interface PhotosResponse {
  data: {
    retail: string[];
  };
}

interface WholesaleListing {
  buyNowPrice: number;
  miles: number;
  state: string;
}

export type {
  Vehicle, RetailListing, WholesaleListing, Listing, ListingsResponse,
  VinDecode, PaymentsResponse, RecallRecord, TcoResponse, AprResponse,
  PlateResponse, BuildData, PhotosResponse,
};
```

---

## Next.js Patterns

### API Route (App Router)

```typescript
// app/api/listings/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { autodevFetch, AutodevError } from '@/lib/autodev';
import type { ListingsResponse } from '@/types/autodev';

export async function GET(request: NextRequest) {
  const { searchParams } = request.nextUrl;

  const params: Record<string, string> = {};
  for (const [key, value] of searchParams.entries()) {
    params[key] = value;
  }

  try {
    const data = await autodevFetch<ListingsResponse>('/listings', params);
    return NextResponse.json(data);
  } catch (error) {
    if (error instanceof AutodevError) {
      return NextResponse.json(
        { error: error.message, code: error.code },
        { status: error.status }
      );
    }
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 });
  }
}
```

### Server Action

```typescript
// app/actions/vehicles.ts
'use server';

import { autodevFetch } from '@/lib/autodev';
import type { ListingsResponse, VinDecode } from '@/types/autodev';

export async function searchListings(filters: {
  make?: string;
  model?: string;
  priceRange?: string;
  state?: string;
  zip?: string;
  distance?: string;
  page?: string;
}) {
  const params: Record<string, string> = {};
  if (filters.make) params['vehicle.make'] = filters.make;
  if (filters.model) params['vehicle.model'] = filters.model;
  if (filters.priceRange) params['retailListing.price'] = filters.priceRange;
  if (filters.state) params['retailListing.state'] = filters.state;
  if (filters.zip) params['zip'] = filters.zip;
  if (filters.distance) params['distance'] = filters.distance;
  if (filters.page) params['page'] = filters.page;

  return autodevFetch<ListingsResponse>('/listings', params);
}

export async function decodeVin(vin: string) {
  return autodevFetch<VinDecode>(`/vin/${vin}`);
}
```

### React Component

```tsx
// components/VehicleSearch.tsx
'use client';

import { useState, useTransition } from 'react';
import { searchListings } from '@/app/actions/vehicles';
import type { Listing } from '@/types/autodev';

export function VehicleSearch() {
  const [results, setResults] = useState<Listing[]>([]);
  const [isPending, startTransition] = useTransition();

  function handleSearch(formData: FormData) {
    startTransition(async () => {
      const data = await searchListings({
        make: formData.get('make') as string,
        model: formData.get('model') as string,
        priceRange: `1-${formData.get('maxPrice')}`,
        state: formData.get('state') as string,
      });
      setResults(data.data);
    });
  }

  return (
    <div>
      <form action={handleSearch}>
        <input name="make" placeholder="Make (e.g., Toyota)" />
        <input name="model" placeholder="Model (e.g., Camry)" />
        <input name="maxPrice" type="number" placeholder="Max Price" />
        <input name="state" placeholder="State (e.g., FL)" maxLength={2} />
        <button type="submit" disabled={isPending}>
          {isPending ? 'Searching...' : 'Search'}
        </button>
      </form>

      <div>
        {results.map((listing) => (
          <div key={listing.vin}>
            <h3>{listing.vehicle.year} {listing.vehicle.make} {listing.vehicle.model}</h3>
            <p>{listing.vehicle.trim} — {listing.vehicle.drivetrain}</p>
            <p>${listing.retailListing?.price?.toLocaleString()}</p>
            <p>{listing.retailListing?.dealer} — {listing.retailListing?.city}, {listing.retailListing?.state}</p>
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## Express / Node.js Pattern

```typescript
// routes/vehicles.ts
import { Router } from 'express';
import { autodevFetch } from '../lib/autodev';
import type { ListingsResponse, VinDecode, PaymentsResponse } from '../types/autodev';

const router = Router();

router.get('/search', async (req, res) => {
  try {
    const { make, model, maxPrice, state, zip, distance, page } = req.query;

    const params: Record<string, string> = {};
    if (make) params['vehicle.make'] = make as string;
    if (model) params['vehicle.model'] = model as string;
    if (maxPrice) params['retailListing.price'] = `1-${maxPrice}`;
    if (state) params['retailListing.state'] = state as string;
    if (zip) params['zip'] = zip as string;
    if (distance) params['distance'] = distance as string;
    if (page) params['page'] = page as string;

    const data = await autodevFetch<ListingsResponse>('/listings', params);
    res.json(data);
  } catch (error) {
    res.status(error.status || 500).json({ error: error.message });
  }
});

router.get('/vin/:vin', async (req, res) => {
  try {
    const data = await autodevFetch<VinDecode>(`/vin/${req.params.vin}`);
    res.json(data);
  } catch (error) {
    res.status(error.status || 500).json({ error: error.message });
  }
});

router.get('/payments/:vin', async (req, res) => {
  try {
    const { price, zip, downPayment, loanTerm } = req.query;
    const data = await autodevFetch<PaymentsResponse>(
      `/payments/${req.params.vin}`,
      { price, zip, downPayment, loanTerm } as Record<string, string>
    );
    res.json(data);
  } catch (error) {
    res.status(error.status || 500).json({ error: error.message });
  }
});

export default router;
```

---

## Python / Flask Pattern

```python
# routes/vehicles.py
from flask import Blueprint, request, jsonify
from autodev import autodev_fetch, AutodevError

vehicles = Blueprint('vehicles', __name__)


@vehicles.route('/search')
def search_listings():
    try:
        params = {}
        if request.args.get('make'):
            params['vehicle.make'] = request.args['make']
        if request.args.get('model'):
            params['vehicle.model'] = request.args['model']
        if request.args.get('max_price'):
            params['retailListing.price'] = f"1-{request.args['max_price']}"
        if request.args.get('state'):
            params['retailListing.state'] = request.args['state']
        if request.args.get('zip'):
            params['zip'] = request.args['zip']

        data = autodev_fetch('/listings', params)
        return jsonify(data)
    except AutodevError as e:
        return jsonify({'error': e.message, 'code': e.code}), e.status


@vehicles.route('/vin/<vin>')
def decode_vin(vin):
    try:
        data = autodev_fetch(f'/vin/{vin}')
        return jsonify(data)
    except AutodevError as e:
        return jsonify({'error': e.message, 'code': e.code}), e.status


@vehicles.route('/payments/<vin>')
def calculate_payments(vin):
    try:
        params = {
            'price': request.args['price'],
            'zip': request.args['zip'],
        }
        if request.args.get('downPayment'):
            params['downPayment'] = request.args['downPayment']
        if request.args.get('loanTerm'):
            params['loanTerm'] = request.args['loanTerm']

        data = autodev_fetch(f'/payments/{vin}', params)
        return jsonify(data)
    except AutodevError as e:
        return jsonify({'error': e.message, 'code': e.code}), e.status
```

---

## Python Data Analysis Pattern

```python
# analysis.py
import pandas as pd
from autodev import autodev_fetch


def listings_to_dataframe(make: str, model: str, state: str, max_price: int) -> pd.DataFrame:
    """Fetch listings and return as a pandas DataFrame for analysis."""
    all_listings = []
    page = 1

    while True:
        data = autodev_fetch('/listings', {
            'vehicle.make': make,
            'vehicle.model': model,
            'retailListing.state': state,
            'retailListing.price': f'1-{max_price}',
            'page': str(page),
        })

        listings = data.get('data', [])
        if not listings:
            break

        for item in listings:
            v = item.get('vehicle', {}) or {}
            r = item.get('retailListing', {}) or {}
            all_listings.append({
                'vin': item.get('vin'),
                'year': v.get('year'),
                'make': v.get('make'),
                'model': v.get('model'),
                'trim': v.get('trim'),
                'drivetrain': v.get('drivetrain'),
                'engine': v.get('engine'),
                'exterior_color': v.get('exteriorColor'),
                'interior_color': v.get('interiorColor'),
                'price': r.get('price'),
                'miles': r.get('miles'),
                'dealer': r.get('dealer'),
                'city': r.get('city'),
                'state': r.get('state'),
                'cpo': r.get('cpo'),
            })

        page += 1

    return pd.DataFrame(all_listings)


# Usage:
# df = listings_to_dataframe('Toyota', 'RAV4', 'FL', 40000)
# df.describe()  # price stats
# df.groupby('trim')['price'].mean()  # avg price by trim
# df.to_csv('rav4_fl.csv', index=False)  # export
```

---

## Key Principles for Generated Code

1. **Always use server-side API calls** — never expose AUTODEV_API_KEY in client-side code
2. **Use the autodev helper** — consistent auth, error handling, base URL management
3. **Include TypeScript types** — generate from types/autodev.ts for type safety
4. **Handle errors gracefully** — catch AutodevError and return meaningful messages
5. **Paginate for large result sets** — loop through pages for batch operations
6. **Respect rate limits** — add delays for batch operations (Starter: 5/s, Growth: 10/s, Scale: 50/s)
