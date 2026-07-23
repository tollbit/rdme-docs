---
title: Fastly
excerpt: Learn how to integrate TollBit with Fastly.
deprecated: false
hidden: false
metadata:
  robots: index
---
<<<<<<< Updated upstream
To integrate TollBit with your Fastly instance, we'll use Fastly's HTTPS endpoint to configure Analytics and VCL scripts to implement Agent Site. Please see the steps below outlining each step.

## Steps for Analytics
=======
Every TollBit Fastly setup is built from the same two pieces, which you create and own in your Fastly service:

- A **backend** named `tollbit_origin` that bot traffic is forwarded through.
- An **HTTPS log streaming endpoint** that powers TollBit analytics.

Set those up first, then choose how bot blocking is managed:

- **TollBit Managed** — you give TollBit a scoped API token, and bot blocking is controlled from the TollBit dashboard. TollBit only rewrites the content of a dynamic VCL snippet you own — it never creates objects, clones versions, or activates configurations, so it fits cleanly into Terraform and other infrastructure-as-code workflows.
- **User Managed** — you write and maintain the snippet content yourself, and TollBit needs no API access to your Fastly account.

> 📘 Pro Tip
>
> If you used our earlier dashboard-managed integration, it is still available under the "Fastly Integration (Legacy)" tab on the Integrations page. New setups should follow the steps below.

## Set up the TollBit Origin

Create a backend named exactly `tollbit_origin` in your Fastly service, pointing at `tollbit.<your-domain>` (port 443, TLS). For example, if your site is `www.example.com`, the address is `tollbit.example.com`.

In the Fastly console: Edit Configuration → clone your active version → Origins → Create a host, set the name to `tollbit_origin` and the address to `tollbit.<your-domain>`, and enable TLS.

If you manage your service with Terraform, add this to your `fastly_service_vcl` resource instead:

```hcl
backend {
  name              = "tollbit_origin"
  address           = "tollbit.<your-domain>"
  port              = 443
  use_ssl           = true
  ssl_cert_hostname = "tollbit.<your-domain>"
  ssl_sni_hostname  = "tollbit.<your-domain>"
}
```

## Steps for Analytics

Analytics is powered by a log streaming endpoint on your own Fastly service — TollBit never adds or removes logging configuration for you. If you use the TollBit dashboard, it shows analytics as enabled once it detects the endpoint; you may need to refresh the page after setting it up.
>>>>>>> Stashed changes

**Create a new Logging Configuration**

Go to your <Anchor target="_blank" href="https://manage.fastly.com/configure">Fastly Dashboard</Anchor> and pick the correct domain. Click “Edit Configuration”, and clone your current configuration. This saves a new configuration version as a draft, and allows you to rollback if necessary. This should bring you to a new screen. On the sidebar, scroll down until you see Logging and click on that. Then, click “Create Endpoint”.

![Fastly Sidebar](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-sidebar.png)

**Configure your logs to be sent to our logging endpoint**

![Fastly Http Config](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-http-config.png)

Find the HTTP logging endpoint and click “Create endpoint”. You can set the name to anything descriptive (e.g. `tollbit-prod`). Keep the placement option as the default selection. Make sure your log format is exactly as follows, without extra trailing spaces or newlines:

```json
{ "timestamp": "%{strftime(\{"%Y-%m-%dT%H:%M:%S%z"\}, time.start)}V", "ip_address": "%{req.http.Fastly-Client-IP}V", "geo_country": "%{client.geo.country_name}V", "geo_city": "%{client.geo.city}V", "geo_postal_code":"%{client.geo.postal_code}V", "geo_latitude":"%{client.geo.latitude}V", "geo_longitude":"%{client.geo.longitude}V", "host": "%{if(req.http.Fastly-Orig-Host, req.http.Fastly-Orig-Host, req.http.Host)}V", "url": "%{json.escape(req.url)}V", "request_method": "%{json.escape(req.method)}V", "request_protocol": "%{json.escape(req.proto)}V", "request_referer": "%{json.escape(req.http.referer)}V", "request_user_agent": "%{json.escape(req.http.User-Agent)}V", "request_latency":"%{time.elapsed.usec}V", "response_state": "%{json.escape(fastly_info.state)}V", "response_status": %{std.itoa(resp.status)}V, "response_reason": %{if(resp.response, "%22"+json.escape(resp.response)+"%22", "null")}V, "response_body_size": %{resp.body_bytes_written}V, "fastly_server": "%{json.escape(server.identity)}V", "fastly_is_edge": %{if(fastly.ff.visits_this_service == 0, "true", "false")}V, "signature": "%{json.escape(req.http.signature)}V", "signature_agent": "%{json.escape(req.http.signature-agent)}V", "signature_input": "%{json.escape(req.http.signature-input)}V" }
```

Finally, set the URL to [https://log.tollbit.com/log](https://log.tollbit.com/log).

![Fastly Log Config](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-log-config.png)

**Ensure that your Requests are Authenticated**

Go into Advanced Options and set the “Custom header name” field to “TollbitKey”. You must set the customer header value to your secret key. Log into your <Anchor target="_blank" href="https://app.tollbit.com">TollBit portal</Anchor> and go into the API key tab and copy your secret key. Paste it into the “Custom header value” field with no trailing spaces. Keep all the other settings as default, scroll to the bottom, and save.

> 🚧 Note
>
> The TollBit dashboard detects the endpoint by its URL, so analytics will show as enabled even if the `TollbitKey` header is missing or wrong — but your logs will be rejected and no analytics data will appear. If the analytics charts stay empty, check this header first.

![Fastly Activate](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-activate.png)

Once you are ready to publish these changes, click the “Activate” button. Keep in mind that if you have other unpublished changes in Fastly, this may also publish those as well.

<<<<<<< Updated upstream
## Steps for Agent Site

Fastly allows you to set up redirects using VCL snippets. In this document, we will go over setting up forwarding requests from known bots to your `tollbit` subdomain.

<Callout icon="📘" theme="info">
  ### Pro Tip

  The code shown here is for a clean Fastly environment. If you have any existing VCL scripts that intercept requests, you will need to integrate these scripts into your existing workflow.
</Callout>

You can use Fastly VCL to rewrite URLs and proxy from a different origin, in this case, the `tollbit` subdomain.

Go to the Deliver tab and select the domain you wish to add bot forwarding to. On the right side of the screen, click the Edit configuration button and choose to clone your current active version.

![Fastly Edit Configuration](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-edit-configuration.png)

<br />

**Create a New Host**

Navigate to the Origin tab in your left navigation and create a new host. Use your `tollbit` subdomain as the host.&#x20;

Once you create this, you might see a warning that this backend is unused. No need to worry about this for now.

![](https://files.readme.io/1590d22a507a3fbd4af45aee2e88356de590545284a199a607ed83849baf61f9-image6.png)

<br />

Once it’s been added, scroll down and click the little pencil icon next to the host name to edit this and give it a better name. You can name it something more clear, like “**tollbit\_origin**”. Scroll down and click Update to save.&#x20;

In this example, we are using our test website _thedailydispatching.com_.

![](https://files.readme.io/0cfa040e63a3b941f453e5810ed05c842579e955a1d77af966e089893a14cbb1-image1.png)

<br />

**Add (or Modify) your VCL**

On the left hand sidebar, click "VCL Snippets".

![Fastly Vcl Snippet](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-vcl-snippet.png)

<br />

<Callout icon="🚧" theme="warn">
  ### Note

  If you’ve followed our docs previously to set up your recv and error VCL snippets, this would just require some quick edits. We will walk through both snippets as if setting them up as new in these docs.
</Callout>

In your left nav, go to VCL snippets. If you haven’t already, create a new snippet for the recv step called **tollbit-recv-snippet**.

![](https://files.readme.io/7cca53f95f7b1b2521427658ddcfb6a4e2f2c79ce0f42e7cef00715a54b4b076-image9.png)

<br />

For the recv snippet, paste in the following snippet. Ensure that the backend name is the same that you’ve set before. If you applied our recommended edit, it will be **F\_tollbit\_origin** (line 2 below).

```json
if (req.http.user-agent ~ "(?i)amazonbot|amzn-searchbot|anthropic-ai|bytespider|ccbot|chatgpt-user|claude-code|claude-searchbot|claude-user|claude-web|claudebot|cohere-ai|diffbot|exabot|gptbot|meta-externalagent|meta-webindexer|oai-adsbot|oai-searchbot|perplexity-user|perplexitybot|timpibot|youbot") {
  set req.backend = F_tollbit_origin;
  if (std.prefixof(req.http.host, "www.")) {
      set req.http.host = std.replace_prefix(req.http.host, "www.", "tollbit.");
  } else {
      set req.http.host = "tollbit." + req.http.host;
  }
  return(pass); 
}

```

If you created an error snippet in the past for the redirect, you no longer need it, and can remove it.

<Callout icon="🚧" theme="warn">
  ### Note

  The VCL scripts you just added **will intercept** and potentially **redirect** traffic to your main site. Please ensure that you have tested this in a test environment or for a small subset of pages before activating this across your entire site.
</Callout>

This should now be all you need to forward known bot traffic to your `tollbit` subdomain!&#x20;

Once this is saved or updated, you can now activate your service configuration and this will be live in a few minutes.
=======
If you manage logging with Terraform, add a `logging_https` block to your `fastly_service_vcl` resource with `url = "https://log.tollbit.com/log"`, `method = "POST"`, `header_name = "TollbitKey"`, `header_value` set to your TollBit secret key, `message_type = "blank"`, `format_version = 2`, and the log format above. Note that Terraform treats `%{` in strings as a template directive, so each `%{` in the format string must be escaped as `%%{`.

## Steps for Agent Site

Bot forwarding works the same way in both options: a **dynamic VCL snippet** named `tollbit_recv_dynamic_snippet` (type `recv`) matches known bot user agents and proxies them through the `tollbit_origin` backend to your `tollbit` subdomain. Dynamic snippets keep their content outside the versioned configuration, so the bot list can change without cloning and activating a new service version. The only difference between the two options is who manages that snippet's content.

> 🚧 Note
>
> The snippet **will intercept** and forward matching traffic on your main site. Please ensure that you have tested this in a test environment or for a small subset of pages before activating this across your entire site.

### Fastly (TollBit Managed)
>>>>>>> Stashed changes

**Get Service ID and API Key from Fastly**

Go to your Fastly Dashboard and pick the domain associated with your property.

Right under your service name, you’ll see an alphanumeric string. It should be the same alphanumeric string that completes the URL string for the page. See the highlight below for reference.

![Integrations Fastly Service](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/integrations-fastly-service.png)

Next, hover over Account on the main navigation bar on the left and choose API tokens > personal tokens.

![Integrations Fastly Service 2](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/integrations-fastly-service-2.png)

**Setup Integration in TollBit**

Go to your TollBit dashboard and pick the Integrations tab in the main navigation menu. Input your Fastly API key and service ID in the form and click Save.

![Integrations Fastly Service 3](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/integrations-fastly-service-3.png)

**Create the dynamic snippet and connect**

Create a dynamic VCL snippet named exactly `tollbit_recv_dynamic_snippet` with type `recv`. You can leave its content empty — TollBit manages the content from here on. With Terraform, add this to your `fastly_service_vcl` resource:

```hcl
dynamicsnippet {
  name = "tollbit_recv_dynamic_snippet"
  type = "recv"
}
```

> 🚧 Leave the snippet's content unmanaged
>
> Do **not** add a `fastly_service_dynamic_snippet_content` resource for this snippet. Dynamic snippet content lives outside the versioned configuration — that is what lets TollBit update your bot list without publishing a new version. If Terraform manages the content too, every TollBit update will show up as drift.

Activate the version that contains the backend and snippet, then click **Verify connection** on the Fastly integration page in your TollBit dashboard. If anything is missing or misnamed, the error message will tell you exactly what to fix.

**Manage bot blocking from the dashboard**

Once connected, use the Bot Paywall section of the integration page:

- Toggle on “Block” for each agent you would like to forward.
- “Block All Bots” blocks every bot in the list, including any custom user agents already in your snippet.
- “Allow All Bots” clears the block list entirely.

Every change is applied by rewriting the snippet's content and takes effect within seconds. No new service version is created, so there is nothing to activate and nothing for Terraform to reconcile.

### Fastly (User Managed)

You can also manage the snippet's content yourself, without giving TollBit API access. This allows for additional customizations for implementing bot forwarding. Please reach out to [team@tollbit.com](mailto:team@tollbit.com) if you'd like to discuss this implementation route considering your use case.

On the left hand sidebar of your cloned configuration, click "VCL Snippets".

![Fastly Vcl Snippet](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-vcl-snippet.png)

Create a snippet named `tollbit_recv_dynamic_snippet` and choose the **Dynamic** snippet type. Make sure that the placement of the snippet is within the recv subroutine. Because this is the same snippet name and type the TollBit Managed option uses, you can switch to it at any point by adding your API credentials and clicking Verify connection in the TollBit dashboard.

![Fastly Recv](https://raw.githubusercontent.com/tollbit/rdme-docs/v1.0/public/fastly-recv.png)

Copy and paste the following code block into the VCL input field and save, then activate the version. Edit the user agent list to control which bots are forwarded.

```
if (req.http.user-agent ~ "(?i)chatgpt-user|perplexitybot|gptbot|anthropic-ai|ccbot|claude-web|claudebot|cohere-ai|youbot|diffbot|oai-searchbot|meta-externalagent|timpibot|amazonbot|bytespider|perplexity-user") {
  set req.backend = F_tollbit_origin;
  if (std.prefixof(req.http.host, "www.")) {
    set req.http.host = std.replace_prefix(req.http.host, "www.", "tollbit.");
  } else {
    set req.http.host = "tollbit." + req.http.host;
  }
  return(pass);
}
```

> 📘 Note
>
> Once the snippet exists on your active version, later edits to a dynamic snippet's *content* take effect immediately — there is no draft-and-activate step for content changes.

<br />
