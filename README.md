# Customer Data Export API Documentation

## Overview

This API lets customers export meter data for a given meter (`location_id`) over an optional time range. Readings are returned newest-first (by `rtst`, descending). The response shape depends on the meter's device type (see [Response](#response)).

## Endpoint

### URL

```plaintext
https://api.oleaedge.com/functions/v1/export
```

### Method

```plaintext
POST
```

## Headers

- **Authorization**: `Bearer <token>`
  Required. This is the Olea project's public **anon** key (the same value for every customer). It authorizes the request at the gateway; per-customer access is controlled by `api_key` in the body.

  ```plaintext
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imt4cWh6bmhtcml1cml2bXZoYWV2Iiwicm9sZSI6ImFub24iLCJpYXQiOjE2OTExMTM4NjEsImV4cCI6MjAwNjY4OTg2MX0.Z__6-D1y2CSEebkf-m-l_0vK0U5-QL22-xRLV-59LZk
  ```

- **Content-Type**: `application/json`

## Request Body

A JSON object with the following parameters:

- **api_key** *(String, required)* — Your customer API key. Provided by Olea. Determines which meters you may access.

- **location_id** *(String, required)* — The unique identifier (UUID) for the meter. Provided by Olea.

  ```json
  "location_id": "2a9ad530-0c3b-11f1-8934-23f3ea42ce58"
  ```

- **start_date** *(String, optional)* — Dates are interpreted as UTC unless you include a timezone offset.

  ```json
  "start_date": "2024-08-07"
  ```

  For a local-midnight boundary, pass an ISO 8601 value with offset, e.g. Central Daylight Time (UTC-5):

  ```json
  "start_date": "2025-08-07T00:00:00-05:00"
  ```

- **end_date** *(String, optional)* — Same UTC/offset rules as `start_date`.

  ```json
  "end_date": "2024-08-10"
  ```

### Date range behavior

| start_date | end_date | Window returned |
|------------|----------|-----------------|
| omitted    | omitted  | the single most recent reading |
| provided   | omitted  | `start_date` → `start_date + 7 days` |
| omitted    | provided | `end_date − 7 days` → `end_date` |
| provided   | provided | `start_date` → `end_date` |

**Maximum window per request:** **31 days** for OMR meters (7 days for PRV). A larger range returns `400 Date range cannot be more than N days`. To pull a longer history, page through it in ≤31-day requests.

### Example Request

```bash
curl -L -X POST 'https://api.oleaedge.com/functions/v1/export' \
-H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Imt4cWh6bmhtcml1cml2bXZoYWV2Iiwicm9sZSI6ImFub24iLCJpYXQiOjE2OTExMTM4NjEsImV4cCI6MjAwNjY4OTg2MX0.Z__6-D1y2CSEebkf-m-l_0vK0U5-QL22-xRLV-59LZk' \
-H 'Content-Type: application/json' \
--data '{
    "api_key": "<provided by Olea>",
    "location_id": "2a9ad530-0c3b-11f1-8934-23f3ea42ce58",
    "start_date": "2024-08-07",
    "end_date": "2024-08-10"
}'
```

## Response

On success the API returns `{ "ok": true, "data": [ ... ] }`, with `data` ordered newest-first. The fields in each element depend on the meter's device type.

### OMR meters

Each element contains the reading timestamp, the validated meter value, and the meter's current device status:

- **rtst** — reading timestamp (UTC)
- **ocr_value_manual** — the validated meter reading
- **device_status** — current status of the device (e.g. `"IN SERVICE"`, `"MAINTENANCE"`)

```json
{
  "ok": true,
  "data": [
    { "rtst": "2026-06-08T06:59:42.000Z", "ocr_value_manual": 334380870, "device_status": "IN SERVICE" },
    { "rtst": "2026-06-07T06:59:32.000Z", "ocr_value_manual": 332116750, "device_status": "IN SERVICE" }
  ]
}
```

Notes for OMR:
- Only **validated** readings are returned. Periods with no validated reading return no rows.
- A request returns up to **1,000** readings (the newest in the window). Use a narrower date range to retrieve older readings.

### PRV meters

```json
{
  "ok": true,
  "data": [
    { "rtst": "2024-08-07T12:52:28.000Z", "head1_value": "37", "head2_value": "155" },
    { "rtst": "2024-08-07T12:51:28.000Z", "head1_value": "35", "head2_value": "154" }
  ]
}
```

PRV responses support the optional **page** *(Integer)* parameter: pass `page` to retrieve a batch of up to 1,000 records and increment it until the response returns `{"ok":true,"data":[]}`. (`page` is ignored for OMR meters.)

## Error Handling

The API returns an error object and an appropriate status code.

### Common Errors

- **401 Unauthorized** — Missing or invalid `Authorization` bearer token.
- **400 Bad Request** — Invalid parameters, including: `Missing api_key`, `Missing location_id`, `Invalid api_key`, `Invalid location_id` (the meter isn't accessible to your `api_key`), an invalid date format, or a date range over the allowed maximum.
- **500 Internal Server Error** — An error occurred on the server.
