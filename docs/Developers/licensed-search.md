---
title: Licensed Search
excerpt: >-
  Search for content across the web and discover licensable content through
  TollBit.
---
# Licensed Search Documentation

## Overview

The Licensed Search API allows you to search for content across the web. Results include information about the content's availability for licensing through TollBit, helping you discover and identify licensable content.

## Endpoint

```
GET /dev/v2/search
```

**Authentication**: Requires API key authentication via `TollBitKey` header

## Request Parameters

### Required Parameters

* **`q`** (string, required)
  * The search query string
  * Example: `"artificial intelligence"`

### Optional Parameters

* **`size`** (integer, optional)
  * Number of results to return per page
  * **Default**: `20`
  * **Range**: `1` to `20` (inclusive)

* **`next-token`** (string, optional)
  * Token to get the next page of results

* **`readyToLicense`** (bool, optional)
  * When `true`, limits search to only the properties that are ready to license — those that have set rates and can be licensed programmatically
  * Every result returned will have `availability.readyToLicense` set to `true`

* **`properties`** (string, optional)
  * Comma-separated list of domains to search, only results from these domains will be included in the results
  * **Maximum**: 20 domains
  * **Format**: Domain only (e.g., `example.com` or `www.example.com`)
    * Must not include protocol (`http://`, `https://`)
    * Must not include ports (`:8080`)
    * Must not include paths (`/path`)
  * **Example**: `"example.com,www.anothersite.com"`

## Response Format

The API returns a `PagedSearchResultResponse` object:

```json
{
  "nextToken": "string",
  "items": [
    {
      "title": "string",
      "url": "string",
      "publishedDate": "string",
      "publisher": {
        "domain": "string",
        "name": "string"
      },
      "availability": {
        "discoverable": boolean,
        "readyToLicense": boolean
      }
    }
  ]
}
```

### Response Fields

#### Top-Level Fields

* **`nextToken`** (string)
  * Token to use for retrieving the next page of results
  * Empty string (`""`) if there are no more results

* **`items`** (array)
  * Array of search results
  * May be empty if no results are found

#### SearchResult Object

Each item in the `items` array is a `SearchResult` object with the following fields:

* **`title`** (string)
  * The title of the content/page

* **`url`** (string)
  * The full URL of the content

* **`publishedDate`** (string)
  * The publication date of the content (format may vary based on source)

* **`publisher`** (object)
  * Information about the publisher of the content
  * **`domain`** (string): The domain name of the publisher (e.g., `"example.com"`)
  * **`name`** (string): The name of the publisher

* **`availability`** (object)
  * Information about the content's availability for licensing
  * **`discoverable`** (boolean): `true` if the publisher is part of TollBit's network
  * **`readyToLicense`** (boolean): `true` if the publisher has set rates and is ready to transact now

## Usage Examples

### Basic Search

```bash
curl --location 'https://gateway.tollbit.com/dev/v2/search?q=Tollbit' \
  --header 'TollBitKey: <KEY>'
```

**Response:**

```json
{
  "nextToken": "MToyMA==",
  "items": [
    {
      "title": "Understanding Artificial Intelligence",
      "url": "https://example.com/ai-guide",
      "publishedDate": "2024-01-15",
      "publisher": {
        "domain": "example.com",
        "name": "example.com"
      },
      "availability": {
        "discoverable": true,
        "readyToLicense": true
      }
    }
  ]
}
```

### Search with Custom Page Size

```bash
curl --location 'https://gateway.tollbit.com/dev/v2/search?q=machine%20learning&size=10' \
  --header 'TollBitKey: <KEY>'
```

### Search with Property Filtering

```bash
curl --location 'https://gateway.tollbit.com/dev/v2/search?q=technology&properties=example.com,www.techsite.com' \
  --header 'TollBitKey: <KEY>'
```

This will filter the search results to only show results from `example.com` and `www.techsite.com`. Only verified properties in the TollBit system will be used.

### Search Only Properties Ready to License

```bash
curl --location 'https://gateway.tollbit.com/dev/v2/search?q=technology&readyToLicense=true' \
  --header 'TollBitKey: <KEY>'
```

This will filter the search results to only properties that have set rates and can be licensed programmatically. Every result returned will have `availability.readyToLicense` set to `true`. When combined with `properties`, the two filters intersect.

### Pagination

**First Request:**

```bash
curl --location 'https://gateway.tollbit.com/dev/v2/search?q=technology&size=20' \
  --header 'TollBitKey: <KEY>'
```

**Response:**

```json
{
  "nextToken": "MToyMA==",
  "items": [...]
}
```

**Next Page Request:**

```bash
curl --location 'https://gateway.tollbit.com/dev/v2/search?q=technology&next-token=MToyMA==' \
  --header 'TollBitKey: <KEY>'
```

## Error Responses

The Licensed Search API may return the following error responses:

* **400 Bad Request**: Invalid parameters (e.g., missing `q`, invalid `size` range, invalid `properties` format)
* **401 Unauthorized**: Missing or invalid `TollBitKey` header
* **500 Internal Server Error**: Server-side error

Error responses follow the standard `ErrorResponse` format.

## Notes

* **Maximum Pages**: The API supports up to 10 pages (pages 0-9). After page 9, `nextToken` will be empty.
* **Property Filtering**: Only properties in the TollBit network are used when filtering by `properties`. Invalid or unverified domains are silently ignored.
* **License Checking**: The `readyToLicense` field indicates whether a publisher has set rates and is ready to transact.
* **Domain Matching**: The system handles both `www` and non-`www` variants of domains automatically.
