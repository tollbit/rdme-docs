---
title: Content
excerpt: Retrieve content by URL, self-report usage, list content catalog, and index content with tokens.
---

# Content

Content represents the actual payload and data that you are looking for as a developer. You get content by making a request to a specific webpage. Every webpage has its own [rate](rates).

## Content Intent

Content Intent represents your use case for the content you are requesting. TollBit currently supports two different intents: Content Retrieval and Indexing.

### Indexing

Indexing content (sometimes called crawling) is used when bots retrieve content to build indexes or analyze content at scale. If you are scanning pages to prebuild search indexes or running NLP to understand content, you are using indexing intent.

If you plan to display the content to end users or use it as model training data, use content retrieval intent instead.

### Content Retrieval

Content Retrieval allows bots to fetch content to directly show or summarize for end users (subject to the license). For example, if you are pulling the latest news for a user or researching trips, you are using content retrieval.

---

## Get Content

Request content for a webpage. The content path is the full URL of the intended webpage (e.g. `example.com/find/my/page`). Use a token from **Generate Content Token** (Tokens API) for paid access.

### Endpoint

```
GET /dev/v2/content/{content_path}
```

**Base URL**: `https://gateway.tollbit.com`

**Authentication**: Requires a one-time token via the `TollbitToken` header and your `User-Agent` (must contain your registered AgentID).

### Request headers

- **`TollbitToken`** (string, required)
  - The generated one-time token used to access data (from Generate Content Token).

- **`User-Agent`** (string, required)
  - Unique User-Agent string for who is allowed to access the page. Must contain your AgentID that you previously registered.

- **`Tollbit-Accept-Content`** (string, optional)
  - The MIME content type you want in the `content` response. Accepted: `text/html`, `text/markdown`. Default: `text/markdown`.

### Path parameters

- **`content_path`** (string, required)
  - The full URL of the intended webpage. URL-encode when used in the path.

### Response format

```json
{
  "content": {
    "header": "...",
    "body": "...",
    "footer": "..."
  },
  "metadata": {
    "author": "string",
    "description": "string",
    "imageUrl": "string",
    "modified": "string",
    "published": "string",
    "title": "string"
  },
  "rate": {
    "license": { "id": "string", "licensePath": "string", "licenseType": "string", "permissions": [] },
    "price": { "currency": "string", "priceMicros": 0 }
  }
}
```

### Response fields

- **`content`** (object): Sections of the page.
  - **`header`** (string): Navigation and auxiliary info.
  - **`body`** (string): Main content in markdown, excluding header/nav/footer.
  - **`footer`** (string): Follow-up articles, terms, social links.

- **`metadata`** (object): Article metadata (`author`, `description`, `imageUrl`, `modified`, `published`, `title`).

- **`rate`** (object): Rate data for this page. See [Rates](rates) for details.

### Example request

```bash
curl --location 'https://gateway.tollbit.com/dev/v2/content/https%3A%2F%2Fexample.com%2Farticle' \
  --header 'TollbitToken: eyJhbGciOiJF...' \
  --header 'User-Agent: my-agent/1.0'
```

---

## Index Content

Request content for a webpage for **indexing/crawl** purposes. The path and method are the same as Get Content; the difference is the token: use a token from **Generate Indexing Token** (Tokens API). Content providers may allow or disallow access for indexing.

### Endpoint

```
GET /dev/v2/content/{content_path}
```

**Base URL**: `https://gateway.tollbit.com`

**Authentication**: Same as Get Content: `TollbitToken` (from Generate Indexing Token) and `User-Agent`.

### Request headers

- **`TollbitToken`** (string, required)
  - The generated one-time token from Generate Indexing Token.

- **`User-Agent`** (string, required)
  - Must contain your registered AgentID.

### Path parameters

- **`content_path`** (string, required)
  - The full URL of the intended webpage. URL-encode when used in the path.

### Response format

Same structure as Get Content, but typically without `rate` (indexing use):

```json
{
  "content": {
    "header": "...",
    "body": "...",
    "footer": "..."
  },
  "metadata": {
    "author": "string",
    "description": "string",
    "imageUrl": "string",
    "modified": "string",
    "published": "string",
    "title": "string"
  }
}
```

### Example request

```bash
curl --location 'https://gateway.tollbit.com/dev/v2/content/https%3A%2F%2Fexample.com%2Farticle' \
  --header 'TollbitToken: eyJhbGciOiJF...' \
  --header 'User-Agent: my-agent/1.0'
```

---

## Self Report Usage

Self-report your usage asynchronously. Use your API key and User-Agent (cURL examples use `TollbitKey`).

### Endpoint

```
POST /tollbit/dev/v2/transactions/selfReport
```

**Base URL**: `https://gateway.tollbit.com`

**Authentication**: API key via `TollbitKey` header; User-Agent via `UserAgent` header (per cURL example).

### Request headers

- **`TollbitKey`** (string, required)
  - Your API key.

- **`UserAgent`** (string, required)
  - Your registered AgentID / user agent.

### Request body (JSON)

- **`idempotencyId`** (string, required)
  - A unique ID you generate for idempotency when reporting usage. Useful for deduplication and retries.

- **`usage`** (array of objects, required)
  - List of usage entries. Each entry:
    - **`url`** (string, required): The URL you wish to report usage on.
    - **`timesUsed`** (integer, required): Number of times you used this URL.
    - **`licenseType`** (string, required): One of `ON_DEMAND_LICENSE`, `ON_DEMAND_FULL_USE_LICENSE`, `CUSTOM_LICENSE`. For `CUSTOM_LICENSE` include **`licenseCuid`** (string) with the license ID.

### Response format

The API returns an array of transaction results, one per usage entry:

```json
[
  {
    "url": "https://example.com/article",
    "perUnitPriceMicros": 5000,
    "totalUsePriceMicros": 10000,
    "currency": "USD",
    "license": {
      "cuid": "ji73lwbqjnhfgpu1bk3sjzj1",
      "licenseType": "ON_DEMAND_LICENSE",
      "licensePath": "<license_url>",
      "permissions": [{ "name": "PARTIAL_USE" }]
    }
  }
]
```

### Example request

```bash
curl --location 'https://gateway.tollbit.com/tollbit/dev/v2/transactions/selfReport' \
  --header 'UserAgent: my-agent/1.0' \
  --header 'TollbitKey: YOUR_API_KEY_HERE' \
  --header 'Content-Type: application/json' \
  --data '{
    "idempotencyId": "1234",
    "usage": [
      {
        "url": "https://example.com/article",
        "timesUsed": 2,
        "licenseType": "ON_DEMAND_LICENSE"
      }
    ]
  }'
```

---

## List Content Catalog

Paginate through a flattened sitemap of a particular website.

### Endpoint

```
GET /dev/v2/content/{base_url}/catalog/list
```

**Base URL**: `https://gateway.tollbit.com`

**Authentication**: API key via `TollbitKey` header (per cURL example).

### Request headers

- **`TollbitKey`** (string, required)
  - Your API key.

### Path parameters

- **`base_url`** (string, required)
  - The URL root domain you want to list the content catalog from (e.g. `example.com`). URL-encode when used in the path.

### Optional query parameters

- **`pageToken`** (string, optional)
  - Pagination token to fetch the next page (from previous response).

- **`pageSize`** (integer, optional)
  - Number of items per page (if supported).

- **`from`** (string, optional)
  - Only return pages last modified on or after this date. RFC3339 date or datetime (e.g. `2025-12-01` or `2025-12-01T00:00:00Z`); inclusive, UTC day granularity.

- **`to`** (string, optional)
  - Only return pages last modified on or before this date. Same format as `from`; inclusive, UTC day granularity.

### Response format

```json
{
  "pageToken": "example_token",
  "pages": [
    {
      "lastMod": "2025-12-06T02:17:08Z",
      "pageUrl": "https://example.com/page-1",
      "propertyId": "example_id"
    },
    {
      "lastMod": "2025-12-06T02:11:49Z",
      "pageUrl": "https://example.com/page-2",
      "propertyId": "example_id"
    }
  ]
}
```

### Response fields

- **`pageToken`** (string): Pagination token to fetch the next page. Omit or empty when no more pages.

- **`pages`** (array): Pages in the flattened sitemap, ordered by last modified date (if present) descending.
  - **`lastMod`** (string): Last modified date (ISO 8601).
  - **`pageUrl`** (string): Full page URL.
  - **`propertyId`** (string): Property identifier.

### Example request

```bash
curl --location 'https://gateway.tollbit.com/dev/v2/content/example.com/catalog/list' \
  --header 'TollbitKey: YOUR_API_KEY_HERE'
```

---

## Error responses

- **400 Bad Request**: Invalid request (e.g. invalid token, invalid parameters, access not allowed).
- **401 Unauthorized**: Missing or invalid `TollbitKey` or invalid/missing token.

Error responses follow the standard format with `detail`, `instance`, `status`, `title`, and `type`.

## Notes

- **Get Content vs Index Content**: Same path and method; the token type (content token vs indexing token) determines the intent. Indexing access may be restricted by the content provider.
- **License types**: `ON_DEMAND_LICENSE`, `ON_DEMAND_FULL_USE_LICENSE`, `CUSTOM_LICENSE`. For `CUSTOM_LICENSE` include `licenseCuid` where required (e.g. Self Report Usage).
