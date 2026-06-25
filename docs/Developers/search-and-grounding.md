---
title: Search and Grounding
excerpt: >-
  Ground your agent in licensed content — either with TollBit's licensed search,
  or by pairing a third-party search API (Perplexity, Firecrawl, Exa) with
  TollBit licensing.
---
# Search and Grounding

When you ground an agent's response in real web content, you need two things: a way to **discover** relevant pages, and a way to **license** the ones you use. TollBit supports two approaches:

1. **TollBit Licensed Search** — discovery and licensing in a single network. You search TollBit, and every one of the results is ready to be licensed.
2. **Bring your own search** — use a third-party search API (Perplexity, Firecrawl, or Exa) for discovery, then use TollBit to check which results are licensable and to license the ones you need.

Use **Approach 1** when you want the simplest path and are happy to discover content within the TollBit network. Use **Approach 2** when you already have a preferred search provider, or want broader web coverage, and only need TollBit for the licensing layer.

## Prerequisites

Before you begin, make sure you have:

<Tabs>
  <Tab title="cURL">
    1. **cURL installed**: cURL is available on most systems. Check with:
       ```bash
       curl --version
       ```

    2. **A TollBit organization API key**: Required for both approaches. Replace {'`<key>`'} in the examples below with your actual key.

    3. **A user agent**: Identifies your application. Add it to the dev dashboard under the `My Agents` tab.

    4. **A third-party search API key** (Approach 2 only): A Perplexity, Firecrawl, or Exa API key, depending on which provider you use.
  </Tab>

  <Tab title="Python">
    Install the **TollBit SDK** (used for licensing in both approaches), plus **`requests`** for the Approach 2 third-party search calls:

    ```bash
    pip install tollbit-python-sdk requests
    ```

    Then set your credentials as environment variables. Add your user agent to the dev dashboard under the `My Agents` tab first, then:

    ```bash
    export TOLLBIT_ORG_API_KEY="your-tollbit-key-here"  # required for both approaches
    export TOLLBIT_USER_AGENT="your-app-name/1.0.0"

    # Approach 2 only — set whichever provider you use
    export PERPLEXITY_API_KEY="your-perplexity-key-here"
    export FIRECRAWL_API_KEY="your-firecrawl-key-here"
    export EXA_API_KEY="your-exa-key-here"
    ```
  </Tab>
</Tabs>

***

## Approach 1: TollBit Licensed Search

With licensed search, discovery happens inside TollBit. The workflow is three steps:

1. **Search** for content with the licensed search endpoint
2. **Get rates** for a result to check pricing and license options
3. **License the content** — acquire a token and retrieve the content

Each search result includes an `availability` block that tells you whether the content is `discoverable` (in the TollBit network) and `readyToLicense` (rates are set and it can be transacted now), so you know up front what is licensable.

> 📘 Full walkthrough
>
> This section is a condensed end-to-end. For the complete reference — every query parameter, response field, pagination, error handling, and a full bash + Python script — see the **Search to Content Workflow** guide.

### Step 1: Search

<Tabs>
  <Tab title="cURL">
    ```bash
    curl --location 'https://gateway.tollbit.com/dev/v2/search?q=DIY%20projects%20for%20Millenials' \
    --header 'TollbitKey: <key>'
    ```
  </Tab>

  <Tab title="Python">
    ```python
    from tollbit import search
    import os

    api_key = os.getenv("TOLLBIT_ORG_API_KEY", "YOUR_API_KEY_HERE")
    user_agent = os.getenv("TOLLBIT_USER_AGENT", "tollbit-python-sdk-example/0.1.0")

    search_client = search.create_client(secret_key=api_key, user_agent=user_agent)
    results = search_client.search(q="DIY home projects for millennials")

    for item in results.items:
        print(f"{item.title} — {item.url}")
        print(f"  discoverable={item.availability.discoverable} "
              f"ready_to_license={item.availability.ready_to_license}")
    ```
  </Tab>
</Tabs>

Useful query parameters: `size` (max 20), `next-token` (pagination), `allowedOnly` (limit to properties your account is authorized to license), and `properties` (comma-separated domains, max 20).

### Step 2: Get Rates

<Tabs>
  <Tab title="cURL">
    ```bash
    curl --location 'https://gateway.tollbit.com/dev/v2/rates/https://www.example.com/article' \
    --header 'TollbitKey: <key>'
    ```
  </Tab>

  <Tab title="Python">
    ```python
    from tollbit import use_content
    import os

    api_key = os.getenv("TOLLBIT_ORG_API_KEY", "YOUR_API_KEY_HERE")
    user_agent = os.getenv("TOLLBIT_USER_AGENT", "tollbit-python-sdk-example/0.1.0")

    content_client = use_content.create_client(secret_key=api_key, user_agent=user_agent)
    rate_info = content_client.get_rate(url=results.items[0].url)

    for rate in rate_info:
        print(f"{rate.price.price_micros / 1_000_000:.2f} {rate.price.currency} "
              f"— {rate.license.license_type}")
    ```
  </Tab>
</Tabs>

### Step 3: License the Content

Once you've picked a licensable URL, licensing is two parts: get an access token, then use it to retrieve the content. With the Python SDK, `get_sanctioned_content` handles both in one call. With cURL, you obtain the token first, then request the content.

#### Step 3a: Get a Token (cURL only)

<Tabs>
  <Tab title="cURL">
    ```bash
    curl --location 'https://gateway.tollbit.com/dev/v2/tokens/content' \
    --header 'Content-Type: application/json' \
    --header 'TollbitKey: <key>' \
    --data '{
        "url": "https://www.example.com/article",
        "userAgent": "test-agent",
        "maxPriceMicros": 11000000,
        "currency": "USD",
        "licenseType": "ON_DEMAND_LICENSE"
    }'
    ```

    The response contains a `token` you'll use in Step 3b. For `CUSTOM_LICENSE`, also include `licenseCuid` in the request body.
  </Tab>

  <Tab title="Python">
    The Python SDK acquires the token automatically inside `get_sanctioned_content` (Step 3b) — no separate token call needed.
  </Tab>
</Tabs>

#### Step 3b: Get the Content

<Tabs>
  <Tab title="cURL">
    ```bash
    curl --location 'https://gateway.tollbit.com/dev/v2/content/https://www.example.com/article' \
    --header 'TollbitToken: <Token>' \
    --header 'User-Agent: test-agent'
    ```

    Request a format with the `Accept` header: `text/markdown` (default) or `text/html`.
  </Tab>

  <Tab title="Python">
    ```python
    from tollbit import use_content, licenses, currencies
    import os

    api_key = os.getenv("TOLLBIT_ORG_API_KEY", "YOUR_API_KEY_HERE")
    user_agent = os.getenv("TOLLBIT_USER_AGENT", "tollbit-python-sdk-example/0.1.0")

    content_client = use_content.create_client(secret_key=api_key, user_agent=user_agent)

    # Automatically acquires a token and retrieves the content
    data = content_client.get_sanctioned_content(
        url=results.items[0].url,
        max_price_micros=11000000,  # 11.00 USD
        currency=currencies.USD,
        license_type=licenses.types.ON_DEMAND_LICENSE,
    )

    print(data.metadata.title)
    print(data.content.body)
    ```
  </Tab>
</Tabs>

The content response contains a `content` block (`header`, `body`, `footer`), a `metadata` block (`title`, `description`, `author`, `published`, …), and the `rate` that was applied.

**Available license types**: `ON_DEMAND_LICENSE`, `ON_DEMAND_FULL_USE_LICENSE`, `CUSTOM_LICENSE` (requires `licenseCuid` for cURL / `license_id` for the SDK).

***

## Approach 2: Third-Party Search + TollBit Licensing

If you already use a search provider, you can keep it for discovery and use TollBit only for licensing. The workflow is three steps:

1. **Search** with your preferred search API (Perplexity, Firecrawl, or Exa) to get a list of result URLs
2. **Check licenseability** by passing those URLs to TollBit's batch rate endpoint to see which are available to license
3. **License the content** for the URLs you want — acquire a token and retrieve the content

### Step 1: Search with a Third-Party API

Run a search with your provider of choice and collect the result URLs. You'll pass these URLs to TollBit in Step 2.

<Tabs>
  <Tab title="Perplexity">
    **Endpoint**: `POST https://api.perplexity.ai/search` · **Auth**: `Authorization: Bearer <PERPLEXITY_API_KEY>`

    ```bash
    curl --location 'https://api.perplexity.ai/search' \
    --header 'Authorization: Bearer <PERPLEXITY_API_KEY>' \
    --header 'Content-Type: application/json' \
    --data '{
        "query": "DIY home projects for millennials",
        "max_results": 10
    }'
    ```

    ```python
    import os, requests

    resp = requests.post(
        "https://api.perplexity.ai/search",
        headers={"Authorization": f"Bearer {os.environ['PERPLEXITY_API_KEY']}"},
        json={"query": "DIY home projects for millennials", "max_results": 10},
    )
    results = resp.json()["results"]
    urls = [r["url"] for r in results]
    print(urls)
    ```

    Each item in `results` includes `title`, `url`, `snippet`, `date`, and `last_updated`.
  </Tab>

  <Tab title="Firecrawl">
    **Endpoint**: `POST https://api.firecrawl.dev/v2/search` · **Auth**: `Authorization: Bearer <FIRECRAWL_API_KEY>`

    ```bash
    curl --location 'https://api.firecrawl.dev/v2/search' \
    --header 'Authorization: Bearer <FIRECRAWL_API_KEY>' \
    --header 'Content-Type: application/json' \
    --data '{
        "query": "DIY home projects for millennials",
        "limit": 10
    }'
    ```

    ```python
    import os, requests

    resp = requests.post(
        "https://api.firecrawl.dev/v2/search",
        headers={"Authorization": f"Bearer {os.environ['FIRECRAWL_API_KEY']}"},
        json={"query": "DIY home projects for millennials", "limit": 10},
    )
    results = resp.json()["data"]["web"]
    urls = [r["url"] for r in results]
    print(urls)
    ```

    Each item in `data.web` includes `title`, `url`, `description`, and `metadata`.
  </Tab>

  <Tab title="Exa">
    **Endpoint**: `POST https://api.exa.ai/search` · **Auth**: `x-api-key: <EXA_API_KEY>`

    ```bash
    curl --location 'https://api.exa.ai/search' \
    --header 'x-api-key: <EXA_API_KEY>' \
    --header 'Content-Type: application/json' \
    --data '{
        "query": "DIY home projects for millennials",
        "numResults": 10
    }'
    ```

    ```python
    import os, requests

    resp = requests.post(
        "https://api.exa.ai/search",
        headers={"x-api-key": os.environ["EXA_API_KEY"]},
        json={"query": "DIY home projects for millennials", "numResults": 10},
    )
    results = resp.json()["results"]
    urls = [r["url"] for r in results]
    print(urls)
    ```

    Each item in `results` includes `title`, `url`, `publishedDate`, `author`, and `id`.
  </Tab>
</Tabs>

At the end of this step you have a list of URLs from across the web — some licensable through TollBit, some not. The next step tells you which is which.

### Step 2: Check Licenseability (Batch Rate Check)

Pass the URLs from Step 1 to TollBit's batch rate endpoint. For each URL, TollBit returns the available licenses and rates. A URL with one or more `rates` is licensable through TollBit; a URL that returns an `error` or no rates is not available to license. Use this to filter your search results down to the content you can transact on.

This is the same `rates/batch` endpoint covered in the **Rates** reference — here it's used as a licenseability filter over third-party search results.

**Endpoint**

```
POST /dev/v2/rates/batch
```

**Base URL**: `https://gateway.tollbit.com` · **Authentication**: `TollbitKey` header

**Request body**

- **`urls`** (array of strings, required) — the URLs to check, e.g. the result URLs from Step 1.

<Tabs>
  <Tab title="cURL">
    ```bash
    curl --location 'https://gateway.tollbit.com/dev/v2/rates/batch' \
    --header 'TollbitKey: <key>' \
    --header 'Content-Type: application/json' \
    --data '{
        "urls": [
            "https://www.example.com/article-1",
            "https://www.example.com/article-2"
        ]
    }'
    ```
  </Tab>

  <Tab title="Python">
    ```python
    import os, requests

    # urls comes from your Step 1 search results
    resp = requests.post(
        "https://gateway.tollbit.com/dev/v2/rates/batch",
        headers={"TollbitKey": os.environ["TOLLBIT_ORG_API_KEY"]},
        json={"urls": urls},
    )
    batch = resp.json()

    # Keep only the URLs that are licensable through TollBit
    licensable = [
        entry["url"]
        for entry in batch
        if entry.get("rates") and not any(r.get("error") for r in entry["rates"])
    ]
    print(f"{len(licensable)} of {len(urls)} results are licensable")
    ```
  </Tab>
</Tabs>

**Response**

The endpoint returns an array with one entry per URL. Each entry has the URL and its `rates` array; each rate carries a `price`, a `license`, and an `error` string (empty when the rate is valid).

```json
[
  {
    "url": "https://www.example.com/article-1",
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
          "permissions": [{ "name": "PARTIAL_USE" }],
          "validUntil": "2026-01-01T00:00:00Z"
        },
        "error": ""
      }
    ]
  },
  {
    "url": "https://www.example.com/article-2",
    "rates": []
  }
]
```

**Response fields (per URL entry)**

- **`url`** (string) — the URL this entry refers to.
- **`rates`** (array) — the available license options. Empty when the URL is not licensable through TollBit.
  - **`price`** — `priceMicros` (1/1,000,000 of the currency unit; divide by 1,000,000 for the amount) and `currency`.
  - **`license`** — `cuid`, `licenseType`, `licensePath`, `permissions`, and `validUntil`.
  - **`error`** (string) — populated when a rate could not be returned for the URL; empty otherwise.

> 👍 Tip
>
> The batch endpoint accepts URLs from any source, so you can check a full page of search results in a single request rather than calling the single-URL rate endpoint once per result.

### Step 3: License the Content

Once you've picked a licensable URL (from either approach), licensing is two parts: get an access token, then use it to retrieve the content. With the Python SDK, `get_sanctioned_content` handles both in one call. With cURL, you obtain the token first, then request the content.

#### Step 3a: Get a Token (cURL only)

<Tabs>
  <Tab title="cURL">
    ```bash
    curl --location 'https://gateway.tollbit.com/dev/v2/tokens/content' \
    --header 'Content-Type: application/json' \
    --header 'TollbitKey: <key>' \
    --data '{
        "url": "https://www.example.com/article-1",
        "userAgent": "test-agent",
        "maxPriceMicros": 11000000,
        "currency": "USD",
        "licenseType": "ON_DEMAND_LICENSE"
    }'
    ```

    The response contains a `token` you'll use in Step 3b. For `CUSTOM_LICENSE`, also include `licenseCuid` in the request body.
  </Tab>

  <Tab title="Python">
    The Python SDK acquires the token automatically inside `get_sanctioned_content` (Step 3b) — no separate token call needed.
  </Tab>
</Tabs>

#### Step 3b: Get the Content

<Tabs>
  <Tab title="cURL">
    ```bash
    curl --location 'https://gateway.tollbit.com/dev/v2/content/https://www.example.com/article-1' \
    --header 'TollbitToken: <Token>' \
    --header 'User-Agent: test-agent'
    ```

    Request a format with the `Accept` header: `text/markdown` (default) or `text/html`.
  </Tab>

  <Tab title="Python">
    ```python
    from tollbit import use_content, licenses, currencies
    import os

    api_key = os.getenv("TOLLBIT_ORG_API_KEY", "YOUR_API_KEY_HERE")
    user_agent = os.getenv("TOLLBIT_USER_AGENT", "tollbit-python-sdk-example/0.1.0")

    content_client = use_content.create_client(secret_key=api_key, user_agent=user_agent)

    # Automatically acquires a token and retrieves the content
    data = content_client.get_sanctioned_content(
        url="https://www.example.com/article-1",
        max_price_micros=11000000,  # 11.00 USD
        currency=currencies.USD,
        license_type=licenses.types.ON_DEMAND_LICENSE,
    )

    print(data.metadata.title)
    print(data.content.body)
    ```
  </Tab>
</Tabs>

The content response contains a `content` block (`header`, `body`, `footer`), a `metadata` block (`title`, `description`, `author`, `published`, …), and the `rate` that was applied.

**Available license types**: `ON_DEMAND_LICENSE`, `ON_DEMAND_FULL_USE_LICENSE`, `CUSTOM_LICENSE` (requires `licenseCuid` for cURL / `license_id` for the SDK).

***

## End-to-End Example

Tie the three steps together. The **TollBit** tab shows Approach 1 — licensed search end to end. The **Perplexity**, **Firecrawl**, and **Exa** tabs show Approach 2 — third-party search filtered through TollBit licensing; only Step 1 changes per provider, while Steps 2 and 3 are identical across all three.

<Tabs>
  <Tab title="TollBit">
    ```python
    import os
    from tollbit import search, use_content, licenses, currencies

    TOLLBIT_KEY = os.environ["TOLLBIT_ORG_API_KEY"]
    USER_AGENT = os.getenv("TOLLBIT_USER_AGENT", "tollbit-python-sdk-example/0.1.0")

    # Step 1: Search TollBit's licensed search
    search_client = search.create_client(secret_key=TOLLBIT_KEY, user_agent=USER_AGENT)
    results = search_client.search(q="DIY home projects for millennials")

    # Step 2: Keep only results that are ready to license now
    licensable = [item for item in results.items if item.availability.ready_to_license]
    if not licensable:
        print("No results are ready to license.")
        raise SystemExit(0)

    # Step 3: License the first available result
    target = licensable[0]
    content_client = use_content.create_client(secret_key=TOLLBIT_KEY, user_agent=USER_AGENT)

    # Check the rate, then license up to that price
    rate = content_client.get_rate(url=target.url)[0]
    data = content_client.get_sanctioned_content(
        url=target.url,
        max_price_micros=rate.price.price_micros,
        currency=currencies.USD,
        license_type=licenses.types.ON_DEMAND_LICENSE,
    )

    print(f"Licensed: {data.metadata.title}")
    print(data.content.body[:200] + "...")
    ```
  </Tab>

  <Tab title="Perplexity">
    ```python
    import os, requests
    from tollbit import use_content, licenses, currencies

    TOLLBIT_KEY = os.environ["TOLLBIT_ORG_API_KEY"]
    USER_AGENT = os.getenv("TOLLBIT_USER_AGENT", "tollbit-python-sdk-example/0.1.0")

    # Step 1: Search with Perplexity
    search_resp = requests.post(
        "https://api.perplexity.ai/search",
        headers={"Authorization": f"Bearer {os.environ['PERPLEXITY_API_KEY']}"},
        json={"query": "DIY home projects for millennials", "max_results": 10},
    )
    urls = [r["url"] for r in search_resp.json()["results"]]

    # Step 2: Check licenseability via TollBit's batch rate endpoint
    batch_resp = requests.post(
        "https://gateway.tollbit.com/dev/v2/rates/batch",
        headers={"TollbitKey": TOLLBIT_KEY},
        json={"urls": urls},
    )
    licensable = [
        entry for entry in batch_resp.json()
        if entry.get("rates") and not any(r.get("error") for r in entry["rates"])
    ]
    if not licensable:
        print("None of the search results are licensable through TollBit.")
        raise SystemExit(0)

    # Step 3: License the first available result
    target = licensable[0]
    max_price = target["rates"][0]["price"]["priceMicros"]

    content_client = use_content.create_client(secret_key=TOLLBIT_KEY, user_agent=USER_AGENT)
    data = content_client.get_sanctioned_content(
        url=target["url"],
        max_price_micros=max_price,
        currency=currencies.USD,
        license_type=licenses.types.ON_DEMAND_LICENSE,
    )

    print(f"Licensed: {data.metadata.title}")
    print(data.content.body[:200] + "...")
    ```
  </Tab>

  <Tab title="Firecrawl">
    ```python
    import os, requests
    from tollbit import use_content, licenses, currencies

    TOLLBIT_KEY = os.environ["TOLLBIT_ORG_API_KEY"]
    USER_AGENT = os.getenv("TOLLBIT_USER_AGENT", "tollbit-python-sdk-example/0.1.0")

    # Step 1: Search with Firecrawl
    search_resp = requests.post(
        "https://api.firecrawl.dev/v2/search",
        headers={"Authorization": f"Bearer {os.environ['FIRECRAWL_API_KEY']}"},
        json={"query": "DIY home projects for millennials", "limit": 10},
    )
    urls = [r["url"] for r in search_resp.json()["data"]["web"]]

    # Step 2: Check licenseability via TollBit's batch rate endpoint
    batch_resp = requests.post(
        "https://gateway.tollbit.com/dev/v2/rates/batch",
        headers={"TollbitKey": TOLLBIT_KEY},
        json={"urls": urls},
    )
    licensable = [
        entry for entry in batch_resp.json()
        if entry.get("rates") and not any(r.get("error") for r in entry["rates"])
    ]
    if not licensable:
        print("None of the search results are licensable through TollBit.")
        raise SystemExit(0)

    # Step 3: License the first available result
    target = licensable[0]
    max_price = target["rates"][0]["price"]["priceMicros"]

    content_client = use_content.create_client(secret_key=TOLLBIT_KEY, user_agent=USER_AGENT)
    data = content_client.get_sanctioned_content(
        url=target["url"],
        max_price_micros=max_price,
        currency=currencies.USD,
        license_type=licenses.types.ON_DEMAND_LICENSE,
    )

    print(f"Licensed: {data.metadata.title}")
    print(data.content.body[:200] + "...")
    ```
  </Tab>

  <Tab title="Exa">
    ```python
    import os, requests
    from tollbit import use_content, licenses, currencies

    TOLLBIT_KEY = os.environ["TOLLBIT_ORG_API_KEY"]
    USER_AGENT = os.getenv("TOLLBIT_USER_AGENT", "tollbit-python-sdk-example/0.1.0")

    # Step 1: Search with Exa
    search_resp = requests.post(
        "https://api.exa.ai/search",
        headers={"x-api-key": os.environ["EXA_API_KEY"]},
        json={"query": "DIY home projects for millennials", "numResults": 10},
    )
    urls = [r["url"] for r in search_resp.json()["results"]]

    # Step 2: Check licenseability via TollBit's batch rate endpoint
    batch_resp = requests.post(
        "https://gateway.tollbit.com/dev/v2/rates/batch",
        headers={"TollbitKey": TOLLBIT_KEY},
        json={"urls": urls},
    )
    licensable = [
        entry for entry in batch_resp.json()
        if entry.get("rates") and not any(r.get("error") for r in entry["rates"])
    ]
    if not licensable:
        print("None of the search results are licensable through TollBit.")
        raise SystemExit(0)

    # Step 3: License the first available result
    target = licensable[0]
    max_price = target["rates"][0]["price"]["priceMicros"]

    content_client = use_content.create_client(secret_key=TOLLBIT_KEY, user_agent=USER_AGENT)
    data = content_client.get_sanctioned_content(
        url=target["url"],
        max_price_micros=max_price,
        currency=currencies.USD,
        license_type=licenses.types.ON_DEMAND_LICENSE,
    )

    print(f"Licensed: {data.metadata.title}")
    print(data.content.body[:200] + "...")
    ```
  </Tab>
</Tabs>

***

## Error Handling

Both approaches use standard HTTP status codes on the TollBit endpoints:

- **200 OK**: Request successful
- **400 Bad Request**: Invalid request (e.g. access not allowed for a URL, invalid parameters)
- **401 Unauthorized**: Invalid or missing API key
- **500–599 Server Error**: Server-side errors

For Approach 2, a URL that exists in TollBit but isn't enabled for your organization surfaces as an `error` on its batch rate entry (or a `400` on the single-URL rate endpoint) with a message like `"access to this page is not allowed"`. Reach out to your TollBit contact to request access to that property.

Third-party search providers return their own status codes and error shapes — consult each provider's documentation.

***

## Additional Resources

{/* markdownlint-disable-next-line MD044 */}

- **TollBit API documentation**: [docs.tollbit.com](https://docs.tollbit.com)
- **Perplexity Search API**: [docs.perplexity.ai](https://docs.perplexity.ai)
- **Firecrawl Search API**: [docs.firecrawl.dev](https://docs.firecrawl.dev)
- **Exa Search API**: [docs.exa.ai](https://docs.exa.ai)
