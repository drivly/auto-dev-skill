# Integration Recipes

Patterns for connecting Auto.dev APIs with external services to build monitoring, alerting, and automation workflows.

**Note:** Auto.dev is a REST API platform — it does not support webhooks, streaming, or push notifications. All integrations use a polling pattern (scheduled API calls on an interval).

## Recipe 1: Daily Inventory Alert (Node.js + Cron)

Monitor for new listings matching criteria and send notifications.

```typescript
// jobs/inventory-monitor.ts
import { autodevFetch } from '../lib/autodev';
import type { ListingsResponse, Listing } from '../types/autodev';

interface MonitorConfig {
  name: string;
  filters: Record<string, string>;
  notify: (listings: Listing[]) => Promise<void>;
}

async function checkForNewListings(config: MonitorConfig, lastChecked: string) {
  const data = await autodevFetch<ListingsResponse>('/listings', config.filters);

  const newListings = data.data.filter(
    (listing) => listing.createdAt > lastChecked
  );

  if (newListings.length > 0) {
    await config.notify(newListings);
  }

  return { found: newListings.length, checkedAt: new Date().toISOString() };
}

// Example config:
const monitor: MonitorConfig = {
  name: 'BMW X5 Under 50k',
  filters: {
    'vehicle.make': 'BMW',
    'vehicle.model': 'X5',
    'retailListing.price': '1-50000',
    'retailListing.state': 'FL',
  },
  notify: async (listings) => {
    // Send to Slack, email, etc.
    console.log(`Found ${listings.length} new BMW X5 listings!`);
  },
};
```

### Cron Setup

```bash
# crontab -e
# Run every 6 hours
0 */6 * * * node /path/to/jobs/inventory-monitor.js
```

### GitHub Actions (Serverless Cron)

```yaml
# .github/workflows/inventory-check.yml
name: Inventory Monitor
on:
  schedule:
    - cron: '0 */6 * * *'  # every 6 hours
  workflow_dispatch:

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: node jobs/inventory-monitor.js
        env:
          AUTODEV_API_KEY: ${{ secrets.AUTODEV_API_KEY }}
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

## Recipe 2: Slack Notifications

```typescript
// notifications/slack.ts
import type { Listing } from '../types/autodev';

async function sendSlackAlert(webhookUrl: string, listings: Listing[]) {
  const blocks = listings.slice(0, 5).map((listing) => ({
    type: 'section',
    text: {
      type: 'mrkdwn',
      text: [
        `*${listing.vehicle.year} ${listing.vehicle.make} ${listing.vehicle.model}*`,
        `${listing.vehicle.trim} | ${listing.vehicle.drivetrain}`,
        `Price: $${listing.retailListing?.price?.toLocaleString()}`,
        `Miles: ${listing.retailListing?.miles?.toLocaleString()}`,
        `Dealer: ${listing.retailListing?.dealer}`,
        `Location: ${listing.retailListing?.city}, ${listing.retailListing?.state}`,
        `<${listing.retailListing?.vdp}|View Listing>`,
      ].join('\n'),
    },
    accessory: listing.retailListing?.primaryImage ? {
      type: 'image',
      image_url: listing.retailListing.primaryImage,
      alt_text: `${listing.vehicle.year} ${listing.vehicle.make} ${listing.vehicle.model}`,
    } : undefined,
  }));

  await fetch(webhookUrl, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      text: `Found ${listings.length} new listings!`,
      blocks: [
        {
          type: 'header',
          text: { type: 'plain_text', text: `🚗 ${listings.length} New Listings Found` },
        },
        ...blocks,
      ],
    }),
  });
}

export { sendSlackAlert };
```

## Recipe 3: Email Digest (SendGrid)

```typescript
// notifications/email.ts
import sgMail from '@sendgrid/mail';
import type { Listing } from '../types/autodev';

sgMail.setApiKey(process.env.SENDGRID_API_KEY!);

async function sendEmailDigest(to: string, listings: Listing[], searchName: string) {
  const rows = listings.map((l) => `
    <tr>
      <td>${l.vehicle.year} ${l.vehicle.make} ${l.vehicle.model}</td>
      <td>${l.vehicle.trim}</td>
      <td>$${l.retailListing?.price?.toLocaleString()}</td>
      <td>${l.retailListing?.miles?.toLocaleString()} mi</td>
      <td>${l.retailListing?.city}, ${l.retailListing?.state}</td>
      <td><a href="${l.retailListing?.vdp}">View</a></td>
    </tr>
  `).join('');

  await sgMail.send({
    to,
    from: 'alerts@yourdomain.com',
    subject: `${searchName}: ${listings.length} new listings`,
    html: `
      <h2>${searchName}</h2>
      <p>Found ${listings.length} new listings matching your criteria.</p>
      <table border="1" cellpadding="8" cellspacing="0">
        <tr>
          <th>Vehicle</th><th>Trim</th><th>Price</th>
          <th>Miles</th><th>Location</th><th>Link</th>
        </tr>
        ${rows}
      </table>
    `,
  });
}

export { sendEmailDigest };
```

## Recipe 4: Webhook Receiver (Express)

For systems that push data to your server:

```typescript
// routes/webhooks.ts
import { Router } from 'express';
import { autodevFetch } from '../lib/autodev';

const router = Router();

// Receive a VIN from an external system, enrich it, and forward
router.post('/enrich-vin', async (req, res) => {
  const { vin, callbackUrl } = req.body;

  try {
    const [decode, specs, recalls] = await Promise.all([
      autodevFetch(`/vin/${vin}`),
      autodevFetch(`/specs/${vin}`).catch(() => null),
      autodevFetch(`/recalls/${vin}`).catch(() => ({ data: [] })),
    ]);

    const enriched = { vin, decode, specs, recalls };

    if (callbackUrl) {
      await fetch(callbackUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(enriched),
      });
    }

    res.json(enriched);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

export default router;
```

## Recipe 5: Google Sheets Integration (Apps Script)

```javascript
// Google Apps Script — Tools > Script editor
function searchAutodev() {
  const API_KEY = PropertiesService.getScriptProperties().getProperty('AUTODEV_API_KEY');
  const sheet = SpreadsheetApp.getActiveSheet();

  const make = sheet.getRange('B1').getValue();
  const model = sheet.getRange('B2').getValue();
  const maxPrice = sheet.getRange('B3').getValue();
  const state = sheet.getRange('B4').getValue();

  const url = `https://api.auto.dev/listings`
    + `?vehicle.make=${encodeURIComponent(make)}`
    + `&vehicle.model=${encodeURIComponent(model)}`
    + `&retailListing.price=1-${maxPrice}`
    + `&retailListing.state=${state}`
    + `&apiKey=${API_KEY}`;

  const response = UrlFetchApp.fetch(url);
  const data = JSON.parse(response.getContentText());

  // Clear previous results
  const resultsRange = sheet.getRange('A6:J500');
  resultsRange.clear();

  // Headers
  const headers = ['VIN', 'Year', 'Make', 'Model', 'Trim', 'Price', 'Miles', 'Dealer', 'City', 'State'];
  sheet.getRange('A5:J5').setValues([headers]);

  // Data rows
  const rows = data.data.map(function(item) {
    var v = item.vehicle || {};
    var r = item.retailListing || {};
    return [item.vin, v.year, v.make, v.model, v.trim, r.price, r.miles, r.dealer, r.city, r.state];
  });

  if (rows.length > 0) {
    sheet.getRange(6, 1, rows.length, 10).setValues(rows);
  }
}

// Add menu item
function onOpen() {
  SpreadsheetApp.getUi()
    .createMenu('Auto.dev')
    .addItem('Search Listings', 'searchAutodev')
    .addToUi();
}
```

## Recipe 6: Zapier / Make.com Webhook

Create a webhook endpoint that Zapier/Make can call:

```typescript
// routes/zapier.ts
import { Router } from 'express';
import { autodevFetch } from '../lib/autodev';

const router = Router();

// Zapier calls this with search criteria, gets back listings
router.get('/zapier/search', async (req, res) => {
  const params: Record<string, string> = {};

  // Map Zapier-friendly param names to Auto.dev params
  const mapping: Record<string, string> = {
    make: 'vehicle.make',
    model: 'vehicle.model',
    max_price: 'retailListing.price',
    state: 'retailListing.state',
    zip: 'zip',
    max_miles: 'retailListing.miles',
  };

  for (const [zapierKey, autodevKey] of Object.entries(mapping)) {
    if (req.query[zapierKey]) {
      const value = req.query[zapierKey] as string;
      if (zapierKey === 'max_price') {
        params[autodevKey] = `1-${value}`;
      } else if (zapierKey === 'max_miles') {
        params[autodevKey] = `1-${value}`;
      } else {
        params[autodevKey] = value;
      }
    }
  }

  const data = await autodevFetch('/listings', params);

  // Flatten for Zapier compatibility
  const flat = data.data.map((item: any) => ({
    vin: item.vin,
    year: item.vehicle?.year,
    make: item.vehicle?.make,
    model: item.vehicle?.model,
    trim: item.vehicle?.trim,
    price: item.retailListing?.price,
    miles: item.retailListing?.miles,
    dealer: item.retailListing?.dealer,
    city: item.retailListing?.city,
    state: item.retailListing?.state,
    listing_url: item.retailListing?.vdp,
    image_url: item.retailListing?.primaryImage,
  }));

  res.json(flat);
});

export default router;
```

## Recipe 7: Python CLI Tool

```python
#!/usr/bin/env python3
# cli.py — pip install click requests
import click
import csv
import sys
from autodev import autodev_fetch


@click.group()
def cli():
    """Auto.dev CLI — search vehicles from your terminal."""
    pass


@cli.command()
@click.option('--make', required=True, help='Vehicle make')
@click.option('--model', help='Vehicle model')
@click.option('--max-price', type=int, help='Maximum price')
@click.option('--state', help='State abbreviation')
@click.option('--zip', 'zipcode', help='ZIP code')
@click.option('--output', type=click.Choice(['table', 'csv', 'json']), default='table')
def search(make, model, max_price, state, zipcode, output):
    """Search vehicle listings."""
    params = {'vehicle.make': make}
    if model:
        params['vehicle.model'] = model
    if max_price:
        params['retailListing.price'] = f'1-{max_price}'
    if state:
        params['retailListing.state'] = state
    if zipcode:
        params['zip'] = zipcode

    data = autodev_fetch('/listings', params)
    listings = data.get('data', [])

    if output == 'json':
        import json
        click.echo(json.dumps(listings, indent=2))
    elif output == 'csv':
        writer = csv.writer(sys.stdout)
        writer.writerow(['VIN', 'Year', 'Make', 'Model', 'Trim', 'Price', 'Miles', 'City', 'State'])
        for item in listings:
            v = item.get('vehicle', {})
            r = item.get('retailListing', {})
            writer.writerow([item['vin'], v.get('year'), v.get('make'), v.get('model'),
                           v.get('trim'), r.get('price'), r.get('miles'), r.get('city'), r.get('state')])
    else:
        for item in listings:
            v = item.get('vehicle', {})
            r = item.get('retailListing', {})
            click.echo(f"{v.get('year')} {v.get('make')} {v.get('model')} {v.get('trim')}")
            click.echo(f"  ${r.get('price', 0):,} | {r.get('miles', 0):,} mi | {r.get('city')}, {r.get('state')}")
            click.echo()


@cli.command()
@click.argument('vin')
def decode(vin):
    """Decode a VIN."""
    data = autodev_fetch(f'/vin/{vin}')
    click.echo(f"{data.get('vehicle', {}).get('year')} {data.get('make')} {data.get('model')}")
    click.echo(f"Trim: {data.get('trim')}")
    click.echo(f"Engine: {data.get('engine')}")
    click.echo(f"Drive: {data.get('drive')}")
    click.echo(f"Origin: {data.get('origin')}")


if __name__ == '__main__':
    cli()
```
