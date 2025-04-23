# Customer Data Export API Documentation

## Overview

This API allows customers to export data based on specific parameters such as `api_key`, `location_id`, `start_date`, `end_date`, and `page`. The data returned includes values related to specific timestamps (`rtst`), head values, and expression values. The API is designed to handle paginated requests and can return data in batches of 1,000 records.

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

- **Authorization**: Bearer Token
  This token is required to authenticate the request.

  Example:
  ```plaintext
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZS1kZW1vIiwicm9sZSI6ImFub24iLCJleHAiOjE5ODM4MTI5OTZ9.CRXP1A7WOeoJeXxjNni43kdQwgnWNReilDMblYTn_I0
  ```

- **Content-Type**: application/json
  This header specifies that the request body is in JSON format.

## Request Body

The request body must be a JSON object with the following parameters:

- **api_key**: *(String)*
  The API key used for authentication.

  Example:
  ```json
  "api_key": *(Provided By Olea)*
  ```

- **location_id**: *(String)*
  The unique identifier for the meter.

  Example:
  ```json
  "location_id": "354bdd20-cce7-11ee-9f32-678c43cfd720"
  ```

- **start_date**: *(String, Optional)*
  All dates are stored in UTC time in the Olea database so unless you specify a
  start_date with a time zone, the data will be returned in UTC.

  If start_date is not provided, the most recent data is returned.

  #### Example returning data from 2024-08-07T00:00:00.000Z:
  ```json
  "start_date": "2024-08-07"
  ```

  If you need data starting at midnight of your time zone, you will want to
  provide a date in ISO 8601 format with the time zone offset.

  #### Example for Central Daylight Time (CDT) which is UTC-5:
  ```json
  "start_date": "2025-08-07T00:00:00-05:00"
  ```

- **end_date**: *(String, Optional)*
  All dates are stored in UTC time in the Olea database so unless you specify an
  end_date with a time zone, the data will be returned in UTC.

  If provided with a start_date, data between `start_date` and `end_date` is returned.

  If provided without a start_date, the start_date is set to 7 days prior to the end_date.

  Example returning data before 2024-08-10T00:00:00.000Z:
  ```json
  "end_date": "2024-08-10"
  ```

  If you need data before a specific date in your time zone, you will want to
  provide a date in ISO 8601 format with the time zone offset.

  ### Example for Central Daylight Time (CDT) which is UTC-5:
  ```json
  "end_date": "2025-08-10T00:00:00-05:00"
  ```

### Example Request

```bash
curl -L -X POST 'https://kxqhznhmriurivmvhaev.supabase.co/functions/v1/export' \
-H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZS1kZW1vIiwicm9sZSI6ImFub24iLCJleHAiOjE5ODM4MTI5OTZ9.CRXP1A7WOeoJeXxjNni43kdQwgnWNReilDMblYTn_I0' \
-H 'Content-Type: application/json' \
--data '{
    "api_key": "xxxx",
    "location_id": "354bdd20-cce7-11ee-9f32-678c43cfd720",
    "start_date": "2024-08-07",
    "end_date": "2024-08-10"
}'
```

## Response

### Successful Response

If the request is successful, the API will return a JSON object containing the data associated with the provided parameters. The response structure is as follows:

- **ok**: *(Boolean)*
  Indicates if the request was successful.

- **data**: *(Array of Objects)*
  An array containing the data points, with each batch containing up to 1,000 records if `page` is specified.

#### Example Response

```json
{
  "ok": true,
  "data": [
    {
      "rtst": "2024-08-07T12:52:28.000Z",
      "head1_value": "155",
      "head2_value": "155"
    },
    {
      "rtst": "2024-08-07T12:51:28.000Z",
      "head1_value": "155",
      "head2_value": "155"
    }
    // Additional records up to 1,000
  ]
}
```

### Pagination

The API supports pagination using the `page` parameter. If `page` is specified, the API will return a single batch of data. If `page` is not specified, the API will return the first 1,000 records in date ascending order.

To get the next batch of data, increment the `page` parameter. Continue this process until the API returns `{"ok":true,"data":[]}`.

### Single Value Response

If neither `start_date` or `end_date` is provided, the API returns a single value, which is the last available data point:

```json
{
  "ok": true,
  "data": [
    {
      "rtst": "2024-08-07T12:52:28.000Z",
      "head1_value": "155",
      "head2_value": "155"
    }
  ]
}
```

## Error Handling

If the request fails, the API will return an error response with an appropriate status code and message. Ensure to handle these errors gracefully in your implementation.

### Common Errors

- **401 Unauthorized**: Invalid or missing authentication token.
- **400 Bad Request**: Invalid request parameters.
- **500 Internal Server Error**: An error occurred on the server.
