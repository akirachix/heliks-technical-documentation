# Metals-API External Price Feed Integration

This reference spec documents the integration layer connecting the value engine to external pricing mechanisms.

##  External Price Fetching Routine
The calculation engine connects to **Metals-API** via standard synchronous HTTP requests to gather live commodities market prices for copper assets.

### Integration Properties:
* **API Endpoint Base:** `https://metals-api.com`
* **Target Symbol Token:** `XCU` (LME Copper Asset Notation)
* **Historical Validation Limit:** If an optional `price_date` parameter is supplied to calculations, the backend validates that the requested query window sits strictly **within the last 60 days**. Older requests are rejected via a `422 Unprocessable Entity` response.
