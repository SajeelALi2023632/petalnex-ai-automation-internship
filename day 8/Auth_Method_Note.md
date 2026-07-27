# Day 8 Deliverable — Note on Authentication Method

**Workflow:** Weather Notification Automation
**Trigger schedule:** Every morning at 9:00 AM
**Location monitored:** Gilgit, Northern Areas, Pakistan
**API used:** WeatherAPI.com — `https://api.weatherapi.com/v1/forecast.json`

## Authentication method: API Key (via Query Parameter)

WeatherAPI.com authenticates every request using a single API key passed as
a query string parameter, `key=<your_api_key>`, appended directly to the
request URL:

```
https://api.weatherapi.com/v1/forecast.json?key=XXXXXXXX&q=Gilgit&days=1&aqi=no&alerts=no
```

This is the simplest of the four common patterns covered in Day 8 (API key,
Bearer token, Basic auth, OAuth) — the key just needs to be present
somewhere in the request. Here it travels in the query string rather than
as a header.

## How it was stored (never hard-coded)

Instead of pasting the key into the HTTP Request node's URL or query
fields, it was stored as an **n8n credential**:

1. On the HTTP Request node: **Authentication → Generic Credential Type →
   Query Auth**.
2. Created a **Query Auth** credential named "Query Auth account" with:
   - **Name:** `key`
   - **Value:** `<the WeatherAPI.com API key, entered as a masked secret>`
3. n8n automatically appends this as a `key=...` query parameter on every
   request that node makes, pulling from the credential vault instead of
   from plain text in the node's configuration.

Because of this, the exported workflow JSON contains **no visible API
key** — only a reference to a credential ID that must be re-created
locally after import.

## Troubleshooting note (for reference)

During setup, the request initially failed with `"API key is invalid"`
(WeatherAPI error code 2006). This turned out to be a key-provisioning
issue on WeatherAPI.com's side rather than an n8n configuration problem —
confirmed by testing the same key directly in a browser and getting the
identical error. Re-checking the key on the WeatherAPI.com dashboard and
re-copying it into the credential resolved the issue; the node now
returns a full JSON response (`location`, `current`, `forecast`) on
execution.

## Why this matters
- The key never appears in version control, screenshots, or exported JSON.
- If the key is ever compromised, it can be rotated in one place (n8n
  credentials) without touching the workflow logic.
- This mirrors how Bearer tokens or Basic auth would be handled for other
  APIs — same principle, different credential type.
