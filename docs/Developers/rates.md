---
title: Rates
excerpt: Get rates and license options for content. Single URL or bulk by list of URLs.
---

# Rates

Content providers may set rates when allowing licensed access to their content. The standard licenses are an agreement between the providers and consumers of the content, and only apply to a single use of that content. A page may have multiple options for licenses, and each one may have a different rate for the terms of access.

---

## Get Rate

Get the rate(s) for a single webpage. The content path is the full URL of the intended webpage (URL-encoded when used in the path).

### Endpoint

```
GET /tollbit/dev/v2/rate/{content_path}
```

**Base URL**: `https://gateway.tollbit.com`

**Authentication**: Requires API key via the `TollbitKey` header.

### Request headers

- **`TollbitKey`** (string, required)
  - Your API key, available in the access page of the dashboard.

### Path parameters

- **`content_path`** (string, required)
  - The full URL of the intended webpage. URL-encode the URL when used in the path (e.g. `https%3A%2F%2Fexample.com%2Farticle`).

### Response format

The API returns an array of rate options. Each option includes a license and price:

```json
[
  {
    "license": {
      "id": "nl1sqiauvujk8i6u507wy2mi",
      "licensePath": "<license_url>",
      "licenseType": "ON_DEMAND_LICENSE",
      "permissions": [
        { "name": "PARTIAL_USE" }
      ]
    },
    "price": {
      "currency": "USD",
      "priceMicros": 5000
    }
  },
  {
    "license": {
      "id": "rrmeq2kjsvouhda4ud28uykl",
      "licensePath": "<license_url>",
      "licenseType": "ON_DEMAND_FULL_USE_LICENSE",
      "permissions": [
        { "name": "FULL_USE" },
        { "name": "PARTIAL_USE" }
      ]
    },
    "price": {
      "currency": "USD",
      "priceMicros": 10000
    }
  }
]
```

### Response fields (per rate option)

- **`license`** (object)
  - **`id`** (string): License identifier.
  - **`licensePath`** (string): URL to view the license.
  - **`licenseType`** (string): One of `ON_DEMAND_LICENSE`, `ON_DEMAND_FULL_USE_LICENSE`, `CUSTOM_LICENSE`.
  - **`permissions`** (array): Rights granted. May include `PARTIAL_USE`, `FULL_USE`, `CITATION_REQUIRED`.

- **`price`** (object)
  - **`priceMicros`** (integer): Price in micros ($1 = 1,000,000 micros).
  - **`currency`** (string): Currency code (e.g. `USD`).

### Example request

```bash
# URL-encode the content URL when using it in the path
curl -G "https://gateway.tollbit.com/tollbit/dev/v2/rate/https%3A%2F%2Fexample.com%2Farticle" \
  --header "TollbitKey: YOUR_API_KEY_HERE"
```

---

## Bulk Get Rate

Get the rates for a list of URLs in a single request.

### Endpoint

```
POST /tollbit/dev/v2/rate/batch
```

**Base URL**: `https://gateway.tollbit.com`

**Authentication**: Requires API key via the `TollbitKey` header.

### Request headers

- **`TollbitKey`** (string, required)
  - Your API key, available in the access page of the dashboard.

### Request body (JSON)

- **`urls`** (array of strings, required)
  - List of URLs that you want to fetch rates for.

### Response format

The API returns an array of objects, one per URL. Each object has a `url` and a `rates` array:

```json
[
  {
    "url": "https://example.com/article-1",
    "rates": [
      {
        "price": {
          "priceMicros": 5000,
          "currency": "USD"
        },
        "license": {
          "cuid": "ji73lwbqjnhfgpu1bk3sjzj1",
          "licenseType": "ON_DEMAND_LICENSE",
          "licensePath": "<license_url>",
          "permissions": [{ "name": "PARTIAL_USE" }]
        }
      },
      {
        "price": {
          "priceMicros": 10000,
          "currency": "USD"
        },
        "license": {
          "cuid": "ec1lhwillyl6i3x4gnb4a6gt",
          "licenseType": "ON_DEMAND_FULL_USE_LICENSE",
          "licensePath": "<license_url>",
          "permissions": [
            { "name": "FULL_USE" },
            { "name": "PARTIAL_USE" }
          ]
        }
      }
    ]
  },
  {
    "url": "https://example.com/article-2",
    "rates": [...]
  }
]
```

### Response fields (per URL entry)

- **`url`** (string): The URL that this set of licenses and rates belongs to.
- **`rates`** (array): List of rate options (each with `price` and `license` as above).

### Example request

```bash
curl --location 'https://gateway.tollbit.com/tollbit/dev/v2/rate/batch' \
  --header 'TollbitKey: YOUR_API_KEY_HERE' \
  --header 'Content-Type: application/json' \
  --data '{
    "urls": [
      "https://example.com/article-1",
      "https://example.com/article-2"
    ]
  }'
```

---

## Error responses

- **400 Bad Request**: Invalid request (e.g. access not allowed for a URL, invalid parameters).
- **401 Unauthorized**: Missing or invalid `TollbitKey` header.

Error responses follow the standard format with `detail`, `instance`, `status`, `title`, and `type`.

## Notes

- **Price in micros**: $1 = 1,000,000 micros. Divide `priceMicros` by 1,000,000 for the amount in the currency unit.
- **License types**: `ON_DEMAND_LICENSE`, `ON_DEMAND_FULL_USE_LICENSE`, `CUSTOM_LICENSE`. For `CUSTOM_LICENSE` you may need a `licenseCuid` when requesting tokens or content.
