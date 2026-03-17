---
title: Search to Content Workflow
excerpt: >-
  Complete workflow guide for searching content, checking rates, and retrieving
  content using the Tollbit API with cURL or Python SDK.
---
# Search to Content Workflow

This guide walks you through the complete workflow of using the Tollbit API to search for content, check rates, and retrieve content. You can use either cURL commands or the Python SDK.

## Prerequisites

Before you begin, make sure you have:

<Tabs>
  <Tab title="cURL">
    1. **cURL installed**: cURL should be available on most systems. Check with:
       ```bash
       curl --version
       ```

    2. **Obtained your API key**: You'll need a Tollbit organization API key. Replace `<key>` in the examples below with your actual API key.

    3. **Set your user agent**: This identifies your application. The user agent should be added to the dev dashboard in the `My Agents` tab
  </Tab>

  <Tab title="Python">
    1. **Installed the SDK**:
       ```bash
       pip install tollbit-python-sdk
       ```

    2. **Obtained your API key**: You'll need a Tollbit organization API key. Set it as an environment variable:
       ```bash
       export TOLLBIT_ORG_API_KEY="your-api-key-here"
       ```

    3. **Set your user agent**: This identifies your application:
       ```bash
       export TOLLBIT_USER_AGENT="your-app-name/1.0.0"
       ```
  </Tab>
</Tabs>

## Complete Workflow

The typical workflow consists of three main steps:

1. **Search** for content
2. **Get rates** for a specific result
3. **Get the content** (token acquisition is handled automatically with Python SDK, or separately with cURL)

Let's walk through each step with examples.

***

## Step 1: Make a Search

Start by searching for content across the TollBit platform.

### Basic Search

<Tabs>
  <Tab title="cURL">
    ```bash
    curl --location 'https://gateway.tollbit.com/dev/v2/search?q=DIY%20projects%20for%20Millenials' \
    --header 'TollBitKey: <key>'
    ```
  </Tab>

  <Tab title="Python">
    ```python
    from tollbit import search
    import os

    # Get your API key and user agent
    api_key = os.getenv("TOLLBIT_ORG_API_KEY", "YOUR_API_KEY_HERE")
    user_agent = os.getenv("TOLLBIT_USER_AGENT", "tollbit-python-sdk-example/0.1.0")

    # Create a search client
    search_client = search.create_client(secret_key=api_key, user_agent=user_agent)

    # Perform a search
    results = search_client.search(q="DIY home projects for millennials")

    # Display results
    print(f"Found {len(results.items)} results")
    print(f"Next token: {results.next_token}")

    for i, item in enumerate(results.items, 1):
        print(f"\nResult {i}:")
        print(f"  Title: {item.title}")
        print(f"  URL: {item.url}")
        print(f"  Published: {item.published_date}")
        print(f"  Publisher: {item.publisher.name} ({item.publisher.domain})")
        print(f"  Discoverable: {item.availability.discoverable}")
        print(f"  Ready to license: {item.availability.ready_to_license}")
    ```
  </Tab>
</Tabs>

### Search with Additional Parameters

You can customize your search with additional query parameters:

* `size`: Number of results to return (maximum 20)
* `next-token`: Token for pagination to get the next page
* `properties`: Comma-separated list of domains (max 20) to filter results to only those properties

<Tabs>
  <Tab title="cURL">
    ```bash
    # Search with size limit
    curl --location 'https://gateway.tollbit.com/dev/v2/search?q=python%20tutorial&size=5' \
    --header 'TollBitKey: <key>'

    # Search on specific properties (filter to only these domains)
    curl --location 'https://gateway.tollbit.com/dev/v2/search?q=tutorial&size=10&properties=example.com,tutorial.com' \
    --header 'TollBitKey: <key>'

    # Pagination - get next page
    curl --location 'https://gateway.tollbit.com/dev/v2/search?q=DIY%20projects&next-token=<next_token_from_previous_response>' \
    --header 'TollBitKey: <key>'
    ```
  </Tab>

  <Tab title="Python">
    ```python
    # Search with size limit
    results = search_client.search(q="python tutorial", size=5)

    # Search on specific properties (filter to only these domains)
    results = search_client.search(
        q="tutorial",
        size=10,
        properties=["example.com", "tutorial.com"],
    )

    # Pagination - get next page
    if results.next_token:
        next_page = search_client.search(
            q="DIY home projects for millennials",
            next_token=results.next_token,
        )
    ```
  </Tab>
</Tabs>

### Understanding the Response

The search response is a JSON object with the following structure:

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
        "discoverable": true,
        "readyToLicense": true
      }
    }
  ]
}
```

**Understanding Search Result Flags**:

* `discoverable`: Indicates the content is available in Tollbit
* `readyToLicense` / `ready_to_license`: Indicates the property owner has already set rates and the content is ready to transact

### Example: Parsing Search Results

<Tabs>
  <Tab title="cURL">
    ```bash
    # Save the response to a file for easier parsing
    curl --location 'https://gateway.tollbit.com/dev/v2/search?q=DIY%20projects%20for%20Millenials' \
    --header 'TollBitKey: <key>' \
    --output search_results.json

    # Using jq to parse (if installed)
    cat search_results.json | jq '.items[0].url'  # Get first result URL
    cat search_results.json | jq '.items[0].title'  # Get first result title
    ```
  </Tab>

  <Tab title="Python">
    ```python
    # Results are already parsed into objects
    if results.items:
        first_result = results.items[0]
        print(f"URL: {first_result.url}")
        print(f"Title: {first_result.title}")
    ```
  </Tab>
</Tabs>

***

## Step 2: Get Rates for a Result

Once you've found a result you're interested in, check the available rates and licenses for that content.

### Get Rates

<Tabs>
  <Tab title="cURL">
    ```bash
    curl --location 'https://gateway.tollbit.com/tollbit/dev/v2/rate/https://www.forbes.com/sites/jefffromm/2018/03/27/marketing-to-millennial-new-home-buyers-lowes-innovative-diy-training-program/' \
    --header 'TollbitKey: <key>'
    ```

    **Note**: The URL path in the rate endpoint should be the full URL of the content (including protocol), URL-encoded if necessary.
  </Tab>

  <Tab title="Python">
    ```python
    from tollbit import use_content
    import os

    api_key = os.getenv("TOLLBIT_ORG_API_KEY", "YOUR_API_KEY_HERE")
    user_agent = os.getenv("TOLLBIT_USER_AGENT", "tollbit-python-sdk-example/0.1.0")

    # Create a use_content client
    content_client = use_content.create_client(secret_key=api_key, user_agent=user_agent)

    # Get rates for a specific URL (from your search results)
    selected_url = results.items[0].url  # Using the first result from Step 1
    rate_info = content_client.get_rate(url=selected_url)

    # Display rate information
    print(f"\nFound {len(rate_info)} rate(s) for {selected_url}:")
    for rate in rate_info:
        print(f"\n  Price: {rate.price.price_micros / 1_000_000:.2f} {rate.price.currency}")
        print(f"  License ID: {rate.license.id}")
        print(f"  License Type: {rate.license.license_type}")
        print(f"  License Path: {rate.license.license_path}")
        print(f"  Permissions: {[p.name for p in rate.license.permissions]}")
    ```
  </Tab>
</Tabs>

### Understanding the Response

The rates response is a JSON array of rate objects:

```json
[
  {
    "price": {
      "priceMicros": 11000000,
      "currency": "USD"
    },
    "license": {
      "id": "string",
      "licenseType": "ON_DEMAND_LICENSE",
      "licensePath": "string",
      "permissions": [
        {
          "name": "string"
        }
      ]
    }
  }
]
```

**Note**: The `priceMicros` / `price_micros` field represents the price in micros (1/1,000,000 of the currency unit). To convert to dollars, divide by 1,000,000. For example, `11000000` micros = `11.00` USD.

### Handling Access Restrictions

If you receive a `400 Bad Request` error with a message like `"access to this page is not allowed"`, it means the property exists in Tollbit but is not enabled for your organization. In this case, you should reach out to your Tollbit contact to request access to that property.

Example error response:

```json
{
    "detail": "access to this page is not allowed",
    "instance": "/tollbit/dev/v2/rate/https://www.example.com/article",
    "status": 400,
    "title": "Bad Request",
    "type": "about:blank"
}
```

<Tabs>
  <Tab title="cURL">
    Check the HTTP status code in your response. A 400 status indicates access restrictions.
  </Tab>

  <Tab title="Python">
    ```python
    from tollbit._apis.errors import BadRequestError

    try:
        rate_info = content_client.get_rate(url=selected_url)
    except BadRequestError as e:
        if "access to this page is not allowed" in str(e):
            print("This property is not enabled for your organization.")
            print("Please contact your Tollbit representative to enable access.")
        else:
            raise
    ```
  </Tab>
</Tabs>

### Example: Parsing Rates

<Tabs>
  <Tab title="cURL">
    ```bash
    # Save the response
    curl --location 'https://gateway.tollbit.com/tollbit/dev/v2/rate/<url>' \
    --header 'TollbitKey: <key>' \
    --output rates.json

    # Extract price using jq
    cat rates.json | jq '.[0].price.priceMicros'  # Get first rate price in micros
    cat rates.json | jq '.[0].license.licenseType'  # Get first rate license type
    ```
  </Tab>

  <Tab title="Python">
    ```python
    # Rates are already parsed into objects
    if rate_info:
        first_rate = rate_info[0]
        price_usd = first_rate.price.price_micros / 1_000_000
        print(f"Price: {price_usd:.2f} {first_rate.price.currency}")
        print(f"License Type: {first_rate.license.license_type}")
    ```
  </Tab>
</Tabs>

***

## Step 3: Get the Content

To retrieve content, you need to:

1. Obtain an access token
2. Use that token to retrieve the content

**Note**: With the Python SDK, token acquisition is handled automatically by the `get_sanctioned_content` method. With cURL, you need to obtain the token separately.

### Step 3a: Get a Token (cURL only)

<Tabs>
  <Tab title="cURL">
    First, obtain an access token by specifying:

    * The URL of the content
    * Maximum price you're willing to pay (in micros)
    * Currency
    * License type
    * User agent

    ```bash
    curl --location 'https://gateway.tollbit.com/dev/v2/tokens/content' \
    --header 'Content-Type: application/json' \
    --header 'TollbitKey: <key>' \
    --data '{
        "url": "https://www.forbes.com/sites/jefffromm/2018/03/27/marketing-to-millennial-new-home-buyers-lowes-innovative-diy-training-program/",
        "userAgent": "test-agent",
        "maxPriceMicros": 11000000,
        "currency": "USD",
        "licenseType": "ON_DEMAND_LICENSE"
    }'
    ```

    **Note**: For `CUSTOM_LICENSE` type, you'll also need to include `licenseCuid` in the request body.

    ### Understanding the Token Response

    The token response is a JSON object:

    ```json
    {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
    }
    ```

    Save this token value - you'll need it in the next step.
  </Tab>

  <Tab title="Python">
    The Python SDK handles token acquisition automatically when you call `get_sanctioned_content`. You don't need to manually obtain a token.
  </Tab>
</Tabs>

### Step 3b: Get the Content

<Tabs>
  <Tab title="cURL">
    Use the token obtained in Step 3a to retrieve the actual content:

    ```bash
    curl --location 'https://gateway.tollbit.com/dev/v2/content/https://www.forbes.com/sites/jefffromm/2018/03/27/marketing-to-millennial-new-home-buyers-lowes-innovative-diy-training-program/' \
    --header 'TollbitToken: <Token>' \
    --header 'User-Agent: test-agent'
    ```

    **Note**: The URL path in the content endpoint should be the full URL of the content (including protocol).

    ### Content Format Options

    You can request different content formats by adding an `Accept` header:

    ```bash
    # Get Markdown format (default)
    curl --location 'https://gateway.tollbit.com/dev/v2/content/<url>' \
    --header 'TollbitToken: <Token>' \
    --header 'User-Agent: test-agent' \
    --header 'Accept: text/markdown'

    # Get HTML format
    curl --location 'https://gateway.tollbit.com/dev/v2/content/<url>' \
    --header 'TollbitToken: <Token>' \
    --header 'User-Agent: test-agent' \
    --header 'Accept: text/html'
    ```
  </Tab>

  <Tab title="Python">
    Retrieve the actual content using `get_sanctioned_content`. This method automatically handles token acquisition and content retrieval in a single call. You need to specify:

    * The URL of the content
    * Maximum price you're willing to pay (in micros)
    * Currency
    * License type

    ```python
    from tollbit import use_content
    from tollbit import licenses
    from tollbit import currencies
    from tollbit import content_formats
    import os

    api_key = os.getenv("TOLLBIT_ORG_API_KEY", "YOUR_API_KEY_HERE")
    user_agent = os.getenv("TOLLBIT_USER_AGENT", "tollbit-python-sdk-example/0.1.0")

    content_client = use_content.create_client(secret_key=api_key, user_agent=user_agent)

    # Select a URL from your search results (from Step 1)
    selected_url = results.items[0].url

    # Optionally get rates first to determine pricing (from Step 2)
    rate_info = content_client.get_rate(url=selected_url)
    if rate_info:
        selected_rate = rate_info[0]
        max_price = selected_rate.price.price_micros
    else:
        max_price = 11000000  # 11.00 USD in micros

    # Get content in Markdown format (default)
    # This automatically obtains a token and retrieves the content
    data = content_client.get_sanctioned_content(
        url=selected_url,
        max_price_micros=max_price,
        currency=currencies.USD,
        license_type=licenses.types.ON_DEMAND_LICENSE,
    )

    # Access the content
    print("Content:")
    print(f"Header: {data.content.header}")
    print(f"Body: {data.content.body}")
    print(f"Footer: {data.content.footer}")

    # Access metadata
    print("\nMetadata:")
    print(f"Title: {data.metadata.title}")
    print(f"Description: {data.metadata.description}")
    print(f"Author: {data.metadata.author}")
    print(f"Published: {data.metadata.published}")
    print(f"Modified: {data.metadata.modified}")

    # Access rate information (if available)
    if data.rate:
        print(f"\nRate: {data.rate.price.price_micros / 1_000_000:.2f} {data.rate.price.currency}")

    # Get content in HTML format
    data_html = content_client.get_sanctioned_content(
        url=selected_url,
        max_price_micros=11000000,
        currency=currencies.USD,
        license_type=licenses.types.ON_DEMAND_LICENSE,
        format=content_formats.HTML,
    )
    ```
  </Tab>
</Tabs>

### Understanding the Content Response

The content response is a JSON object:

```json
{
  "content": {
    "header": "string",
    "body": "string",
    "footer": "string"
  },
  "metadata": {
    "title": "string",
    "description": "string",
    "imageUrl": "string",
    "author": "string",
    "published": "string",
    "modified": "string"
  },
  "rate": {
    "price": {
      "priceMicros": 11000000,
      "currency": "USD"
    },
    "license": {
      "id": "string",
      "licenseType": "ON_DEMAND_LICENSE",
      "licensePath": "string",
      "permissions": [
        {
          "name": "string"
        }
      ]
    }
  }
}
```

### Available License Types

* `ON_DEMAND_LICENSE`: Standard on-demand license
* `ON_DEMAND_FULL_USE_LICENSE`: Full-use on-demand license
* `CUSTOM_LICENSE`: Custom license (requires `licenseCuid` parameter in token request for cURL, or `license_id` for Python SDK)

### Available Content Formats

* **Markdown** (default): `text/markdown` for cURL, `content_formats.MARKDOWN` for Python
* **HTML**: `text/html` for cURL, `content_formats.HTML` for Python

***

## Complete Example: End-to-End Workflow

Here's a complete example that ties all the steps together:

<Tabs>
  <Tab title="cURL">
    ```bash
    #!/bin/bash

    # Configuration
    API_KEY="<your-api-key>"
    USER_AGENT="test-agent"
    SEARCH_QUERY="DIY projects for Millenials"

    # Step 1: Search for content
    echo "Step 1: Searching for content..."
    SEARCH_RESPONSE=$(curl -s --location "https://gateway.tollbit.com/dev/v2/search?q=$(echo $SEARCH_QUERY | jq -sRr @uri)" \
      --header "TollBitKey: $API_KEY")

    # Extract first result URL (requires jq)
    SELECTED_URL=$(echo $SEARCH_RESPONSE | jq -r '.items[0].url')
    SELECTED_TITLE=$(echo $SEARCH_RESPONSE | jq -r '.items[0].title')

    if [ "$SELECTED_URL" == "null" ] || [ -z "$SELECTED_URL" ]; then
      echo "No results found."
      exit 1
    fi

    echo "Selected: $SELECTED_TITLE"
    echo "URL: $SELECTED_URL"

    # Step 2: Get rates
    echo ""
    echo "Step 2: Getting rates..."
    # URL encode the URL for the rate endpoint
    ENCODED_URL=$(echo "$SELECTED_URL" | jq -sRr @uri)
    RATES_RESPONSE=$(curl -s --location "https://gateway.tollbit.com/tollbit/dev/v2/rate/$ENCODED_URL" \
      --header "TollbitKey: $API_KEY")

    # Extract first rate price (requires jq)
    MAX_PRICE_MICROS=$(echo $RATES_RESPONSE | jq -r '.[0].price.priceMicros')
    LICENSE_TYPE=$(echo $RATES_RESPONSE | jq -r '.[0].license.licenseType')

    if [ "$MAX_PRICE_MICROS" == "null" ] || [ -z "$MAX_PRICE_MICROS" ]; then
      echo "No rates available for this content."
      exit 1
    fi

    echo "Price: $((MAX_PRICE_MICROS / 1000000)).$((MAX_PRICE_MICROS % 1000000 / 10000)) USD"
    echo "License Type: $LICENSE_TYPE"

    # Step 3a: Get token
    echo ""
    echo "Step 3a: Getting token..."
    TOKEN_RESPONSE=$(curl -s --location 'https://gateway.tollbit.com/dev/v2/tokens/content' \
      --header 'Content-Type: application/json' \
      --header "TollbitKey: $API_KEY" \
      --data "{
        \"url\": \"$SELECTED_URL\",
        \"userAgent\": \"$USER_AGENT\",
        \"maxPriceMicros\": $MAX_PRICE_MICROS,
        \"currency\": \"USD\",
        \"licenseType\": \"$LICENSE_TYPE\"
      }")

    TOKEN=$(echo $TOKEN_RESPONSE | jq -r '.token')

    if [ "$TOKEN" == "null" ] || [ -z "$TOKEN" ]; then
      echo "Failed to get token."
      exit 1
    fi

    # Step 3b: Get content
    echo ""
    echo "Step 3b: Getting content..."
    CONTENT_RESPONSE=$(curl -s --location "https://gateway.tollbit.com/dev/v2/content/$ENCODED_URL" \
      --header "TollbitToken: $TOKEN" \
      --header "User-Agent: $USER_AGENT")

    # Display content
    TITLE=$(echo $CONTENT_RESPONSE | jq -r '.metadata.title')
    BODY_PREVIEW=$(echo $CONTENT_RESPONSE | jq -r '.content.body' | head -c 200)

    echo ""
    echo "Content retrieved successfully!"
    echo "Title: $TITLE"
    echo ""
    echo "Content preview (first 200 chars):"
    echo "$BODY_PREVIEW..."
    ```
  </Tab>

  <Tab title="Python">
    ```python
    from tollbit import search
    from tollbit import use_content
    from tollbit import licenses
    from tollbit import currencies
    import os

    # Setup
    api_key = os.getenv("TOLLBIT_ORG_API_KEY", "YOUR_API_KEY_HERE")
    user_agent = os.getenv("TOLLBIT_USER_AGENT", "tollbit-python-sdk-example/0.1.0")

    # Step 1: Search for content
    print("Step 1: Searching for content...")
    search_client = search.create_client(secret_key=api_key, user_agent=user_agent)
    results = search_client.search(q="DIY home projects", size=5)

    if not results.items:
        print("No results found.")
        exit(1)

    # Select the first result
    selected_result = results.items[0]
    print(f"\nSelected: {selected_result.title}")
    print(f"URL: {selected_result.url}")

    # Step 2: Get rates
    print("\nStep 2: Getting rates...")
    content_client = use_content.create_client(secret_key=api_key, user_agent=user_agent)
    rate_info = content_client.get_rate(url=selected_result.url)

    if not rate_info:
        print("No rates available for this content.")
        exit(1)

    # Display available rates
    print(f"\nAvailable rates:")
    for i, rate in enumerate(rate_info, 1):
        price_usd = rate.price.price_micros / 1_000_000
        print(f"  {i}. {price_usd:.2f} {rate.price.currency} - {rate.license.license_type}")

    # Step 3: Get content (token acquisition is handled automatically)
    print("\nStep 3: Getting content...")
    selected_rate = rate_info[0]  # Use the first available rate

    try:
        data = content_client.get_sanctioned_content(
            url=selected_result.url,
            max_price_micros=selected_rate.price.price_micros,
            currency=currencies.USD,
            license_type=licenses.types.ON_DEMAND_LICENSE,
        )
        
        print(f"\nContent retrieved successfully!")
        print(f"\nTitle: {data.metadata.title}")
        print(f"\nContent preview (first 200 chars):")
        print(data.content.body[:200] + "...")
        
    except Exception as e:
        print(f"Error retrieving content: {e}")
    ```
  </Tab>
</Tabs>

***

## Error Handling

The API returns standard HTTP status codes:

* **200 OK**: Request successful
* **400 Bad Request**: Invalid request (e.g., access not allowed, invalid parameters)
* **401 Unauthorized**: Invalid or missing API key
* **500-599 Server Error**: Server-side errors

### Example Error Response

```json
{
    "type": "about:blank",
    "title": "Bad Request",
    "status": 400,
    "detail": "access to this page is not allowed",
    "instance": "/tollbit/dev/v2/rate/https://www.example.com/article"
}
```

<Tabs>
  <Tab title="cURL">
    ```bash
    # Check HTTP status code
    HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" \
      --location 'https://gateway.tollbit.com/dev/v2/search?q=test' \
      --header 'TollBitKey: <key>')

    if [ $HTTP_CODE -ne 200 ]; then
      echo "Error: HTTP $HTTP_CODE"
      exit 1
    fi
    ```
  </Tab>

  <Tab title="Python">
    ```python
    from tollbit._apis.errors import (
        UnauthorizedError,  # 401 - Invalid API key
        BadRequestError,    # 400 - Invalid request
        ServerError,        # 500-599 - Server errors
        UnknownError,       # Other status codes
    )

    try:
        data = content_client.get_sanctioned_content(...)
    except UnauthorizedError:
        print("Invalid API key")
    except BadRequestError as e:
        print(f"Invalid request: {e}")
    except ServerError:
        print("Server error occurred")
    except Exception as e:
        print(f"Unexpected error: {e}")
    ```
  </Tab>
</Tabs>

***

## Additional Resources

* **API Documentation**: Visit [docs.tollbit.com](https://docs.tollbit.com)

***

## Summary

The typical workflow is:

1. **Search** → Find content using the search endpoint
2. **Get Rates** → Check pricing for a specific result
3. **Get Token** (cURL only) → Obtain access token
4. **Get Content** → Retrieve the actual content

**With Python SDK**: The `get_sanctioned_content` method handles both token acquisition and content retrieval in a single call, simplifying your code.

**With cURL**: You need to obtain the token separately before requesting content.
