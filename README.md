# Customer Data Export API Documentation

## Overview

This API allows customers to export data based on specific parameters such as `api_key`, `meter_id`, `start_date`, `end_date`, and `page`. The data returned includes values related to specific timestamps (`rtst`), head values, and expression values. The API is designed to handle paginated requests and can return data in batches of 1,000 records.

## Endpoint

### URL

\`\`\`plaintext
https://api.oleaedge.com/functions/v1/export
\`\`\`

### Method

\`\`\`plaintext
POST
\`\`\`

## Headers

- **Authorization**: Bearer Token  
  This token is required to authenticate the request.
  
  Example:
  \`\`\`plaintext
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZS1kZW1vIiwicm9sZSI6ImFub24iLCJleHAiOjE5ODM4MTI5OTZ9.CRXP1A7WOeoJeXxjNni43kdQwgnWNReilDMblYTn_I0
  \`\`\`

- **Content-Type**: application/json  
  This header specifies that the request body is in JSON format.

## Request Body

The request body must be a JSON object with the following parameters:

- **api_key**: *(String)*  
  The API key used for authentication.
  
  Example:
  \`\`\`json
  "api_key": *(Provided By Olea)*
  \`\`\`

- **meter_id**: *(String)*  
  The unique identifier for the meter.
  
  Example:
  \`\`\`json
  "meter_id": "354bdd20-cce7-11ee-9f32-678c43cfd720"
  \`\`\`

- **start_date**: *(String, Optional)*  
  The start date for the data export in the format `YYYY-MM-DD`. If not provided, the most recent data is returned.
  
  Example:
  \`\`\`json
  "start_date": "2024-08-07"
  \`\`\`

- **end_date**: *(String, Optional)*  
  The end date for the data export in the format `YYYY-MM-DD`. If provided, data between `start_date` and `end_date` is returned.
  
  Example:
  \`\`\`json
  "end_date": "2024-08-10"
  \`\`\`

- **page**: *(Integer, Optional)*  
  Specifies the page of data to be returned in batches of 1,000 records. If not provided, only the last value is returned if `start_date` and `end_date` are also not provided.
  
  Example:
  \`\`\`json
  "page": 2
  \`\`\`

### Expression Value

The `expression_value` is a calculated value based on a project-specific expression configured on the website [http://kingpin.oleaege.com](http://kingpin.oleaege.com). If this setting is configured for the project, the `expression_value` column will be included in the output. 

For example, if the project's expression is set to `(x - y) * 0.5`, the `expression_value` will represent the result of this calculation for each data point.

### Example Request

\`\`\`bash
curl -L -X POST 'https://kxqhznhmriurivmvhaev.supabase.co/functions/v1/export' \
-H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZS1kZW1vIiwicm9sZSI6ImFub24iLCJleHAiOjE5ODM4MTI5OTZ9.CRXP1A7WOeoJeXxjNni43kdQwgnWNReilDMblYTn_I0' \
-H 'Content-Type: application/json' \
--data '{
    "api_key": "xxxx",
    "meter_id": "354bdd20-cce7-11ee-9f32-678c43cfd720",
    "start_date": "2024-08-07",
    "end_date": "2024-08-10",
    "page": 2
}'
\`\`\`

## Response

### Successful Response

If the request is successful, the API will return a JSON object containing the data associated with the provided parameters. The response structure is as follows:

- **ok**: *(Boolean)*  
  Indicates if the request was successful.
  
- **data**: *(Array of Objects)*  
  An array containing the data points, with each batch containing up to 1,000 records if `page` is specified.

#### Example Response

\`\`\`json
{
  "ok": true,
  "data": [
    {
      "rtst": "2024-08-07T12:52:28.000Z",
      "head1_value": "155",
      "head2_value": "155",
      "expression_value": 0
    },
    {
      "rtst": "2024-08-07T12:51:28.000Z",
      "head1_value": "155",
      "head2_value": "155",
      "expression_value": 0
    }
    // Additional records up to 1,000 if `page` is specified
  ]
}
\`\`\`

### Single Value Response

If no `start_date`, `end_date`, or `page` is provided, the API returns a single value, which is the last available data point:

\`\`\`json
{
  "ok": true,
  "data": [
    {
      "rtst": "2024-08-07T12:52:28.000Z",
      "head1_value": "155",
      "head2_value": "155",
      "expression_value": 0
    }
  ]
}
\`\`\`

## Error Handling

If the request fails, the API will return an error response with an appropriate status code and message. Ensure to handle these errors gracefully in your implementation.

### Common Errors

- **401 Unauthorized**: Invalid or missing authentication token.
- **400 Bad Request**: Invalid request parameters.
- **500 Internal Server Error**: An error occurred on the server.
