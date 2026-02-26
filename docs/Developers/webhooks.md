---
title: Webhooks
excerpt: >-
  Receive real-time notifications when content becomes available from TollBit
  publishers, and retrieve webhook delivery history.
next:
  pages:
    - slug: devv1webhookhistory
      title: /dev/v1/webhook/history
      type: endpoint
---

# Webhooks

This section covers TollBit's webhooks model, including how to validate a webhook from TollBit and how to retrieve the history of webhook deliveries.

## The Webhooks model

Webhooks allow your application to receive real-time notifications when content becomes available from TollBit publishers. Instead of polling our API repeatedly, you can register a webhook endpoint and we'll push events to you as they happen.

### Webhook payload attributes

Each webhook delivery includes the following fields:

- **`sourceCuid`** (string)
  - Account identifier for the property.

- **`sourceHost`** (string)
  - Domain host of the content.

- **`contentUrl`** (string)
  - The specific page URL for the intended content to access.

- **`timestamp`** (string)
  - UTC timestamp of when content was appended (e.g. ISO 8601 format).

- **`notificationType`** (string)
  - Either `CREATED` or `UPDATED`.

### Example webhook payload

```json
{
  "sourceCuid": "rd4mezz10lifxlk90d9nd38p",
  "sourceHost": "pioneervalleygazette.com",
  "contentUrl": "https://pioneervalleygazette.com/article/bubble-fairy",
  "timestamp": "2025-09-30T23:08:40.620435262Z",
  "notificationType": "CREATED"
}
```

## Validating a webhook

Use the following to validate that a webhook is from TollBit:

- **IP range**: Webhook deliveries use a NAT Gateway with a single public IP: **52.22.183.94**. All webhooks from TollBit originate from this IP.

- **Signature**: We use HMAC-SHA256 for the signature. The value is in the `Authorization` header. The signature is computed over only the webhook URL you supplied; the key is the **Signing Secret** associated with the Subscription (available in the UI).

- **User-Agent**: Requests use the User-Agent value **`tollbot/1.0`**. You can verify this to confirm the request is from TollBit.

## Get webhook history

Retrieve the history of webhook deliveries within an optional lookback window, including content URLs and last attempted delivery time and status.

### Endpoint

```
GET /dev/v1/webhook/history
```

**Base URL**: `https://gateway.tollbit.com`

**Authentication**: Requires API key via the `TollbitKey` header.

### Request parameters (query)

All parameters are optional.

- **`next-token`** (string, optional)
  - Pagination token. The response may include a token to retrieve the next page.

- **`size`** (integer, optional)
  - Number of items per page.
  - **Default**: `25`
  - **Maximum**: `1000`

- **`days-ago`** (integer, optional)
  - Number of days back from the current date to retrieve webhook history.

### Response format

The API returns an object with an array of delivery records and an optional continuation token:

```json
{
  "Items": [],
  "nextToken": "string"
}
```

- **`Items`** (array): List of webhook delivery records (see below). May be empty.
- **`nextToken`** (string): If present, use this to request the next page of results.

### Item (webhook delivery record)

Each element in `Items` has the following shape:

- **`subscriptionId`** (string): ID of the subscription linking the recipient org to the publisher.
- **`contentUrl`** (string): URL of the content that was updated (e.g. a Time.com URL).
- **`deliveryUrl`** (string): URL that TollBit attempted to send the webhook to.
- **`eventType`** (string): Either `CREATE` or `UPDATE`.
- **`lastAttempt`** (string): Timestamp in UTC of the last attempt to send the webhook to `deliveryUrl`.
- **`lastStatus`** (integer): HTTP status code of the last attempt.
- **`numAttempts`** (integer): Number of delivery attempts.

### Example response (item)

```json
{
  "subscriptionId": "sub_abc123",
  "contentUrl": "https://example.com/article/123",
  "deliveryUrl": "https://your-app.com/webhooks/tollbit",
  "eventType": "CREATE",
  "lastAttempt": "2025-09-30T23:08:40.620435262Z",
  "lastStatus": 200,
  "numAttempts": 1
}
```

## Usage examples

### Get recent webhook history

```bash
curl --location 'https://gateway.tollbit.com/dev/v1/webhook/history?size=10&days-ago=30' \
  --header 'TollbitKey: YOUR_API_KEY_HERE'
```

### Get history with pagination

**First request:**

```bash
curl --location 'https://gateway.tollbit.com/dev/v1/webhook/history?size=25&days-ago=7' \
  --header 'TollbitKey: YOUR_API_KEY_HERE'
```

**Next page (if `nextToken` is returned):**

```bash
curl --location 'https://gateway.tollbit.com/dev/v1/webhook/history?size=25&days-ago=7&next-token=YOUR_NEXT_TOKEN' \
  --header 'TollbitKey: YOUR_API_KEY_HERE'
```

## Error responses

- **401 Unauthorized**: Missing or invalid `TollbitKey` header.
- **500 Internal Server Error**: Server-side error.

Error responses follow the standard API error format (e.g. `detail`, `status`, `title`, `type`).